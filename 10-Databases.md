# Databases (PostgreSQL, MySQL, NoSQL, Redis) - Ответы на вопросы интервью

## 📚 Теория

### 1. PostgreSQL MVCC

**Вопрос:** Объясните как работает MVCC в PostgreSQL и почему нужен VACUUM.

**Ответ:**

**MVCC (Multi-Version Concurrency Control)** — механизм управления конкурентным доступом, который позволяет читателям и писателям не блокировать друг друга.

```
┌─────────────────────────────────────────────────────────────┐
│                    PostgreSQL MVCC                          │
│                                                              │
│  Transaction A (T1)           Transaction B (T2)            │
│  BEGIN;                       BEGIN;                        │
│  UPDATE users SET             SELECT * FROM users;          │
│    name = 'Alice' WHERE                                          │
│    id = 1;                                                   │
│  ┌─────────────────┐          ┌─────────────────┐           │
│  │ Row Versions:   │          │ Sees:           │           │
│  │                 │          │   xid=100       │           │
│  │ [xid=100]       │          │   name='Bob'    │           │
│  │ name='Bob'      │◄─────────│   (старая       │           │
│  │ xmin=100        │          │   версия)       │           │
│  │ xmax=null       │          │                 │           │
│  │                 │          │ T2 видит        │           │
│  │ [xid=101]       │          │ снимок на       │           │
│  │ name='Alice'    │          │ момент начала   │           │
│  │ xmin=101        │          │ транзакции      │           │
│  │ xmax=null       │          │                 │           │
│  └─────────────────┘          └─────────────────┘           │
│                                                              │
│  Системные колонки:                                          │
│  • xmin — ID транзакции создавшей версию                     │
│  • xmax — ID транзакции удалившей версию (null = активная)   │
│  • cmin/cmax — для команд внутри транзакции                  │
└─────────────────────────────────────────────────────────────┘
```

**Как работает Visibility:**

```sql
-- Каждая транзакция видит снимок (snapshot) на момент старта
-- Visibility rules:
-- 1. Строка видима если: xmin committed AND (xmax is NULL OR xmax aborted)
-- 2. Строка НЕ видима если: xmin in progress или xmax committed
```

**Почему нужен VACUUM:**

```
Проблема: UPDATE и DELETE создают мертвые строки (dead tuples)

Table growth без VACUUM:
┌─────────────────────────────────────┐
│ Row 1 (dead)  xmin=100 xmax=101     │
│ Row 1 (dead)  xmin=101 xmax=102     │
│ Row 1 (active) xmin=102 xmax=null   │
│                                     │
│ Размер таблицы растет бесконечно!   │
└─────────────────────────────────────┘

После VACUUM:
┌─────────────────────────────────────┐
│ Row 1 (active) xmin=102 xmax=null   │
│ [Free Space Map]                    │
│ (место для reuse)                   │
└─────────────────────────────────────┘
```

**Типы VACUUM:**

| Тип | Что делает | Когда использовать |
|-----|------------|-------------------|
| **VACUUM** | Помечает dead tuples как reusable | Регулярно (autovacuum) |
| **VACUUM FULL** | Полная реорганизация, минимальный размер | Редко, требует lock |
| **VACUUM ANALYZE** | + обновление статистики | После bulk load |

**Autovacuum настройки:**
```sql
-- Когда запускать vacuum
autovacuum_vacuum_threshold = 50  -- минимум 50 dead tuples
autovacuum_vacuum_scale_factor = 0.2  -- + 20% от размера таблицы

-- Для больших таблиц лучше:
ALTER TABLE big_table SET (
  autovacuum_vacuum_scale_factor = 0.01,
  autovacuum_vacuum_threshold = 1000
);
```

**Transaction ID Wraparound:**
```
XID — 32-bit integer (~4 billion)
После достижения лимита — wraparound!

Решение: 
• Frozen XID — специальное значение означающее "старый"
• VACUUM FREEZE — помечает старые строки как frozen
• autovacuum_freeze_max_age — автоматический freeze

Опасность: Если не freeze — data loss при wraparound!
```

---

### 2. PostgreSQL Index Types

**Вопрос:** Сравните типы индексов в PostgreSQL: B-tree, GiST, GIN, BRIN, Hash.

**Ответ:**

| Тип | Структура | Use Case | Пример |
|-----|-----------|----------|--------|
| **B-tree** | Сбалансированное дерево | Равенство, диапазоны | `WHERE id = 5`, `WHERE age > 18` |
| **Hash** | Хеш-таблица | Только равенство | `WHERE hash_code = 'abc'` |
| **GiST** | Обобщенное дерево поиска | Геометрия, близость | `WHERE point <@ circle` |
| **GIN** | Обобщенный инвертированный индекс | Массивы, JSONB, полнотекст | `WHERE tags @> '{"sql"}'` |
| **SP-GiST** | Пространственное разбиение | KD-деревья, префиксные | IP ranges, quad trees |
| **BRIN** | Блочный диапазон | Огромные таблицы с естественным порядком | Time-series data |

**B-tree (по умолчанию):**
```sql
CREATE INDEX idx_users_email ON users(email);

-- Поддерживает:
WHERE email = 'test@example.com'        -- Equality
WHERE email > 'a' AND email < 'b'       -- Range
WHERE email LIKE 'test%'                -- Prefix (только)
ORDER BY email                          -- Sorting
```

