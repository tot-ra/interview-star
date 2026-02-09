
---

## 10. Databases Deep Dive

### 📚 Теория (10 вопросов)

1. **PostgreSQL MVCC:** Объясните как работает MVCC в PostgreSQL и почему нужен VACUUM. [➜](10-Databases.md#1-postgresql-mvcc)
2. **PostgreSQL Index Types:** Сравните типы индексов в PostgreSQL: B-tree, GiST, GIN, BRIN, Hash. [➜](10-Databases.md#2-postgresql-index-types)
3. **PostgreSQL Replication:** Объясните streaming replication в PostgreSQL. Чем отличается synchronous от asynchronous? [➜](10-Databases.md#3-postgresql-replication)
4. **MySQL InnoDB Internals:** Объясните архитектуру InnoDB. Чем отличается от PostgreSQL? [➜](10-Databases.md#4-mysql-innodb-internals)
5. **NoSQL Comparison:** Сравните MongoDB, Cassandra, DynamoDB. Когда какой выбрать? [➜](10-Databases.md#5-nosql-comparison)
6. **Redis Data Structures:** Объясните основные структуры данных Redis и их применение. [➜](10-Databases.md#6-redis-data-structures)
7. **Redis Persistence:** Сравните RDB и AOF persistence в Redis. Как выбрать? [➜](10-Databases.md#7-redis-persistence)
8. **Redis Clustering:** Объясните Redis Cluster. Как работает sharding и failover? [➜](10-Databases.md#8-redis-clustering)
9. **Redis Use Cases:** Расскажите о типичных паттернах использования Redis. [➜](10-Databases.md#9-redis-use-cases)
10. **Database Scaling Strategies:** Какие стратегии масштабирования баз данных существуют? [➜](10-Databases.md#10-database-scaling-strategies)

### 💻 Практика / Database Tasks (5 задач)

1. **Query Optimization:** Оптимизируйте медленный SQL запрос. [➜](10-Databases.md#1-query-optimization)
2. **Database Migration Strategy:** Спланируйте zero-downtime миграцию схемы. [➜](10-Databases.md#2-database-migration-strategy)
3. **Sharding Implementation:** Спроектируйте шардирование для высоконагруженной системы. [➜](10-Databases.md#3-sharding-implementation)
4. **Cache Strategy:** Спроектируйте многоуровневое кеширование с Redis. [➜](10-Databases.md#4-cache-strategy)
5. **Replication Setup:** Настройте master-slave репликацию с failover. [➜](10-Databases.md#5-replication-setup)

---

## 11. Go Programming

### 📚 Теория (10 вопросов)

1. **Goroutines и Channels:** Объясните, как работают goroutines и channels в Go. В чем разница между buffered и unbuffered channels? [➜](11-Go.md#1-goroutines-и-channels)
2. **Interfaces:** Как работают interfaces в Go? Объясните duck typing и empty interface. [➜](11-Go.md#2-interfaces)
3. **Memory Management:** Как Go управляет памятью? Объясните garbage collector и escape analysis. [➜](11-Go.md#3-memory-management)
4. **Error Handling:** Как работает обработка ошибок в Go? Объясните error interface и best practices. [➜](11-Go.md#4-error-handling)
5. **Synchronization Primitives:** Объясните sync.Mutex, sync.RWMutex, sync.WaitGroup, sync.Once. Когда использовать каждый? [➜](11-Go.md#5-synchronization-primitives)
6. **Concurrency Patterns:** Опишите common concurrency patterns в Go: worker pool, fan-out/fan-in, pipeline, context cancellation. [➜](11-Go.md#6-concurrency-patterns)
7. **Structs, Methods, Embedding:** Объясните структуры, методы и embedding в Go. В чем разница между value и pointer receivers? [➜](11-Go.md#7-structs-methods-embedding)
8. **Defer, Panic, Recover:** Как работают defer, panic и recover? Какие best practices? [➜](11-Go.md#8-defer-panic-recover)
9. **Testing:** Как писать тесты в Go? Объясните table-driven tests, mocking, benchmarking. [➜](11-Go.md#9-testing)
10. **Go Modules и Tooling:** Как работают Go modules? Объясните go.mod, versioning, vendoring. [➜](11-Go.md#10-go-modules-и-tooling)

### 💻 Практика / Go Tasks (5 задач)

1. **HTTP Server:** Напишите production-ready HTTP сервер с graceful shutdown, middleware и routing. [➜](11-Go.md#1-http-server)
2. **Worker Pool:** Реализуйте worker pool с динамическим масштабированием и контекстом отмены. [➜](11-Go.md#2-worker-pool)
3. **Concurrency Safe Cache:** Создайте thread-safe кеш с TTL (Time-To-Live) eviction. [➜](11-Go.md#3-concurrency-safe-cache)
4. **Pipeline:** Реализуйте data processing pipeline с горутинами. [➜](11-Go.md#4-pipeline)
5. **CLI Tool:** Создайте CLI инструмент с подкомандами и конфигурацией. [➜](11-Go.md#5-cli-tool)
