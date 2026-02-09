# Web Architecture & Performance - Ответы на вопросы интервью

## 📚 Теория

### 1. HTTP/2 vs HTTP/3

**Вопрос:** Объясните различия между HTTP/2 и HTTP/3. Какие преимущества дает QUIC?

**Ответ:**

**HTTP/2 (2015):**
```
┌─────────────────────────────────────────────────────────────┐
│                    HTTP/2 Architecture                       │
│                                                              │
│  • Binary framing layer (не текстовый как HTTP/1.1)          │
│  • Multiplexing — несколько запросов в одном TCP соединении  │
│  • Header compression (HPACK)                               │
│  • Server push (deprecated в Chrome)                        │
│                                                              │
│  Connection:                                                │
│  TCP ──► TLS (1.2+) ──► HTTP/2                              │
└─────────────────────────────────────────────────────────────┘
```

**HTTP/3 (2022):**
```
┌─────────────────────────────────────────────────────────────┐
│                    HTTP/3 Architecture                       │
│                                                              │
│  • QUIC вместо TCP                                          │
│  • Встроенное шифрование (TLS 1.3)                          │
│  • Fast connection establishment (0-RTT или 1-RTT)         │
│  • Improved congestion control                              │
│                                                              │
│  Connection:                                                │
│  QUIC (UDP) ──► HTTP/3                                      │
│  └── TLS 1.3 встроен в QUIC                                │
└─────────────────────────────────────────────────────────────┘
```

**QUIC (Quick UDP Internet Connections):**

| Аспект | HTTP/2 (TCP) | HTTP/3 (QUIC) |
|--------|--------------|---------------|
| Transport | TCP | UDP |
| Encryption | TLS поверх TCP | TLS встроен в QUIC |
| Handshake | 2-3 RTT | 0-RTT или 1-RTT |
| Connection ID | IP + port | Connection ID (persistent) |
| Multiplexing | Stream multiplexing | Native streams |
| Head-of-line blocking | TCP-level | Packet-level only |

**Преимущества QUIC:**
- **Fast connection:** 1 RTT для новых, 0-RTT для повторных соединений
- **Connection migration:** переключение WiFi → 4G без разрыва соединения
- **Improved security:** TLS 1.3 обязателен, уменьшает fingerprinting
- **Better congestion control:** информация на уровне приложения

**0-RTT (Zero Round Trip Time):**
```
Client                                 Server
   │                                       │
   │── ClientHello + early data ──────────►│
   │                                       │
   │◄───── ServerHello + response ────────│
   │                                       │
   (Application data в первом пакете!)
```

---

### 2. WebSockets

**Вопрос:** Как работают WebSockets? Когда использовать вместо HTTP polling?

**Ответ:**

```
┌─────────────────────────────────────────────────────────────┐
│                    WebSocket Handshake                       │
│                                                              │
│  HTTP Upgrade Request:                                       │
│  GET /chat HTTP/1.1                                         │
│  Host: server.example.com                                   │
│  Upgrade: websocket                                         │
│  Connection: Upgrade                                        │
│  Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==               │
│  Sec-WebSocket-Version: 13                                  │
│                                                              │
│  Server Response:                                            │
│  HTTP/1.1 101 Switching Protocols                          │
│  Upgrade: websocket                                         │
│  Connection: Upgrade                                        │
│  Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=        │
│                                                              │
│  После этого: Full-duplex TCP соединение                   │
└─────────────────────────────────────────────────────────────┘
```

**WebSocket фреймы:**
```
┌───────────────────────────────────────────────────────────┐
│  0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15                    │
│ ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┐                          │
│ │F│R│R│R│opcode│M│ Payload len │    Masked payload        │
│ │I│S│S│S│      │A│             │    (if masked)           │
│ │N│V│V│V│      │S│             │                          │
│ │ │0│1│2│      │K│             │                          │
│ └─┴─┴─┴─┴──────┴─┴─────────────┘                          │
│                                                            │
│  FIN: последний фрейм сообщения                           │
│  Opcode: text(1), binary(2), close(8), ping(9), pong(10)   │
│  Mask: XOR шифрование (client → server)                   │
└───────────────────────────────────────────────────────────┘
```