**GIN (Generalized Inverted Index):**
```sql
-- JSONB
CREATE INDEX idx_data_gin ON documents USING GIN (data);
SELECT * FROM documents WHERE data @> '{"tags": ["important"]}';

-- Arrays
CREATE INDEX idx_tags ON posts USING GIN (tags);
SELECT * FROM posts WHERE tags && ARRAY['sql', 'postgres']; -- overlap

-- Full-text search
CREATE INDEX idx_search ON articles USING GIN (to_tsvector('english', content));
```

**GiST (Generalized Search Tree):**
```sql
-- Геометрические данные
CREATE INDEX idx_location ON places USING GiST (coordinates);
SELECT * FROM places WHERE coordinates <@ circle '((0,0), 1000)';

-- Ближайшие соседи (KNN)
SELECT * FROM places 
ORDER BY coordinates <-> point '(0,0)' 
LIMIT 10;
```

**BRIN (Block Range INdex):**
```sql
-- Для очень больших таблиц с естественным порядком
CREATE INDEX idx_logs_time ON logs USING BRIN (created_at);

-- Размер: ~100x меньше чем B-tree
-- Подходит для: time-series, sensor data
-- Не подходит для: случайный доступ, frequent updates
```

**Partial Index:**
```sql
-- Индекс только для активных пользователей (меньше размер)
CREATE INDEX idx_active_users ON users(email) WHERE active = true;

-- Покрывающий индекс (Index Only Scan)
CREATE INDEX idx_users_covering ON users(created_at) INCLUDE (email, name);
```

**Index Only Scan:**
```sql
-- Все данные в индексе — не нужно обращаться к таблице
EXPLAIN ANALYZE SELECT email FROM users WHERE email = 'test@example.com';
-- Index Only Scan using idx_users_email
```

---

### 3. PostgreSQL Replication

**Вопрос:** Объясните streaming replication в PostgreSQL. Чем отличается synchronous от asynchronous?

**Ответ:**

**Streaming Replication:**
```
┌─────────────────────────────────────────────────────────────┐
│                 PostgreSQL Replication                       │
│                                                              │
│   Primary Server                    Standby Server(s)       │
│   ┌─────────────┐                   ┌─────────────┐         │
│   │   WAL       │ ────streaming────►│   WAL       │         │
│   │   Writer    │                   │   Receiver  │         │
│   └──────┬──────┘                   └──────┬──────┘         │
│          │                                  │                │
│          ▼                                  ▼                │
│   ┌─────────────┐                   ┌─────────────┐         │
│   │   Shared    │                   │   Shared    │         │
│   │   Buffers   │                   │   Buffers   │         │
│   └─────────────┘                   └─────────────┘         │
│                                                              │
│   WAL (Write-Ahead Log) — журнал изменений                   │
│   • Каждая модификация сначала пишется в WAL                 │
│   • Стендбай читает WAL и применяет изменения                │
└─────────────────────────────────────────────────────────────┘
```

**Synchronous vs Asynchronous:**

```
Asynchronous Replication (async):
┌────────────────────────────────────────────────────────┐
│ Client                                                 │
│   │                                                    │
│   │ COMMIT                                            │
│   ▼                                                    │
│ Primary ──► WAL written ──► Return OK to client       │
│   │                                                    │
│   │ async streaming                                    │
│   ▼                                                    │
│ Standby (может отставать на несколько транзакций)     │
│                                                        │
│ Latency: минимальна                                    │
│ Risk: potential data loss if primary fails            │
└────────────────────────────────────────────────────────┘

Synchronous Replication (sync):
┌────────────────────────────────────────────────────────┐
│ Client                                                 │
│   │                                                    │
│   │ COMMIT                                            │
│   ▼                                                    │
│ Primary ──► WAL written ──► Standby write ──► OK      │
│                                                        │
│ Latency: выше (сеть + standby write)                  │
│ Guarantee: zero data loss                             │
└────────────────────────────────────────────────────────┘
```

**Настройка:**
```sql
-- postgresql.conf на primary
synchronous_commit = remote_apply  -- или on, remote_write
synchronous_standby_names = 'replica1, replica2'

-- Уровни синхронизации:
-- off       — не ждать WAL write (dangerous)
-- local     — ждать local WAL write
-- remote_write — ждать standby получит WAL
-- on        — ждать standby flush WAL to disk (default)
-- remote_apply — ждать standby применит изменения
```

**Replication Modes:**

| Режим | Гарантия | Производительность |
|-------|----------|-------------------|
| async | Нет | Максимальная |
| sync (1 standby) | Да, но медленно | Ниже |
| sync (quorum) | Да, faster | quorum_commit |
| logical | Гибкость | Накладные расходы |

**Logical Replication:**
```sql
-- Репликация на уровне строк (не WAL байты)
-- Плюсы:
-- • Репликация между версиями PG
-- • Репликация subset of tables
-- • Репликация в разные схемы
-- • Multi-master возможен

-- Настройка:
CREATE PUBLICATION mypub FOR TABLE users, orders;
CREATE SUBSCRIPTION mysub CONNECTION 'host=primary...' PUBLICATION mypub;
```

---

### 4. MySQL InnoDB Internals

**Вопрос:** Объясните архитектуру InnoDB. Чем отличается от PostgreSQL?

**Ответ:**

