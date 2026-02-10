# Топ-вопросы для собеседования на позицию Senior Software Engineer

Данный список вопросов составлен на основе требований к позициям Senior/Staff Engineer, включая Fullstack, Backend (Node.js) и Frontend (React), а также навыки работы с инфраструктурой (Docker, K8S, AWS) и знание Applied AI/ML.

## 📋 Содержание

- [1. JavaScript / TypeScript](01-JavaScript-TypeScript.md)
- [2. Алгоритмы и Структуры Данных](02-Алгоритмы-Структуры-Данных.md)
- [3. Frontend (React / UI)](03-Frontend-React.md)
- [4. Backend (Node.js / Databases)](04-Backend-NodeJS-Databases.md)
- [5. System Design](05-System-Design.md)
- [6. Web Architecture & Performance](06-Web-Architecture-Performance.md)
- [7. Инфраструктура и DevOps](07-Infrastructure-DevOps.md)
- [8. Applied AI / ML Systems](08-Applied-AI-ML.md)
- [9. Behavioral / Leadership](09-Behavioral-Leadership.md)
- [10. Databases Deep Dive](10-Databases.md)
- [11. Go Programming](11-Go.md)

---

## 1. JavaScript / TypeScript

### 📚 Теория (10 вопросов)

1. **Event Loop:** Объясните, как работает Event Loop в JavaScript. Что такое Call Stack, Task Queue и Microtask Queue? [➜](01-JavaScript-TypeScript.md#1-event-loop)
2. **Closures:** Что такое замыкания (closures) и как они работают? Приведите практический пример использования. [➜](01-JavaScript-TypeScript.md#2-closures-замыкания)
3. **Prototypes:** Объясните прототипное наследование в JavaScript. Чем оно отличается от классического ООП? [➜](01-JavaScript-TypeScript.md#3-prototypes-прототипное-наследование)
4. **this:** Как работает ключевое слово `this` в разных контекстах (обычная функция, стрелочная функция, метод объекта, конструктор)? [➜](01-JavaScript-TypeScript.md#4-контекст-this)
5. **Promises:** Объясните различия между `Promise.all()`, `Promise.race()`, `Promise.any()` и `Promise.allSettled()`. [➜](01-JavaScript-TypeScript.md#5-promises)
6. **TypeScript Generics:** Объясните generics в TypeScript. Когда и зачем их использовать? [➜](01-JavaScript-TypeScript.md#6-typescript-generics)
7. **TypeScript Utility Types:** Объясните и продемонстрируйте использование `Pick`, `Omit`, `Exclude`, `ReturnType`, `Partial`, `Required`. [➜](01-JavaScript-TypeScript.md#7-typescript-utility-types)
8. **Interface vs Type:** В чем разница между `interface` и `type` в TypeScript? Когда использовать каждый? [➜](01-JavaScript-TypeScript.md#8-interface-vs-type)
9. **Coercion:** Как работает приведение типов в JavaScript? Приведите примеры явного и неявного приведения. [➜](01-JavaScript-TypeScript.md#9-type-coercion-приведение-типов)
10. **Modules:** Объясните разницу между CommonJS и ES Modules. Как работает tree-shaking? [➜](01-JavaScript-TypeScript.md#10-modules-commonjs-vs-es-modules)

### 💻 Практика / Live Coding (5 задач)

1. **Throttle/Debounce:** Напишите реализацию `throttle` и `debounce` функций с нуля. [➜](01-JavaScript-TypeScript.md#1-throttle-и-debounce)
2. **Promise Implementation:** Реализуйте упрощенную версию `Promise.all()`. [➜](01-JavaScript-TypeScript.md#2-promiseall-implementation)
3. **Deep Clone:** Напишите функцию глубокого клонирования объекта с обработкой циклических ссылок. [➜](01-JavaScript-TypeScript.md#3-deep-clone)
4. **Flatten Array:** Реализуйте функцию `flatten(arr, depth)` для многомерных массивов. [➜](01-JavaScript-TypeScript.md#4-flatten-array)
5. **Event Emitter:** Реализуйте паттерн "Наблюдатель" (Observer/EventEmitter) на чистом JavaScript/TypeScript. [➜](01-JavaScript-TypeScript.md#5-event-emitter)

---

## 2. Алгоритмы и Структуры Данных

### 📚 Теория (10 вопросов)

1. **Big O Notation:** Объясните временную и пространственную сложность. Приведите примеры O(1), O(n), O(log n), O(n²). [➜](02-Алгоритмы-Структуры-Данных.md#1-big-o-notation)
2. **Hash Tables:** Как работают хеш-таблицы? Что такое коллизии и как их разрешать? [➜](02-Алгоритмы-Структуры-Данных.md#2-hash-tables)
3. **Trees:** Объясните разницу между Binary Tree, BST, AVL и Red-Black Tree. Когда какую использовать? [➜](02-Алгоритмы-Структуры-Данных.md#3-trees)
4. **Graphs:** Объясните BFS и DFS. Когда какой алгоритм предпочтительнее? [➜](02-Алгоритмы-Структуры-Данных.md#4-graphs)
5. **Sorting:** Сравните QuickSort, MergeSort и HeapSort по времени, памяти и стабильности. [➜](02-Алгоритмы-Структуры-Данных.md#5-sorting)
6. **Dynamic Programming:** Объясните концепцию DP. Чем отличается memoization от tabulation? [➜](02-Алгоритмы-Структуры-Данных.md#6-dynamic-programming)
7. **Linked Lists:** В чем преимущества и недостатки связанных списков по сравнению с массивами? [➜](02-Алгоритмы-Структуры-Данных.md#7-linked-lists)
8. **Stacks & Queues:** Объясните применение стеков и очередей. Приведите реальные примеры использования. [➜](02-Алгоритмы-Структуры-Данных.md#8-stacks--queues)
9. **Heap:** Как работает куча (heap)? Где применяется Priority Queue? [➜](02-Алгоритмы-Структуры-Данных.md#9-heap)
10. **Amortized Analysis:** Что такое амортизированный анализ? Приведите пример с динамическим массивом. [➜](02-Алгоритмы-Структуры-Данных.md#10-amortized-analysis)

### 💻 Практика / Live Coding (5 задач)

1. **LRU Cache:** Реализуйте LRU Cache с O(1) сложностью для операций get и put. [➜](02-Алгоритмы-Структуры-Данных.md#1-lru-cache)
2. **Longest Palindrome:** Напишите функцию, которая находит самый длинный палиндром в строке. [➜](02-Алгоритмы-Структуры-Данных.md#2-longest-palindrome)
3. **Two Sum / Three Sum:** Решите задачу поиска пар/троек с заданной суммой. [➜](02-Алгоритмы-Структуры-Данных.md#3-two-sum--three-sum)
4. **Binary Search Variations:** Реализуйте бинарный поиск для rotated sorted array. [➜](02-Алгоритмы-Структуры-Данных.md#4-binary-search-variations)
5. **Tree Traversal:** Реализуйте in-order, pre-order, post-order обход дерева итеративно (без рекурсии). [➜](02-Алгоритмы-Структуры-Данных.md#5-tree-traversal)

---

## 3. Frontend (React / UI)

### 📚 Теория (10 вопросов)

1. **Virtual DOM:** Как работает Virtual DOM в React? Объясните процесс reconciliation. [➜](03-Frontend-React.md#1-virtual-dom)
2. **Hooks Lifecycle:** Объясните жизненный цикл компонента с хуками. Как `useEffect` соотносится с `componentDidMount/Update/Unmount`? [➜](03-Frontend-React.md#2-hooks-lifecycle)
3. **State Management:** Сравните Redux, MobX, Zustand и React Context. Когда какой инструмент выбрать? [➜](03-Frontend-React.md#3-state-management)
4. **Performance:** Как работают `useMemo`, `useCallback` и `React.memo`? Когда их использовать? [➜](03-Frontend-React.md#4-performance-optimization)
5. **Zombie Child Problem:** Объясните проблему "Zombie Child" в контексте MobX/Redux и как ее избежать. [➜](03-Frontend-React.md#5-zombie-child-problem)
6. **CSS Isolation:** Объясните принципы БЭМ, CSS Modules и CSS-in-JS. Как обеспечить изоляцию стилей? [➜](03-Frontend-React.md#6-css-isolation)
7. **Accessibility (A11y):** Какие принципы accessibility важны для веб-приложений? Как тестировать A11y? [➜](03-Frontend-React.md#7-accessibility-a11y)
8. **Server-Side Rendering:** Объясните SSR, SSG и ISR. Когда использовать каждый подход? [➜](03-Frontend-React.md#8-server-side-rendering)
9. **Testing Strategies:** Какой подход к тестированию (Unit/Integration/E2E) предпочтителен для React? Какие инструменты? [➜](03-Frontend-React.md#9-testing-strategies)
10. **Browser APIs:** Объясните Intersection Observer, ResizeObserver, MutationObserver. Приведите практические примеры. [➜](03-Frontend-React.md#10-browser-apis)

### 💻 Практика / Live Coding (5 задач)

1. **Custom Hook:** Создайте custom hook `useDebounce` или `useLocalStorage`. [➜](03-Frontend-React.md#1-custom-hook)
2. **Accordion/Modal:** Создайте компонент Accordion или Modal с учетом accessibility и анимации. [➜](03-Frontend-React.md#2-accordionmodal)
3. **Infinite Scroll:** Реализуйте компонент с бесконечной прокруткой используя Intersection Observer. [➜](03-Frontend-React.md#3-infinite-scroll)
4. **Form Validation:** Создайте форму с кастомной валидацией без использования библиотек. [➜](03-Frontend-React.md#4-form-validation)
5. **Data Fetching:** Реализуйте компонент с загрузкой данных, обработкой ошибок и состоянием loading. [➜](03-Frontend-React.md#5-data-fetching)

---

## 4. Backend (Node.js / Databases)

### 📚 Теория (10 вопросов)

1. **Event Loop in Node:** Как работает Event Loop в Node.js? В чем разница с браузерным? [➜](04-Backend-NodeJS-Databases.md#1-event-loop-in-node)
2. **Concurrency Model:** В чем разница между потоками, процессами и асинхронным I/O? Как Node.js обрабатывает CPU-bound операции? [➜](04-Backend-NodeJS-Databases.md#2-concurrency-model)
3. **Streams:** Объясните типы потоков в Node.js (Readable, Writable, Duplex, Transform). Когда их использовать? [➜](04-Backend-NodeJS-Databases.md#3-streams)
4. **Transaction Isolation:** Объясните уровни изоляции транзакций в PostgreSQL (Read Uncommitted, Read Committed, Repeatable Read, Serializable). [➜](04-Backend-NodeJS-Databases.md#4-transaction-isolation)
5. **SQL Optimization:** Как работают индексы? Объясните B-tree vs Hash индексы. Как использовать EXPLAIN ANALYZE? [➜](04-Backend-NodeJS-Databases.md#5-sql-optimization)
6. **Security:** Опишите меры для предотвращения XSS, CSRF, SQL Injection и SSRF. [➜](04-Backend-NodeJS-Databases.md#6-security)
7. **ORM vs Raw SQL:** Сравните подходы Raw SQL, ORM (Sequelize/TypeORM/Prisma) и Query Builder. Когда какой выбрать? [➜](04-Backend-NodeJS-Databases.md#7-orm-vs-raw-sql)
8. **Connection Pooling:** Зачем нужен connection pooling? Как настроить пул соединений? [➜](04-Backend-NodeJS-Databases.md#8-connection-pooling)
9. **Caching Strategies:** Объясните стратегии кеширования: Cache-Aside, Read-Through, Write-Through, Write-Behind. [➜](04-Backend-NodeJS-Databases.md#9-caching-strategies)
10. **API Design:** Сравните REST, GraphQL и gRPC. Когда какой подход использовать? [➜](04-Backend-NodeJS-Databases.md#10-api-design)

### 💻 Практика / Live Coding (5 задач)

1. **SQL Query:** Напишите сложный SQL-запрос с JOIN, GROUP BY, HAVING и оконными функциями. [➜](04-Backend-NodeJS-Databases.md#1-sql-query)
2. **Rate Limiter:** Реализуйте middleware для rate limiting с использованием sliding window. [➜](04-Backend-NodeJS-Databases.md#2-rate-limiter)
3. **Database Migration:** Напишите миграцию для добавления новой таблицы со связями и индексами. [➜](04-Backend-NodeJS-Databases.md#3-database-migration)
4. **API Endpoint:** Создайте REST endpoint с валидацией, обработкой ошибок и транзакцией. [➜](04-Backend-NodeJS-Databases.md#4-api-endpoint)
5. **Background Job:** Реализуйте систему фоновых задач с retry logic и dead letter queue. [➜](04-Backend-NodeJS-Databases.md#5-background-job)

---

## 5. System Design

### 📚 Теория (10 вопросов)

1. **CAP Theorem:** Объясните теорему CAP. Как она влияет на выбор между PostgreSQL и Cassandra? [➜](05-System-Design.md#1-cap-theorem)
2. **Microservices vs Monolith:** Когда переходить к микросервисам? Назовите ключевые проблемы и способы их решения. [➜](05-System-Design.md#2-microservices-vs-monolith)
3. **Load Balancing:** Объясните алгоритмы балансировки (Round Robin, Least Connections, Consistent Hashing). [➜](05-System-Design.md#3-load-balancing)
4. **Database Sharding:** Объясните горизонтальное и вертикальное партиционирование. Как выбрать ключ шардирования? [➜](05-System-Design.md#4-database-sharding)
5. **Message Queues:** Сравните RabbitMQ и Kafka. Когда использовать каждый? [➜](05-System-Design.md#5-message-queues)
6. **Consistency Patterns:** Объясните eventual consistency, strong consistency и SAGA pattern. [➜](05-System-Design.md#6-consistency-patterns)
7. **Caching Layers:** Как организовать многоуровневое кеширование (CDN, Redis, Application Cache)? [➜](05-System-Design.md#7-caching-layers)
8. **API Gateway:** Зачем нужен API Gateway? Какие функции он выполняет? [➜](05-System-Design.md#8-api-gateway)
9. **Circuit Breaker:** Объясните паттерн Circuit Breaker. Как он помогает в распределенных системах? [➜](05-System-Design.md#9-circuit-breaker)
10. **Observability:** Что такое Three Pillars of Observability (Logs, Metrics, Traces)? Какие инструменты использовать? [➜](05-System-Design.md#10-observability)

### 💻 Практика / System Design Tasks (5 задач)

1. **URL Shortener:** Спроектируйте сервис сокращения ссылок (генерация ID, кеширование, аналитика). [➜](05-System-Design.md#1-url-shortener)
2. **News Feed:** Спроектируйте ленту новостей (Fan-out on write vs read, хранение, персонализация). [➜](05-System-Design.md#2-news-feed)
3. **Chat System:** Спроектируйте систему мгновенных сообщений (WebSocket, гарантии доставки, история). [➜](05-System-Design.md#3-chat-system)
4. **Rate Limiter Service:** Спроектируйте распределенный rate limiter для API (алгоритмы, хранение, масштабирование). [➜](05-System-Design.md#4-rate-limiter-service)
5. **File Storage:** Спроектируйте систему хранения файлов типа Dropbox (chunking, deduplication, sync). [➜](05-System-Design.md#5-file-storage)

---

## 6. Web Architecture & Performance

### 📚 Теория (10 вопросов)

1. **HTTP/2 vs HTTP/3:** Объясните различия между HTTP/2 и HTTP/3. Какие преимущества дает QUIC? [➜](06-Web-Architecture-Performance.md#1-http2-vs-http3)
2. **WebSockets:** Как работают WebSockets? Когда использовать вместо HTTP polling? [➜](06-Web-Architecture-Performance.md#2-websockets)
3. **CDN:** Как работает Content Delivery Network? Какие типы кеширования используются? [➜](06-Web-Architecture-Performance.md#3-cdn)
4. **Browser Rendering:** Объясните Critical Rendering Path. Что такое reflow и repaint? [➜](06-Web-Architecture-Performance.md#4-browser-rendering)
5. **Core Web Vitals:** Что такое LCP, FID, CLS? Как оптимизировать каждый метрик? [➜](06-Web-Architecture-Performance.md#5-core-web-vitals)
6. **Lazy Loading:** Как работает lazy loading изображений, компонентов и модулей? [➜](06-Web-Architecture-Performance.md#6-lazy-loading)
7. **Code Splitting:** Как работает code splitting? Когда использовать dynamic imports? [➜](06-Web-Architecture-Performance.md#7-code-splitting)
8. **Service Workers:** Как работают Service Workers? Как реализовать offline-first приложение? [➜](06-Web-Architecture-Performance.md#8-service-workers)
9. **Web Security:** Объясните CORS, CSP, HSTS, XSS prevention. Как защитить веб-приложение? [➜](06-Web-Architecture-Performance.md#9-web-security)
10. **Progressive Enhancement:** Что такое progressive enhancement и graceful degradation? [➜](06-Web-Architecture-Performance.md#10-progressive-enhancement)

### 💻 Практика / Implementation Tasks (5 задач)

1. **Performance Audit:** Проведите аудит производительности сайта и предложите оптимизации. [➜](06-Web-Architecture-Performance.md#1-performance-audit)
2. **Caching Strategy:** Спроектируйте многоуровневую стратегию кеширования для веб-приложения. [➜](06-Web-Architecture-Performance.md#2-caching-strategy)
3. **Real-time Updates:** Реализуйте систему real-time обновлений с fallback на polling. [➜](06-Web-Architecture-Performance.md#3-real-time-updates)
4. **Image Optimization:** Настройте автоматическую оптимизацию изображений с разными форматами (WebP, AVIF). [➜](06-Web-Architecture-Performance.md#4-image-optimization)
5. **PWA:** Создайте Progressive Web App с offline поддержкой и push notifications. [➜](06-Web-Architecture-Performance.md#5-pwa)

---

## 7. Инфраструктура и DevOps

### 📚 Теория (10 вопросов)

1. **Docker Layers:** Как работают слои в Docker? Как оптимизировать Dockerfile для кеширования? [➜](07-Infrastructure-DevOps.md#1-docker-layers)
2. **Kubernetes Concepts:** Объясните Pod, Deployment, Service, Ingress, ConfigMap, Secret. [➜](07-Infrastructure-DevOps.md#2-kubernetes-concepts)
3. **Deployment Strategies:** Сравните Rolling Update, Blue-Green и Canary deployments. [➜](07-Infrastructure-DevOps.md#3-deployment-strategies)
4. **AWS Services:** Сравните Lambda, ECS, EKS и EC2. Когда использовать каждый? [➜](07-Infrastructure-DevOps.md#4-aws-services)
5. **High Availability:** Как спроектировать отказоустойчивую архитектуру в AWS (Multi-AZ, Auto Scaling)? [➜](07-Infrastructure-DevOps.md#5-high-availability)
6. **CI/CD:** Объясните принципы CI/CD. Какие этапы должен содержать пайплайн? [➜](07-Infrastructure-DevOps.md#6-cicd)
7. **Infrastructure as Code:** Сравните Terraform, CloudFormation и Pulumi. [➜](07-Infrastructure-DevOps.md#7-infrastructure-as-code)
8. **Container Security:** Какие best practices для безопасности контейнеров? [➜](07-Infrastructure-DevOps.md#8-container-security)
9. **Monitoring & Alerting:** Какие метрики важны для backend-сервиса? Как настроить SLO/SLA? [➜](07-Infrastructure-DevOps.md#9-monitoring--alerting)
10. **Secrets Management:** Как безопасно хранить и использовать секреты (Vault, AWS Secrets Manager)? [➜](07-Infrastructure-DevOps.md#10-secrets-management)

### 💻 Практика / Hands-on Tasks (5 задач)

1. **Dockerfile:** Напишите оптимальный multi-stage Dockerfile для Node.js приложения. [➜](07-Infrastructure-DevOps.md#1-dockerfile)
2. **Kubernetes Manifest:** Напишите манифесты для Deployment, Service и Ingress с health checks. [➜](07-Infrastructure-DevOps.md#2-kubernetes-manifest)
3. **CI Pipeline:** Создайте GitHub Actions/GitLab CI pipeline для тестирования, сборки и деплоя. [➜](07-Infrastructure-DevOps.md#3-ci-pipeline)
4. **Terraform:** Напишите Terraform конфигурацию для создания VPC, EC2 и RDS. [➜](07-Infrastructure-DevOps.md#4-terraform)
5. **Helm Chart:** Создайте Helm chart для развертывания приложения с конфигурируемыми параметрами. [➜](07-Infrastructure-DevOps.md#5-helm-chart)

---

## 8. Applied AI / ML Systems

### 📚 Теория (10 вопросов)

1. **LLM Basics:** Объясните, как работают Large Language Models. Что такое tokens, embeddings, attention? [➜](08-Applied-AI-ML.md#1-llm-basics)
2. **Prompt Engineering:** Какие техники prompt engineering существуют (few-shot, chain-of-thought, self-consistency)? [➜](08-Applied-AI-ML.md#2-prompt-engineering)
3. **RAG Architecture:** Объясните Retrieval-Augmented Generation. Когда и зачем использовать? [➜](08-Applied-AI-ML.md#3-rag-architecture)
4. **Vector Databases:** Сравните Pinecone, Weaviate, Milvus. Как работает similarity search? [➜](08-Applied-AI-ML.md#4-vector-databases)
5. **Fine-tuning vs Prompting:** Когда делать fine-tuning модели, а когда достаточно prompt engineering? [➜](08-Applied-AI-ML.md#5-fine-tuning-vs-prompting)
6. **Embeddings:** Что такое embeddings? Как выбрать модель для создания embeddings? [➜](08-Applied-AI-ML.md#6-embeddings)
7. **LLM Limitations:** Какие ограничения у LLM (hallucinations, context window, latency)? Как их митигировать? [➜](08-Applied-AI-ML.md#7-llm-limitations)
8. **AI Safety:** Какие принципы безопасности важны при работе с LLM в продакшене? [➜](08-Applied-AI-ML.md#8-ai-safety)
9. **Model Evaluation:** Как оценивать качество LLM-приложений? Какие метрики использовать? [➜](08-Applied-AI-ML.md#9-model-evaluation)
10. **Cost Optimization:** Как оптимизировать стоимость запросов к LLM API? [➜](08-Applied-AI-ML.md#10-cost-optimization)

### 💻 Практика / Implementation Tasks (5 задач)

1. **RAG Pipeline:** Спроектируйте и реализуйте RAG pipeline для вопросно-ответной системы. [➜](08-Applied-AI-ML.md#1-rag-pipeline)
2. **Prompt Template:** Создайте систему prompt templates с переменными и версионированием. [➜](08-Applied-AI-ML.md#2-prompt-template)
3. **Streaming Response:** Реализуйте API с streaming ответами от LLM. [➜](08-Applied-AI-ML.md#3-streaming-response)
4. **Embedding Search:** Реализуйте semantic search с использованием embeddings и vector database. [➜](08-Applied-AI-ML.md#4-embedding-search)
5. **LLM Evaluation:** Создайте pipeline для автоматической оценки качества ответов LLM. [➜](08-Applied-AI-ML.md#5-llm-evaluation)

---

## 9. Behavioral / Leadership

### 📚 Теория / Концепции (10 вопросов)

1. **Technical Decision Making:** Как вы принимаете технические решения в условиях неопределенности? [➜](09-Behavioral-Leadership.md#1-technical-decision-making)
2. **Conflict Resolution:** Как вы разрешаете технические разногласия в команде? [➜](09-Behavioral-Leadership.md#2-conflict-resolution)
3. **Prioritization Framework:** Какую систему приоритизации задач вы используете? [➜](09-Behavioral-Leadership.md#3-prioritization-framework)
4. **Technical Debt:** Как вы убеждаете бизнес инвестировать в погашение техдолга? [➜](09-Behavioral-Leadership.md#4-technical-debt)
5. **Mentorship Approach:** Как вы обучаете и развиваете junior инженеров? [➜](09-Behavioral-Leadership.md#5-mentorship-approach)
6. **Code Review Philosophy:** Какие принципы code review вы считаете важными? [➜](09-Behavioral-Leadership.md#6-code-review-philosophy)
7. **Failure Analysis:** Как вы проводите post-mortem после инцидентов? [➜](09-Behavioral-Leadership.md#7-failure-analysis)
8. **Estimation:** Как вы оцениваете сроки выполнения задач? [➜](09-Behavioral-Leadership.md#8-estimation)
9. **Cross-team Collaboration:** Как вы работаете с другими командами над общими проектами? [➜](09-Behavioral-Leadership.md#9-cross-team-collaboration)
10. **Staying Current:** Как вы следите за новыми технологиями и трендами? [➜](09-Behavioral-Leadership.md#10-staying-current)

### 💬 Практика / STAR Stories (5 ситуаций)

1. **Conflict Story:** Расскажите о случае, когда вы не согласились с решением архитектора. Как вы вели дискуссию? [➜](09-Behavioral-Leadership.md#1-conflict-story)
2. **Failure Story:** Расскажите о проекте, который провалился. Что вы узнали? [➜](09-Behavioral-Leadership.md#2-failure-story)
3. **Mentorship Story:** Приведите пример успешного менторства junior инженера. [➜](09-Behavioral-Leadership.md#3-mentorship-story)
4. **Prioritization Story:** Опишите ситуацию с критическим багом, горящей фичей и техдолгом. Как расставили приоритеты? [➜](09-Behavioral-Leadership.md#4-prioritization-story)
5. **Impact Story:** Расскажите о своем самом значимом техническом вкладе. Какой был impact? [➜](09-Behavioral-Leadership.md#5-impact-story)

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