**Когда использовать:**

| Scenario | HTTP Polling | WebSockets |
|----------|-------------|------------|
| Real-time chat | ❌ High latency | ✅ Instant |
| Live sports updates | ❌ Resource waste | ✅ Efficient |
| Collaborative editing | ❌ Conflicts | ✅ Operational transforms |
| Stock tickers | ❌ Delayed data | ✅ Real-time |
| Simple notifications | ✅ Good enough | ❌ Overkill |

**Alternatives:**
- **Server-Sent Events (SSE):** однонаправленный (server → client), HTTP-based
- **Long Polling:** fallback, simpler infrastructure

---

### 3. CDN

**Вопрос:** Как работает Content Delivery Network? Какие типы кеширования используются?

**Ответ:**

```
┌─────────────────────────────────────────────────────────────┐
│                    CDN Architecture                          │
│                                                              │
│         ┌─────────┐                                         │
│         │  User   │                                         │
│         │ (Tokyo) │                                         │
│         └────┬────┘                                         │
│              │                                               │
│              ▼                                               │
│   ┌─────────────────────┐                                   │
│   │   Edge Server       │◄─── Cache HIT (80-90%)            │
│   │   (Tokyo POP)       │                                   │
│   └──────────┬──────────┘                                   │
│              │                                               │
│         MISS │                                               │
│              ▼                                               │
│   ┌─────────────────────┐                                   │
│   │   Origin Shield     │                                   │
│   │   (Regional Cache)  │                                   │
│   └──────────┬──────────┘                                   │
│              │                                               │
│              ▼                                               │
│   ┌─────────────────────┐                                   │
│   │   Origin Server     │                                   │
│   │   (US Datacenter)   │                                   │
│   └─────────────────────┘                                   │
└─────────────────────────────────────────────────────────────┘
```

**Типы кеширования:**

**1. Static Content:**
```
Cache-Control: public, max-age=31536000, immutable
ETag: "abc123"
Last-Modified: Mon, 01 Jan 2024 00:00:00 GMT
```

**2. Dynamic Content (stale-while-revalidate):**
```
Cache-Control: public, max-age=60, stale-while-revalidate=3600
// 60 сек свежий, потом устаревший на 1 час пока обновляется
```

**3. Personalized (edge computing):**
```
// Cloudflare Workers, Lambda@Edge
// Кеширование с ключом = URL + User ID
```

**Cache Invalidation:**
- **Purge API:** инвалидация по URL/pattern
- **Versioned URLs:** `script.v2.js` вместо `script.js`
- **Surrogate Keys:** теги для групповой инвалидации

---

### 4. Browser Rendering

**Вопрос:** Объясните Critical Rendering Path. Что такое reflow и repaint?

**Ответ:**