**InnoDB Architecture:**
```
┌─────────────────────────────────────────────────────────────┐
│                    InnoDB Architecture                       │
│                                                              │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                 Buffer Pool (In-Memory)             │   │
│   │  ┌───────────┐  ┌───────────┐  ┌─────────────────┐  │   │
│   │  │ Data      │  │ Index     │  │ Adaptive Hash   │  │   │
│   │  │ Pages     │  │ Pages     │  │ Index           │  │   │
│   │  └───────────┘  └───────────┘  └─────────────────┘  │   │
│   │  ┌───────────┐  ┌───────────┐                        │   │
│   │  │ Change    │  │ Lock      │                        │   │
│   │  │ Buffer    │  │ Info      │                        │   │
│   │  │ (inserts) │  │           │                        │   │
│   │  └───────────┘  └───────────┘                        │   │
│   └─────────────────────────────────────────────────────┘   │
│                           │                                  │
│                           ▼                                  │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                 Redo Log (ib_logfile)               │   │
│   │  • Circular buffer                                  │   │
│   │  • Durability guarantee                             │   │
│   │  • Write-ahead logging                              │   │
│   └─────────────────────────────────────────────────────┘   │
│                           │                                  │
│                           ▼                                  │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                 Undo Log                            │   │
│   │  • For rollback                                     │   │
│   │  • For MVCC (consistent read)                       │   │
│   └─────────────────────────────────────────────────────┘   │
│                           │                                  │
│                           ▼                                  │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                 Tablespace (.ibd files)             │   │
│   │  • Data and indexes together                        │   │
│   │  • Clustered index = table                          │   │
│   └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**Clustered Index:**
```
InnoDB: PRIMARY KEY = clustered index (data stored with index)

┌─────────────────────────────────────────────────┐
│ PRIMARY KEY Index                               │
│ ┌──────────┬──────────────────────────────────┐ │
│ │ id (PK)  │  row_data (all columns)          │ │
│ ├──────────┼──────────────────────────────────┤ │
│ │    1     │  Alice, 25, alice@example.com... │ │
│ │    2     │  Bob, 30, bob@example.com...     │ │
│ │    3     │  Carol, 28, carol@example.com... │ │
│ └──────────┴──────────────────────────────────┘ │
│                                                 │
│ Secondary Index:                                │
│ ┌──────────┬──────────┐                         │
│ │ email    │ id (PK)  │  ───► lookup in PK     │
│ ├──────────┼──────────┤                         │
│ │a@ex.com  │    1     │                         │
│ │b@ex.com  │    2     │                         │
│ └──────────┴──────────┘                         │
└─────────────────────────────────────────────────┘

Ключевое отличие от PostgreSQL:
• InnoDB: Heap-organized with clustered PK
• PostgreSQL: Heap-organized, indexes point to row versions
```

**Redo Log vs WAL:**

| Аспект | PostgreSQL WAL | InnoDB Redo Log |
|--------|----------------|-----------------|
| Формат | Логические изменения | Физические байты |
| Размер | Может быть большим | Фиксированный (circular) |
| Назначение | Replication + Recovery | Recovery only |
| Архивация | Архивируется (WAL archiving) | Перезаписывается |

**Double Write Buffer:**
```
InnoDB использует 16KB pages
Проблема: partial page write при сбое

Решение: Double Write Buffer
1. Записать page в Double Write Buffer (sequential)
2. Записать page в data file (random)
3. При recovery: если data file corrupted — восстановить из DWB

Отключаемо: innodb_doublewrite = OFF (для SSD с atomic writes)
```

**InnoDB vs PostgreSQL:**

| Feature | PostgreSQL | MySQL/InnoDB |
|---------|------------|--------------|
| MVCC | Row versions | Undo log |
| Vacuum | Требуется | Не требуется |
| Clustered index | Нет | Да (PK) |
| Replication | Physical (WAL) | Logical (binlog) |
| Full-text | GIN index | FTS index |
| JSON | Native JSONB | JSON (text) + functional index |
| Parallel query | Да | Limited |

---

### 5. NoSQL Comparison

**Вопрос:** Сравните MongoDB, Cassandra, DynamoDB. Когда какой выбрать?

**Ответ:**

**MongoDB:**
```javascript
// Document store, JSON-like documents
{
  _id: ObjectId("..."),
  name: "Alice",
  orders: [
    { id: 1, total: 100 },
    { id: 2, total: 200 }
  ],
  address: { city: "NYC", zip: "10001" }
}

// Flexible schema
// ACID transactions (since 4.0)
// Rich query language
// Single-node or replica set or sharded cluster
```

**Cassandra:**
```sql
-- Wide-column store
CREATE TABLE users (
  user_id UUID PRIMARY KEY,
  email TEXT,
  created_at TIMESTAMP
);

-- Optimized for write-heavy workloads
-- Linear scalability
-- Eventually consistent (tunable)
-- CQL (SQL-like but limited)
-- No joins, no subqueries
```

**DynamoDB:**
```javascript
// Fully managed key-value + document
{
  "PK": "USER#123",      // Partition Key
  "SK": "PROFILE",       // Sort Key
  "name": "Alice",
  "email": "alice@example.com"
}