```
┌─────────────────────────────────────────────────────────────┐
│              Critical Rendering Path                         │
│                                                              │
│  1. HTML ──► DOM Tree                                        │
│  2. CSS ──► CSSOM Tree                                       │
│  3. DOM + CSSOM ──► Render Tree                              │
│  4. Layout (Reflow) ──► Geometry                             │
│  5. Paint ──► Pixels                                         │
│  6. Composite ──► GPU Layers                                  │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  HTML Parse ──► DOM                                   │  │
│  │        │                                              │  │
│  │        ▼                                              │  │
│  │  CSS Parse ──► CSSOM                                  │  │
│  │        │                                              │  │
│  │        └──────────┬───────────────────────────────────┘  │
│  │                   ▼                                      │
│  │  JavaScript ──► Render Tree (visible nodes only)       │  │
│  │        │                                              │  │
│  │        ▼                                              │  │
│  │  Layout (Reflow) ──► Box positions                    │  │
│  │        │                                              │  │
│  │        ▼                                              │  │
│  │  Paint ──► Draw calls                                 │  │
│  │        │                                              │  │
│  │        ▼                                              │  │
│  │  Composite ──► GPU layers                             │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

**Reflow (Layout):**
- Пересчет геометрии элементов
- Триггеры: изменение размеров, позиции, структуры DOM
- **Стоимость:** O(n) для всего документа

```javascript
// Вызывает reflow
el.style.width = '100px';
el.style.height = '200px';
el.style.margin = '10px';
// 3 reflow! (лучше использовать CSS класс)
```

**Repaint:**
- Перерисовка визуальных свойств без изменения layout
- Триггеры: color, background-color, visibility, border-radius

**Стоимость операций:**
```
Reflow > Repaint > Composite (быстрее всего)
```

**Performance tips:**
- Использовать `transform` и `opacity` (GPU-accelerated)
- Избегать layout thrashing
- Batch DOM changes (DocumentFragment, CSS classes)
- `contain: layout` для изоляции

---

### 5. Core Web Vitals

**Вопрос:** Что такое LCP, FID, CLS? Как оптимизировать каждый метрик?

**Ответ:**

**Core Web Vitals:**

| Metric | Description | Good | Needs Improvement |
|--------|-------------|------|-------------------|
| **LCP** | Largest Contentful Paint | ≤2.5s | ≤4.0s |
| **FID** | First Input Delay | ≤100ms | ≤300ms |
| **INP** | Interaction to Next Paint | ≤200ms | ≤500ms |
| **CLS** | Cumulative Layout Shift | ≤0.1 | ≤0.25 |

*Note: INP заменяет FID с марта 2024*

**LCP Optimization:**
```html
<!-- Предзагрузка LCP элемента -->
<link rel="preload" as="image" href="hero.webp" fetchpriority="high">

<!-- Оптимизация -->
• Compress images (WebP/AVIF)
• Responsive images (srcset)
• CDN for static assets
• Server-side rendering
• Resource hints (preconnect, dns-prefetch)
```

**FID/INP Optimization:**
```javascript
// Плохо: блокируем main thread
button.addEventListener('click', () => {
  heavyComputation(); // 500ms blocking
});

// Хорошо: yield to main thread
button.addEventListener('click', async () => {
  await scheduler.yield();
  heavyComputation();
});

// Или: Web Workers
const worker = new Worker('worker.js');
```

**CLS Optimization:**
```html
<!-- Always specify dimensions -->
<img src="photo.jpg" width="800" height="600" alt="">

<!-- Reserve space for dynamic content -->
<div style="min-height: 300px;">
  <!-- Ad or widget loads here -->
</div>

<!-- Avoid inserting content above existing -->
```

---

### 6. Lazy Loading

**Вопрос:** Как работает lazy loading изображений, компонентов и модулей?

**Ответ:**

**Images:**
```html
<!-- Native lazy loading -->
<img src="image.jpg" loading="lazy" alt="">

<!-- Intersection Observer (fallback) -->
<img data-src="image.jpg" class="lazy-image" alt="">
<script>
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      observer.unobserve(img);
    }
  });
});
document.querySelectorAll('.lazy-image').forEach(img => observer.observe(img));
</script>
```

**Components (React):**
```javascript
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyComponent />
    </Suspense>
  );
}
```

**Routes (React Router):**
```javascript
const Dashboard = React.lazy(() => import('./Dashboard'));

<Route path="/dashboard" element={
  <Suspense fallback={<Loading />}>
    <Dashboard />
  </Suspense>
} />
```

**Modules (dynamic import):**
```javascript
button.addEventListener('click', async () => {
  const { heavyFunction } = await import('./heavy-module.js');
  heavyFunction();
});
```

---

### 7. Code Splitting

**Вопрос:** Как работает code splitting? Когда использовать dynamic imports?

**Ответ:**

**Code Splitting стратегии:**

**1. Route-based:**
```javascript
// Каждый route = отдельный chunk
const Home = lazy(() => import('./Home'));
const About = lazy(() => import('./About'));
const Dashboard = lazy(() => import('./Dashboard'));
```

**2. Component-based:**
```javascript
// Тяжелые компоненты загружаются по требованию
const Chart = lazy(() => import('./Chart'));
const Map = lazy(() => import('./Map'));
```

**3. Library splitting:**
```javascript
// Отдельный chunk для больших библиотек
const moment = await import('moment');
```

**Prefetch/Preload:**
```javascript
// Prefetch: загрузить при idle
const Dashboard = lazy(() => import(
  /* webpackPrefetch: true */ './Dashboard'
));

// Preload: загрузить сразу (важный модуль)
const CriticalModule = lazy(() => import(
  /* webpackPreload: true */ './Critical'
));
```

**Когда использовать dynamic imports:**
- Код не нужен для initial render
- User interaction triggered
- Route-based splitting
- Conditional features
- Large libraries (>100KB)

---

### 8. Service Workers

**Вопрос:** Как работают Service Workers? Как реализовать offline-first приложение?

**Ответ:**

```
┌─────────────────────────────────────────────────────────────┐
│                 Service Worker Lifecycle                     │
│                                                              │
│  1. Install                                                 │
│     └── Кеширование static assets (app shell)              │
│                                                              │
│  2. Activate                                                │
│     └── Удаление старых кешей                              │
│                                                              │
│  3. Fetch Intercept                                         │
│     └── Перехват сетевых запросов                          │
│         • Cache First (static assets)                      │
│         • Network First (API data)                         │
│         • Stale While Revalidate                           │
└─────────────────────────────────────────────────────────────┘
```

**Service Worker Registration:**
```javascript
// main.js
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(reg => console.log('SW registered'))
    .catch(err => console.error('SW failed', err));
}
```

**Cache Strategies:**
```javascript
// sw.js
const CACHE_NAME = 'app-v1';
const STATIC_ASSETS = [
  '/',
  '/index.html',
  '/app.js',
  '/styles.css'
];

// Install: cache static assets
self.addEventListener('install', (e) => {
  e.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(STATIC_ASSETS))
  );
  self.skipWaiting();
});

// Fetch: different strategies
self.addEventListener('fetch', (e) => {
  const { request } = e;
  
  // Cache First for static assets
  if (request.destination === 'image' || 
      request.destination === 'style' ||
      request.destination === 'script') {
    e.respondWith(cacheFirst(request));
  }
  
  // Network First for API calls
  else if (request.url.includes('/api/')) {
    e.respondWith(networkFirst(request));
  }
});

async function cacheFirst(request) {
  const cached = await caches.match(request);
  return cached || fetch(request);
}

async function networkFirst(request) {
  try {
    const networkResponse = await fetch(request);
    const cache = await caches.open(CACHE_NAME);
    cache.put(request, networkResponse.clone());
    return networkResponse;
  } catch (error) {
    return caches.match(request);
  }
}
```

**Background Sync:**
```javascript
// Queue failed requests
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-posts') {
    event.waitUntil(syncPosts());
  }
});

async function syncPosts() {
  const db = await openDB('posts-queue');
  const posts = await db.getAll('pending');
  
  for (const post of posts) {
    await fetch('/api/posts', {
      method: 'POST',
      body: JSON.stringify(post)
    });
    await db.delete('pending', post.id);
  }
}
```

---

### 9. Web Security

**Вопрос:** Объясните CORS, CSP, HSTS, XSS prevention. Как защитить веб-приложение?

**Ответ:**

**CORS (Cross-Origin Resource Sharing):**
```
┌─────────────────────────────────────────────────────────────┐
│                      CORS Flow                               │
│                                                              │
│  Simple Request:                                             │
│  GET /data HTTP/1.1                                         │
│  Origin: https://frontend.com                               │
│                                                              │
│  HTTP/1.1 200 OK                                            │
│  Access-Control-Allow-Origin: https://frontend.com          │
│  Access-Control-Allow-Credentials: true                     │
│                                                              │
│  Preflight (for complex requests):                          │
│  OPTIONS /data HTTP/1.1                                     │
│  Origin: https://frontend.com                               │
│  Access-Control-Request-Method: POST                        │
│  Access-Control-Request-Headers: X-Custom                   │
└─────────────────────────────────────────────────────────────┘
```

**CSP (Content Security Policy):**
```http
Content-Security-Policy: 
  default-src 'self';
  script-src 'self' 'unsafe-inline' cdn.example.com;
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  connect-src 'self' api.example.com;
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self';
```

**HSTS (HTTP Strict Transport Security):**
```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