// Single-digit millisecond latency
// Automatic scaling
// Pay per request or provisioned capacity
// Limited query flexibility (must know PK)
```

**Сравнение:**

| Критерий | MongoDB | Cassandra | DynamoDB |
|----------|---------|-----------|----------|
| **Модель** | Document | Wide-column | Key-value |
| **Масштабирование** | Horizontal | Linear horizontal | Auto |
| **Консистентность** | Strong | Tunable | Eventual/Strong |
| **Запросы** | Rich | Limited | Key-based |
| **Joins** | $lookup | Нет | Нет |
| **ACID** | Да (multi-doc) | Batch | TransactWriteItems |
| **Managed** | Self/Atlas | Self/DataStax | AWS only |
| **Best for** | Flexible data | Write-heavy | Simple, fast |

**Когда MongoDB:**
- Быстро меняющаяся схема
- Сложные документы (вложенные)
- Aggregation pipelines
- Need for ad-hoc queries

**Когда Cassandra:**
- Огромные объемы записи (time-series)
- Географическое распределение
- High availability критична
- Простые запросы, известные заранее

**Когда DynamoDB:**
- AWS infrastructure
- Predictable traffic patterns
- Key-value access patterns
- Serverless applications
- Low operational overhead

**Data Modeling:**

```
MongoDB — "What queries do I need?"
├── Denormalize where beneficial
├── Embedded documents for 1:1, 1:few
└── References for 1:many, many:many

Cassandra — "Query-first design"
├── Denormalize everything
├── One table per query pattern
└── Partition key = group data together

DynamoDB — "Single Table Design"
├── One table for all entities
├── PK/SK patterns for access
└── GSIs for alternate access patterns
```

---

### 6. Redis Data Structures

**Вопрос:** Объясните основные структуры данных Redis и их применение.

**Ответ:**

**Strings:**
```redis
SET user:1:name "Alice"
GET user:1:name
MSET user:2:name "Bob" user:3:name "Carol"
INCR views:page:1  -- atomic counter
SETEX session:abc 3600 "data"  -- с TTL
```
**Use cases:** Caching, counters, rate limiting, sessions

**Lists:**
```redis
LPUSH queue:jobs "job1"
RPUSH queue:jobs "job2"
LPOP queue:jobs  -- FIFO queue
BLPOP queue:jobs 30  -- blocking pop (wait 30 sec)
LRANGE queue:jobs 0 -1  -- get all
```
**Use cases:** Queues, activity feeds, message history

**Sets:**
```redis
SADD tags:post:1 "redis" "database" "cache"
SISMEMBER tags:post:1 "redis"  -- check membership
SINTER tags:post:1 tags:post:2  -- common tags
SUNION tags:post:1 tags:post:2  -- all tags
SPOP tags:post:1  -- random element
```
**Use cases:** Tags, unique visitors, relationships

**Sorted Sets:**
```redis
ZADD leaderboard 100 "Alice" 95 "Bob" 98 "Carol"
ZREVRANGE leaderboard 0 2 WITHSCORES  -- top 3
ZRANGEBYSCORE leaderboard 90 100  -- range query
ZINCRBY leaderboard 5 "Alice"  -- increment score
```
**Use cases:** Leaderboards, time-series, priority queues

**Hashes:**
```redis
HSET user:1 name "Alice" age 25 email "alice@example.com"
HGET user:1 name
HGETALL user:1
HMGET user:1 name email
HINCRBY user:1 visits 1
```
**Use cases:** Objects, user profiles, aggregated data

**Bitmaps:**
```redis
SETBIT active:2024-01-01 123 1  -- user 123 active on Jan 1
BITCOUNT active:2024-01-01  -- count active users
BITOP AND active:week active:day1 active:day2 ...
```
**Use cases:** DAU/MAU tracking, feature flags, presence

**HyperLogLog:**
```redis
PFADD visitors:page1 "user1" "user2" "user3"
PFCOUNT visitors:page1  -- approximate count (0.81% error)
PFMERGE visitors:total visitors:page1 visitors:page2
```
**Use cases:** Cardinality estimation, unique counts (memory efficient)

**Streams:**
```redis
XADD events * type "click" user "123" page "/home"
XREAD COUNT 10 STREAMS events 0  -- read from beginning
XGROUP CREATE events group1 $  -- consumer group
XREADGROUP GROUP group1 consumer1 STREAMS events >
```
**Use cases:** Event sourcing, message queues, log aggregation

**Geospatial:**
```redis
GEOADD places -74.006 40.7128 "NYC" -118.2437 34.0522 "LA"
GEODIST places "NYC" "LA" km
GEORADIUS places -74.006 40.7128 100 km WITHDIST
```
**Use cases:** Location-based services, nearby search

**JSON (RedisJSON module):**
```redis
JSON.SET user:1 $ '{"name": "Alice", "age": 25}'
JSON.GET user:1 $.name
JSON.NUMINCRBY user:1 $.age 1
JSON.ARRAPPEND user:1 $.tags "premium"
```

---

### 7. Redis Persistence

**Вопрос:** Сравните RDB и AOF persistence в Redis. Как выбрать?

**Ответ:**

```
┌─────────────────────────────────────────────────────────────┐
│                  Redis Persistence                           │
│                                                              │
│  RDB (Redis Database Backup):                                │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • Point-in-time snapshots                               ││
│  │ • Binary dump of dataset                                ││
│  │ • Fork child process for snapshotting                   ││
│  │                                                         ││
│  │ save 900 1      -- save if 1+ changes in 900 sec       ││
│  │ save 300 10     -- save if 10+ changes in 300 sec      ││
│  │ save 60 10000   -- save if 10000+ changes in 60 sec    ││
│  │                                                         ││
│  │ Pros:                                                   ││
│  │   ✓ Compact file                                        ││
│  │   ✓ Fast restore                                        ││
│  │   ✓ Good for backups                                    ││
│  │                                                         ││
│  │ Cons:                                                   ││
│  │   ✗ Data loss (last save to crash)                     ││
│  │   ✗ Fork overhead (memory)                             ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  AOF (Append Only File):                                     │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • Log of every write operation                          ││
│  │ • Append-only (safe)                                    ││
│  │ • Can replay log to reconstruct state                   ││
│  │                                                         ││
│  │ appendfsync always   -- every write (slow, safest)     ││
│  │ appendfsync everysec -- every second (default)         ││
│  │ appendfsync no       -- OS decides (fast, unsafe)      ││
│  │                                                         ││
│  │ AOF Rewrite:                                            ││
│  │   • Compact AOF by creating new minimal version        ││
│  │   • Triggered when file too big                        ││
│  │   • Fork + incremental writes                          ││
│  │                                                         ││
│  │ Pros:                                                   ││
│  │   ✓ Durability (1 sec max loss)                        ││
│  │   ✓ Human-readable (sort of)                           ││
│  │   ✓ Append-only = safe                                 ││
│  │                                                         ││
│  │ Cons:                                                   ││
│  │   ✗ Larger file                                         ││
│  │   ✗ Slower restore (replay log)                        ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Рекомендация: Both!                                         │
│  • RDB for fast restarts and backups                        │
│  • AOF for durability                                       │
└─────────────────────────────────────────────────────────────┘
```

**Выбор стратегии:**

| Требование | Стратегия |
|------------|-----------|
| Максимальная производительность | RDB only |
| Минимум потери данных | AOF (always) |
| Баланс | RDB + AOF (everysec) |
| Disable persistence | For pure cache only |

**Hybrid Mode (Redis 4.0+):**
```
• RDB + AOF combined
• AOF contains: RDB prefix + incremental AOF
• Fast restore (RDB) + durability (AOF)
```

---

### 8. Redis Clustering

**Вопрос:** Объясните Redis Cluster. Как работает sharding и failover?

**Ответ:**

```
┌─────────────────────────────────────────────────────────────┐
│                    Redis Cluster                             │
│                                                              │
│  Architecture (3 masters, 3 replicas):                      │
│                                                              │
│        ┌─────────┐            ┌─────────┐                   │
│        │ Master  │◄──────────►│ Master  │                   │
│        │  A:0-5460│           │  B:5461-10922│              │
│        └────┬────┘            └────┬────┘                   │
│             │                      │                         │
│        ┌────▼────┐            ┌────▼────┐                   │
│        │ Replica │            │ Replica │                   │
│        │   A'    │            │   B'    │                   │
│        └─────────┘            └─────────┘                   │
│                                                              │
│                    ┌─────────┐                              │
│                    │ Master  │                              │
│                    │C:10923-16383│                          │
│                    └────┬────┘                              │
│                    ┌────▼────┐                              │
│                    │ Replica │                              │
│                    │   C'    │                              │
│                    └─────────┘                              │
│                                                              │
│  Sharding:                                                   │
│  • 16384 hash slots                                          │
│  • CRC16(key) % 16384 → slot                                 │
│  • Each master owns subset of slots                          │
│                                                              │
│  Client routing:                                             │
│  • MOVED redirect if wrong node                              │
│  • Client caches slot → node mapping                         │
└─────────────────────────────────────────────────────────────┘
```

**Sharding:**
```
Key → CRC16 hash → Slot 0-16383 → Node

{user:1}.name  → hash → Slot 9182 → Node B
{order:100}    → hash → Slot 2048 → Node A

Hash Tags: {user:1}:name и {user:1}:orders → один slot
(гарантия что ключи вместе)
```

**Failover:**
```
1. Master A fails
2. Replicas detect failure (via PING/PONG timeout)
3. Replicas vote for new master
4. Replica A' promoted to master
5. Cluster reconfiguration
6. Clients updated (MOVED redirects)

Requirements for failover:
• Majority of masters available
• Replica has recent data (replication offset)
```

**Configuration:**
```bash
# Create cluster
redis-cli --cluster create \
  192.168.1.1:7000 192.168.1.2:7000 192.168.1.3:7000 \
  192.168.1.1:7001 192.168.1.2:7001 192.168.1.3:7001 \
  --cluster-replicas 1
```

**Redis Sentinel (alternative to Cluster):**
```
• High availability for single master + replicas
• No automatic sharding
• Monitors, notifies, auto-failover, configuration provider
• Good for: HA without sharding needs
```

**Consistency Trade-offs:**
```
Redis Cluster:
• Asynchronous replication (potential data loss)
• No strong consistency guarantee
• Write to master, read from master = safe
• Write to master, read from replica = eventual consistency
```

---

### 9. Redis Use Cases

**Вопрос:** Расскажите о типичных паттернах использования Redis.

**Ответ:**

**1. Session Store:**
```javascript
// Set session with TTL
redis.setex(`session:${sessionId}`, 3600, JSON.stringify(userData));