**XSS Prevention:**
```javascript
// ❌ Небезопасно (DOM XSS)
element.innerHTML = userInput;

// ✅ Безопасно
element.textContent = userInput;

// ✅ Или с sanitization
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```

**Security Headers Checklist:**
- [ ] Content-Security-Policy
- [ ] Strict-Transport-Security (HSTS)
- [ ] X-Frame-Options / frame-ancestors
- [ ] X-Content-Type-Options: nosniff
- [ ] Referrer-Policy
- [ ] Permissions-Policy

---

### 10. Progressive Enhancement

**Вопрос:** Что такое progressive enhancement и graceful degradation?

**Ответ:**

**Progressive Enhancement:**
```
┌─────────────────────────────────────────────────────────────┐
│             Progressive Enhancement Layers                   │
│                                                              │
│  Layer 3: JavaScript (enhanced experience)                  │
│     • SPA functionality                                     │
│     • Animations                                            │
│     • Real-time updates                                     │
│                                                              │
│  Layer 2: CSS (visual design)                               │
│     • Layout                                                │
│     • Colors, typography                                    │
│     • Responsive design                                     │
│                                                              │
│  Layer 1: HTML (core content)                               │
│     • Semantic markup                                       │
│     • Accessible content                                    │
│     • Works everywhere                                      │
└─────────────────────────────────────────────────────────────┘
```

**Graceful Degradation (обратный подход):**
- Строим полный функционал
- Деградируем для старых браузеров
- Feature detection с fallback

**Feature Detection:**
```javascript
// Modern approach
if ('IntersectionObserver' in window) {
  // Use Intersection Observer
} else {
  // Fallback to scroll events
}

// CSS Feature Queries
@supports (display: grid) {
  .container {
    display: grid;
  }
}

@supports not (display: grid) {
  .container {
    display: flex;
    flex-wrap: wrap;
  }
}
```

---

## 💻 Практика / Implementation Tasks

### 1. Performance Audit

**Задача:** Проведите аудит производительности сайта и предложите оптимизации.

**Инструменты:**
- Chrome DevTools (Lighthouse, Performance tab)
- WebPageTest
- PageSpeed Insights
- Real User Monitoring (RUM)

**Чеклист аудита:**
- [ ] LCP, INP, CLS метрики
- [ ] Размер бандла (tree-shaking)
- [ ] Network waterfall analysis
- [ ] Render blocking resources
- [ ] Image optimization
- [ ] Font loading strategy
- [ ] Third-party scripts impact

---

### 2. Caching Strategy

**Задача:** Спроектируйте многоуровневую стратегию кеширования для веб-приложения.

**Стратегия:**
```
CDN (Edge) → Browser Cache → Service Worker → API Cache
```

---

### 3. Real-time Updates

**Задача:** Реализуйте систему real-time обновлений с fallback на polling.

**Архитектура:**
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Client    │◄───►│  WebSocket  │◄───►│   Server    │
│             │     │  (primary)  │     │             │
│   (fallback │     │             │     │             │
│   to SSE    │◄───►│    SSE      │     │             │
│   or polling│     │  (fallback) │     │             │
└─────────────┘     └─────────────┘     └─────────────┘
```

---

### 4. Image Optimization

**Задача:** Настройте автоматическую оптимизацию изображений с разными форматами.

**Подход:**
```html
<picture>
  <source srcset="image.avif" type="image/avif">
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="" loading="lazy">
</picture>
```

---

### 5. PWA

**Задача:** Создайте Progressive Web App с offline поддержкой и push notifications.

**Требования:**
- Web App Manifest
- Service Worker с offline support
- HTTPS
- Responsive design
- Push notifications (опционально)
- Background sync (опционально)