// Get session
const data = await redis.get(`session:${sessionId}`);
if (!data) return null; // Session expired
return JSON.parse(data);
```

**2. Rate Limiting (Sliding Window):**
```javascript
// Token bucket or sliding window log
async function isAllowed(userId) {
  const key = `rate:${userId}`;
  const now = Date.now();
  const window = 60000; // 1 minute
  
  // Remove old entries
  await redis.zremrangebyscore(key, 0, now - window);
  
  // Count current
  const count = await redis.zcard(key);
  
  if (count < LIMIT) {
    await redis.zadd(key, now, now);
    await redis.expire(key, 60);
    return true;
  }
  return false;
}
```

**3. Distributed Lock:**
```javascript
// Redlock algorithm (simplified)
async function acquireLock(resource, ttl) {
  const token = uuid();
  const result = await redis.set(
    `lock:${resource}`, 
    token, 
    'NX', // Only if not exists
    'EX', 
    ttl
  );
  
  if (result === 'OK') {
    return { release: () => redis.del(`lock:${resource}`) };
  }
  return null;
}
```

**4. Real-time Leaderboard:**
```javascript
// Update scores
await redis.zincrby('game:leaderboard', points, userId);

// Get top 10
const top = await redis.zrevrange('game:leaderboard', 0, 9, 'WITHSCORES');

// Get user rank
const rank = await redis.zrevrank('game:leaderboard', userId);
```

**5. Pub/Sub:**
```javascript
// Publisher
redis.publish('channel:room:1', JSON.stringify(message));

// Subscriber
redis.subscribe('channel:room:1');
redis.on('message', (channel, message) => {
  console.log(`Received: ${message}`);
});
```

**6. Cache-aside Pattern:**
```javascript
async function getUser(id) {
  // Try cache
  let user = await redis.get(`user:${id}`);
  if (user) return JSON.parse(user);
  
  // Cache miss
  user = await db.users.findById(id);
  if (user) {
    await redis.setex(`user:${id}`, 3600, JSON.stringify(user));
  }
  return user;
}
```

**7. Real-time Analytics:**
```javascript
// Counters
redis.hincrby('stats:hour:2024-01-01-10', 'pageviews', 1);

// Unique visitors (HyperLogLog)
redis.pfadd('visitors:day:2024-01-01', userId);
const count = await redis.pfcount('visitors:day:2024-01-01');
```

---

### 10. Database Scaling Strategies

**Вопрос:** Какие стратегии масштабирования баз данных существуют?

**Ответ:**

```
┌─────────────────────────────────────────────────────────────┐
│                Database Scaling Strategies                   │
│                                                              │
│  Vertical Scaling (Scale Up):                               │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • Больше CPU, RAM, диск                                 ││
│  │ • Просто, но ограничено физикой                         ││
│  │ • Дорого на высоких уровнях                             ││
│  │ • Единая точка отказа                                   ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Horizontal Scaling (Scale Out):                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ Read Replicas:                                          ││
│  │   • Async replication                                   ││
│  │   • Read traffic распределяется                         ││
│  │   • eventual consistency                                ││
│  │                                                         ││
│  │ Sharding (Partitioning):                                ││
│  │   • Разделение данных по серверам                       ││
│  │   • Application-level или DB-level                      ││
│  │   • Сложность cross-shard queries                       ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Functional Partitioning:                                   │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • Разные базы для разных доменов                        ││
│  │ • Users DB, Orders DB, Products DB                      ││
│  │ • Микросервисы                                          ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Caching:                                                   │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • Redis/Memcached перед БД                              ││
│  │ • Уменьшает нагрузку на БД                              ││
│  │ • Cache-aside, write-through, read-through              ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  CQRS (Command Query Responsibility Segregation):           │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • Отдельные модели для записи и чтения                  ││
│  │ • Write: normalized DB                                  ││
│  │ • Read: denormalized, optimized for queries             ││
│  │ • Event sourcing для синхронизации                      ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

**Read Replica Strategy:**
```
Master ──► Replica 1 (read)
    │
    └────► Replica 2 (read)
    │
    └────► Replica 3 (backup + read)

Connection routing:
- Writes → Master
- Reads → Replicas (load balanced)
- Sticky sessions for read-your-writes
```

**Sharding Strategies:**

| Стратегия | Пример | Плюсы | Минусы |
|-----------|--------|-------|--------|
| **Hash** | user_id % 4 | Равномерно | Resharding сложный |
| **Range** | id 1-1M, 1M-2M... | Range queries | Hotspots |
| **List** | US → shard1, EU → shard2 | Geographic | Manual management |
| **Directory** | Lookup table | Гибкость | Single point |

**Multi-Master:**
```
Master A ◄────► Master B ◄────► Master C
   │               │               │
   └───────────────┴───────────────┘
   
• Конфликтующие writes
• Need conflict resolution
• Rarely used (Galera, CockroachDB)
```

---

## 💻 Практика / Database Tasks

### 1. Query Optimization

**Задача:** Оптимизировать медленный SQL запрос.

**Архитектура анализа:**

```
┌─────────────────────────────────────────────────────────────┐
│              Query Optimization Workflow                     │
│                                                              │
│  1. Identify Slow Query:                                    │
│     • pg_stat_statements (PostgreSQL)                       │
│     • slow_query_log (MySQL)                                │
│     • Application monitoring                                │
│                                                              │
│  2. EXPLAIN ANALYZE:                                        │
│     • Execution plan                                        │
│     • Seq Scan vs Index Scan                                │
│     • Actual vs Estimated rows                              │
│     • Execution time breakdown                              │
│                                                              │
│  3. Common Issues:                                          │
│     • Missing indexes                                       │
│     • Outdated statistics (ANALYZE)                         │
│     • N+1 queries (ORM)                                     │
│     • Inefficient JOINs                                     │
│     • SELECT *                                              │
│     • Functions on indexed columns                          │
│                                                              │
│  4. Optimization Techniques:                                │
│     • Add composite index                                   │
│     • Covering index (INCLUDE)                              │
│     • Partial index                                         │
│     • Query rewrite                                         │
│     • Partitioning                                          │
│     • Denormalization                                       │
└─────────────────────────────────────────────────────────────┘
```

---

### 2. Database Migration Strategy

**Задача:** Спланировать zero-downtime миграцию схемы.

**Архитектура решения:**

```
┌─────────────────────────────────────────────────────────────┐
│             Zero-Downtime Migration Strategy                 │
│                                                              │
│  Problem: Adding NOT NULL column to large table             │
│                                                              │
│  Wrong way (blocking):                                      │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ ALTER TABLE users ADD COLUMN phone VARCHAR(20)         ││
│  │   NOT NULL;  -- Blocks table!                          ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Right way (online):                                        │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ Step 1: Add nullable column                             ││
│  │   ALTER TABLE users ADD COLUMN phone VARCHAR(20);      ││
│  │                                                         ││
│  │ Step 2: Backfill in batches                             ││
│  │   UPDATE users SET phone = 'unknown'                   ││
│  │   WHERE phone IS NULL AND id BETWEEN x AND y;          ││
│  │   (Repeat for all batches)                             ││
│  │                                                         ││
│  │ Step 3: Add default for new inserts                     ││
│  │   ALTER TABLE users ALTER COLUMN phone                 ││
│  │     SET DEFAULT 'unknown';                             ││
│  │                                                         ││
│  │ Step 4: Add constraint (validate first)                 ││
│  │   ALTER TABLE users VALIDATE CONSTRAINT phone_not_null;││
│  │   ALTER TABLE users ALTER COLUMN phone SET NOT NULL;   ││
│  │                                                         ││
│  │ PostgreSQL 11+ Add column with default:                ││
│  │   ALTER TABLE users ADD COLUMN phone VARCHAR(20)       ││
│  │     NOT NULL DEFAULT 'unknown';  -- Fast!              ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Expand/Contract Pattern:                                   │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ Phase 1 - Expand:                                       ││
│  │   • Add new column/table                                ││
│  │   • Dual-write (old and new)                            ││
│  │   • Backfill data                                       ││
│  │                                                         ││
│  │ Phase 2 - Transition:                                   ││
│  │   • Switch reads to new                                 ││
│  │   • Verify consistency                                  ││
│  │                                                         ││
│  │ Phase 3 - Contract:                                     ││
│  │   • Stop writing to old                                 ││
│  │   • Remove old column/table                             ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

---

### 3. Replication Setup

**Задача:** Спроектировать архитектуру репликации с failover.

**Архитектура решения:**

```
┌─────────────────────────────────────────────────────────────┐
│              High Availability with Replication              │
│                                                              │
│  Architecture:                                               │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                                                         ││
│  │                    Application                          ││
│  │                         │                               ││
│  │                         ▼                               ││
│  │              ┌─────────────────────┐                    ││
│  │              │   Connection Pool   │                    ││
│  │              │   or Proxy          │                    ││
│  │              │   (PgBouncer/HAProxy)│                   ││
│  │              └──────────┬──────────┘                    ││
│  │                         │                               ││
│  │              ┌──────────┴──────────┐                    ││
│  │              │                     │                    ││
│  │              ▼                     ▼                    ││
│  │        ┌─────────┐          ┌─────────┐                 ││
│  │        │Primary  │◄────────►│ Standby │                 ││
│  │        │(Master) │  sync    │(Replica)│                 ││
│  │        └────┬────┘          └────┬────┘                 ││
│  │             │                    │                      ││
│  │             └────────┬───────────┘                      ││
│  │                      │                                  ││
│  │                      ▼                                  ││
│  │              ┌─────────────┐                            ││
│  │              │   Patroni   │                            ││
│  │              │  (or repmgr)│                            ││
│  │              └─────────────┘                            ││
│  │                      │                                  ││
│  │                      ▼                                  ││
│  │              ┌─────────────┐                            ││
│  │              │    etcd     │                            ││
│  │              │ (DCS for HA)│                            ││
│  │              └─────────────┘                            ││
│  │                                                         ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Failover Process:                                          │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ 1. Primary failure detected                             ││
│  │ 2. Leader election (quorum)                             ││
│  │ 3. Promote best standby to primary                      ││
│  │ 4. Update leader in DCS                                   ││
│  │ 5. Reconfigure remaining standbys                         ││
│  │ 6. Update connection pool/proxy                           ││
│  │ 7. Old primary becomes standby (when recovered)          ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Connection Handling:                                       │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • Read-write splitter (automatic routing)               ││
│  │ • Separate endpoints for read and write                 ││
│  │ • Retry logic with exponential backoff                  ││
│  │ • Circuit breaker for failed connections                ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

---

### 4. Caching Strategy

**Задача:** Спроектировать multi-tier кеширование.

**Архитектура решения:**

```
┌─────────────────────────────────────────────────────────────┐
│                Multi-Tier Caching Strategy                   │
│                                                              │
│  Application-level (L1):                                    │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • In-memory (Node.js Map/LRU)                           ││
│  │ • Per-process, not shared                               ││
│  │ • Very fast (μs)                                        ││
│  │ • Short TTL (seconds)                                   ││
│  │                                                         ││
│  │ Use cases:                                              ││
│  │   • Config values                                       ││
│  │   • User sessions (if sticky)                           ││
│  │   • Hot data                                            ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Distributed Cache (L2):                                    │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • Redis/Memcached cluster                               ││
│  │ • Shared across app instances                           ││
│  │ • Fast (ms)                                             ││
│  │ • Medium TTL (minutes-hours)                            ││
│  │                                                         ││
│  │ Patterns:                                               ││
│  │   • Cache-aside                                         ││
│  │   • Write-through                                       ││
│  │   • Write-behind                                        ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Database (L3):                                             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ • Query cache (если поддерживается)                     ││
│  │ • Buffer pool                                           ││
│  │ • SSD/NVMe storage                                      ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Cache Invalidation Strategies:                             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ TTL-based:                                              ││
│  │   • Set expiration on write                             ││
│  │   • Simple, eventual consistency                        ││
│  │                                                         ││
│  │ Event-based:                                            ││
│  │   • Invalidate on data change                           ││
│  │   • Pub/Sub for cache invalidation                      ││
│  │                                                         ││
│  │ Write-through:                                          ││
│  │   • Update cache when updating DB                       ││
│  │   • Cache always fresh                                  ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Handling Cache Issues:                                     │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ Cache Stampede:                                        ││
│  │   • Lock while recomputing                              ││
│  │   • Probabilistic early expiration                      ││
│  │                                                         ││
│  │ Thundering Herd:                                        ││
│  │   • Request coalescing                                  ││
│  │   • Per-instance cache before distributed               ││
│  │                                                         ││
│  │ Cold Start:                                             ││
│  │   • Cache warming on startup                            ││
│  │   • Lazy loading                                        ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

---

### 5. Data Modeling for NoSQL

**Задача:** Спроектировать схему для MongoDB/Cassandra/DynamoDB.

**Архитектура решения:**

```
┌─────────────────────────────────────────────────────────────┐
│              NoSQL Data Modeling                             │
│                                                              │
│  MongoDB (E-commerce):                                      │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ {                                                       ││
│  │   _id: ObjectId("..."),                                 ││
│  │   name: "Gaming Laptop",                                ││
│  │   price: 1299.99,                                       ││
│  │   inventory: {                                          ││
│  │     warehouse_1: 10,                                    ││
│  │     warehouse_2: 5                                      ││
│  │   },                                                    ││
│  │   reviews: [  -- Embedded 1:few                        ││
│  │     { user_id: "...", rating: 5, text: "Great!" }      ││
│  │   ],                                                    ││
│  │   category_id: ObjectId("...")  -- Reference 1:many    ││
│  │ }                                                       ││
│  │                                                         ││
│  │ Паттерны:                                               ││
│  │ • Embedding: reviews (few per product)                  ││
│  │ • Referencing: categories (many products share)         ││
│  │ • Bucketing: monthly aggregation                        ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Cassandra (Time-series):                                   │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ CREATE TABLE sensor_readings (                          ││
│  │   sensor_id TEXT,                                       ││
│  │   date TEXT,  -- Partition key                          ││
│  │   timestamp TIMESTAMP,  -- Clustering key               ││
│  │   temperature DOUBLE,                                   ││
│  │   humidity DOUBLE,                                      ││
│  │   PRIMARY KEY ((sensor_id, date), timestamp)            ││
│  │ ) WITH CLUSTERING ORDER BY (timestamp DESC);           ││
│  │                                                         ││
│  │ Query:                                                  ││
│  │ SELECT * FROM sensor_readings                           ││
│  │ WHERE sensor_id = 'sensor1' AND date = '2024-01-01'    ││
│  │ ORDER BY timestamp DESC LIMIT 100;                     ││
│  │                                                         ││
│  │ Паттерны:                                               ││
│  │ • One table per query pattern                           ││
│  │ • Denormalization                                       ││
│  │ • Time-based partitioning                               ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  DynamoDB (Single Table Design):                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ Entity: User                                            ││
│  │ PK: USER#123, SK: PROFILE                               ││
│  │ { name, email, created_at }                             ││
│  │                                                         ││
│  │ Entity: Order                                           ││
│  │ PK: USER#123, SK: ORDER#456                             ││
│  │ { order_id, total, status, items }                      ││
│  │                                                         ││
│  │ Access Patterns:                                        ││
│  │ 1. Get user: PK=USER#123, SK=PROFILE                    ││
│  │ 2. Get user's orders: PK=USER#123, SK begins ORDER#     ││
│  │ 3. Get order by ID: GSI PK=ORDER#456, SK=ORDER#456     ││
│  │                                                         ││
│  │ Паттерны:                                               ││
│  │ • Composite keys (PK, SK)                               ││
│  │ • GSI for alternate access                              ││
│  │ • Overloading attributes                                ││
│  │ • Adjacency list pattern                                ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```
