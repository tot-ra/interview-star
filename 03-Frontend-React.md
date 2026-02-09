# Frontend (React / UI) - Ответы на вопросы интервью

## 📚 Теория

### 1. Virtual DOM

**Вопрос:** Как работает Virtual DOM в React? Объясните процесс reconciliation.

**Ответ:**

**Virtual DOM** — легковесное JavaScript-представление реального DOM. Это обычные JS-объекты, описывающие структуру UI.

```javascript
// Virtual DOM node
{
  type: 'div',
  props: {
    className: 'container',
    children: [
      { type: 'h1', props: { children: 'Hello' } },
      { type: 'p', props: { children: 'World' } }
    ]
  }
}
```

**Процесс обновления:**

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   State      │───►│  New VDOM    │───►│    Diff      │
│   Change     │    │   Tree       │    │  (Reconcile) │
└──────────────┘    └──────────────┘    └──────┬───────┘
                                               │
                    ┌──────────────┐    ┌──────▼───────┐
                    │   Real DOM   │◄───│   Patch      │
                    │   Update     │    │  (Commit)    │
                    └──────────────┘    └──────────────┘
```

**Reconciliation (сверка):**

1. **Render Phase:** создается новое VDOM дерево
2. **Diffing:** сравнение нового и старого VDOM
3. **Commit Phase:** применение минимальных изменений к DOM

**Алгоритм Diffing:**
- **Разные типы элементов** → полная замена поддерева
- **Одинаковые DOM-элементы** → обновление только изменившихся атрибутов
- **Компоненты** → обновление props, вызов render
- **Списки** → используются `key` для оптимизации

**Зачем key:**
```jsx
// Без key React не знает что элементы переместились
// С key React переиспользует существующие DOM-узлы
{items.map(item => <Item key={item.id} {...item} />)}
```

**Оптимизации React 18+ (Fiber):**
- Прерываемый рендеринг
- Приоритизация обновлений
- Concurrent features

---

### 2. Hooks Lifecycle

**Вопрос:** Объясните жизненный цикл компонента с хуками. Как `useEffect` соотносится с lifecycle methods?

**Ответ:**

**Сопоставление с классовыми компонентами:**

| Классовый метод | Hooks эквивалент |
|-----------------|------------------|
| constructor | `useState` начальное значение |
| componentDidMount | `useEffect(() => {}, [])` |
| componentDidUpdate | `useEffect(() => {}, [deps])` |
| componentWillUnmount | `useEffect(() => { return cleanup }, [])` |
| shouldComponentUpdate | `React.memo`, `useMemo` |
| getDerivedStateFromProps | `useState` + условие в render |

**useEffect подробно:**

```javascript
useEffect(() => {
  // Выполняется ПОСЛЕ рендера и коммита в DOM
  // Не блокирует paint браузера
  
  const subscription = subscribe(props.id);
  
  return () => {
    // Cleanup: выполняется перед следующим эффектом
    // и при размонтировании
    subscription.unsubscribe();
  };
}, [props.id]); // Зависимости: эффект перезапускается при их изменении
```

**Порядок выполнения:**

```
1. Render (создание VDOM)
2. React обновляет DOM
3. Browser paint
4. useEffect выполняется (асинхронно после paint)

При обновлении:
1. Render
2. React обновляет DOM  
3. Cleanup предыдущего эффекта
4. Browser paint
5. Новый useEffect
```

**useLayoutEffect:**
- Выполняется синхронно ПОСЛЕ DOM-мутаций, ДО paint
- Используется для измерений DOM, синхронных визуальных обновлений
- Блокирует paint — использовать осторожно

**Правила хуков:**
1. Только на верхнем уровне (не в условиях/циклах)
2. Только в функциональных компонентах или custom hooks

---

### 3. State Management

**Вопрос:** Сравните Redux, MobX, Zustand и React Context. Когда какой инструмент выбрать?

**Ответ:**

| Аспект | Redux | MobX | Zustand | Context |
|--------|-------|------|---------|---------|
| Парадигма | Flux, иммутабельность | Reactive, мутабельность | Flux-lite | Встроенный |
| Boilerplate | Много | Мало | Минимум | Минимум |
| DevTools | Отличные | Хорошие | Есть | Базовые |
| Производительность | Ручная оптимизация | Автоматическая | Хорошая | Ререндер всех consumers |
| Размер | ~7kb | ~16kb | ~1kb | 0 |
| Learning curve | Высокая | Средняя | Низкая | Низкая |

**Redux:**
```javascript
// Много boilerplate, но предсказуемый flow
// Actions → Reducer → Store → View
const slice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: state => { state.value += 1 }
  }
});
```
**Когда:** большие приложения, сложная бизнес-логика, нужен time-travel debugging

**MobX:**
```javascript
// Реактивность "из коробки", мутабельный стейт
class Store {
  @observable count = 0;
  @action increment() { this.count++; }
}
```
**Когда:** сложные взаимосвязанные данные, автоматическая оптимизация ререндеров

**Zustand:**
```javascript
// Минимальный, без провайдеров
const useStore = create(set => ({
  count: 0,
  increment: () => set(state => ({ count: state.count + 1 }))
}));
```
**Когда:** простые/средние приложения, нужна простота без boilerplate

**React Context:**
```javascript
// Встроенный, но ререндерит всех consumers при любом изменении
const Context = createContext();
<Context.Provider value={{ user, setUser }}>
```
**Когда:** редко меняющиеся данные (тема, язык, auth), prop drilling

**Рекомендации:**
- Начинай с `useState` + `useReducer`
- Для prop drilling → Context или Zustand
- Для сложного стейта → Zustand или Redux Toolkit
- Для очень сложных связей данных → MobX

---

### 4. Performance Optimization

**Вопрос:** Как работают `useMemo`, `useCallback` и `React.memo`? Когда их использовать?

**Ответ:**

**React.memo** — HOC для мемоизации компонента:
```javascript
const ExpensiveComponent = React.memo(({ data }) => {
  // Ререндер только если props изменились (shallow compare)
  return <div>{/* complex render */}</div>;
});

// С кастомным сравнением
React.memo(Component, (prevProps, nextProps) => {
  return prevProps.id === nextProps.id; // true = не ререндерить
});
```

**useMemo** — мемоизация вычисленного значения:
```javascript
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]); // Пересчитывается только при изменении a или b
```

**useCallback** — мемоизация функции:
```javascript
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]); // Та же ссылка пока id не изменится
```

**Когда использовать:**

| Ситуация | Решение |
|----------|---------|
| Дочерний компонент с React.memo | `useCallback` для callback props |
| Тяжелые вычисления | `useMemo` |
| Референциальное равенство для useEffect deps | `useMemo`/`useCallback` |
| Передача объекта в мемоизированный компонент | `useMemo` для объекта |

**Когда НЕ использовать:**
- Простые компоненты без детей
- Примитивные props (строки, числа)
- Функции, не передаваемые как props
- Преждевременная оптимизация

**Пример анти-паттерна:**
```javascript
// Бесполезно — объект создается каждый рендер
<Child style={{ color: 'red' }} />

// Правильно
const style = useMemo(() => ({ color: 'red' }), []);
<Child style={style} />
```

**Профилирование:**
- React DevTools Profiler
- `console.time()` / `console.timeEnd()`
- React.memo с `why-did-you-render`

---

### 5. Zombie Child Problem

**Вопрос:** Объясните проблему "Zombie Child" в контексте MobX/Redux и как ее избежать.

**Ответ:**

**Zombie Child** — ситуация, когда дочерний компонент пытается читать удаленные данные из store.

**Сценарий:**

```
1. Parent рендерит список: [Item1, Item2, Item3]
2. Пользователь удаляет Item2
3. Store обновляется: items = [Item1, Item3]
4. Parent начинает ререндер
5. НО: Item2 компонент еще существует и пытается читать свои данные
6. store.items[1] теперь Item3, а не Item2 → БАГИ
```

```
┌─────────────────────────────────────────────────────────────┐
│ Timeline проблемы:                                          │
│                                                              │
│ Store Update ──► Item2 reads ──► Parent rerenders ──► Item2 │
│    (delete)       (stale data)    (removes Item2)   unmounts│
│                      ↑                                       │
│                   ZOMBIE!                                    │
└─────────────────────────────────────────────────────────────┘
```

**В Redux (с connect):**
- `mapStateToProps` вызывается до ререндера родителя
- Компонент может читать удаленные данные

**Решения:**

**1. Защитные проверки в селекторах:**
```javascript
const selectItem = (state, id) => {
  const item = state.items[id];
  if (!item) return null; // Защита от zombie
  return item;
};
```

**2. React-Redux v7+ с useSelector:**
```javascript
// Автоматически проверяет существование перед подпиской
const item = useSelector(state => state.items[id]);
if (!item) return null;
```

**3. MobX с observer:**
```javascript
// observer() оборачивает в try-catch
// Если данные удалены — компонент просто не рендерится
const Item = observer(({ id }) => {
  const item = store.items.get(id);
  if (!item) return null;
  return <div>{item.name}</div>;
});
```

**4. Ключи в списках:**
```javascript
// Правильные ключи помогают React правильно сопоставлять компоненты
{items.map(item => <Item key={item.id} id={item.id} />)}
```

**5. Batching обновлений:**
```javascript
// React 18 автоматически батчит
// Для синхронных операций — все обновления в одном рендере
```

---

### 6. CSS Isolation

**Вопрос:** Объясните принципы БЭМ, CSS Modules и CSS-in-JS. Как обеспечить изоляцию стилей?

**Ответ:**

**БЭМ (Block Element Modifier):**
```css
/* Конвенция именования для избежания конфликтов */
.block {}
.block__element {}
.block--modifier {}

/* Пример */
.card {}
.card__title {}
.card__button {}
.card__button--primary {}
.card__button--disabled {}
```
**Плюсы:** понятная структура, нет специальных инструментов
**Минусы:** длинные имена, ручное соблюдение конвенции

**CSS Modules:**
```javascript
// styles.module.css
.button { color: red; }

// Component.jsx
import styles from './styles.module.css';
<button className={styles.button}>Click</button>

// Результат в DOM:
// <button class="styles_button_x7f3k">Click</button>
```
**Плюсы:** автоматическая изоляция, локальные классы
**Минусы:** неудобная динамика, нет JS-переменных

**CSS-in-JS (styled-components, emotion):**
```javascript
const Button = styled.button`
  color: ${props => props.primary ? 'blue' : 'gray'};
  padding: ${props => props.theme.spacing.md};
`;

<Button primary>Click</Button>
```
**Плюсы:** динамические стили, темизация, JS-логика
**Минусы:** runtime overhead, больший бандл

**Сравнение:**

| Аспект | БЭМ | CSS Modules | CSS-in-JS |
|--------|-----|-------------|-----------|
| Изоляция | Конвенция | Автоматическая | Автоматическая |
| Динамика | Классы | Классы | Пропсы |
| Runtime | Нет | Нет | Есть |
| Bundle size | Минимальный | Минимальный | Больше |
| TypeScript | - | Плагины | Нативно |
| SSR | Просто | Просто | Сложнее |

**Современные альтернативы:**
- **Tailwind CSS:** utility-first, atomic CSS
- **Vanilla Extract:** CSS-in-JS с нулевым runtime
- **Linaria:** compile-time CSS-in-JS

---

### 7. Accessibility (A11y)

**Вопрос:** Какие принципы accessibility важны для веб-приложений? Как тестировать A11y?

**Ответ:**

**WCAG принципы (POUR):**
- **Perceivable:** контент воспринимаем всеми
- **Operable:** интерфейс управляем с клавиатуры
- **Understandable:** понятный контент и поведение
- **Robust:** работает с assistive technologies

**Ключевые практики:**

**1. Семантический HTML:**
```jsx
// Плохо
<div onClick={handleClick}>Click me</div>

// Хорошо
<button onClick={handleClick}>Click me</button>
```

**2. ARIA атрибуты:**
```jsx
<button 
  aria-label="Close dialog"
  aria-expanded={isOpen}
  aria-controls="menu-id"
>
  <CloseIcon />
</button>

<div role="alert" aria-live="polite">
  {errorMessage}
</div>
```

**3. Фокус и клавиатура:**
```jsx
// Управление фокусом
const dialogRef = useRef();
useEffect(() => {
  if (isOpen) dialogRef.current.focus();
}, [isOpen]);

// Focus trap в модалках
// Tab циклится внутри модалки
```

**4. Цветовой контраст:**
- Минимум 4.5:1 для обычного текста
- Минимум 3:1 для крупного текста
- Не только цвет для передачи информации

**5. Альтернативный текст:**
```jsx
<img src="photo.jpg" alt="Описание изображения" />
<img src="decorative.svg" alt="" role="presentation" />
```

**Инструменты тестирования:**

| Инструмент | Тип | Использование |
|------------|-----|---------------|
| axe-core | Автоматический | CI/CD, unit tests |
| Lighthouse | Автоматический | Аудит страницы |
| NVDA/VoiceOver | Ручной | Screen reader тестирование |
| Keyboard only | Ручной | Tab navigation |
| eslint-plugin-jsx-a11y | Линтер | Статический анализ |

**React Testing Library:**
```javascript
// Автоматически поощряет a11y
screen.getByRole('button', { name: /submit/i });
screen.getByLabelText('Email');
```

---

### 8. Server-Side Rendering

**Вопрос:** Объясните SSR, SSG и ISR. Когда использовать каждый подход?

**Ответ:**

**CSR (Client-Side Rendering):**
```
Browser → Пустой HTML → JS загрузка → JS рендерит → Контент виден
```
- Первая загрузка медленная
- SEO проблемы
- Интерактивность сразу после загрузки

**SSR (Server-Side Rendering):**
```
Browser → Сервер рендерит HTML → Контент виден → JS hydration → Интерактивность
```
- Быстрый First Contentful Paint
- Хороший SEO
- Нагрузка на сервер при каждом запросе

**SSG (Static Site Generation):**
```
Build time: Рендер всех страниц → Статичные HTML файлы
Runtime: CDN отдает готовый HTML
```
- Максимальная производительность
- Не подходит для динамических данных
- Долгий build при большом количестве страниц

**ISR (Incremental Static Regeneration):**
```
Первый запрос: отдать кешированный HTML
Фон: перегенерировать страницу
Следующий запрос: отдать обновленный HTML
```
- Баланс между SSG и SSR
- `revalidate` интервал
- Next.js специфичный

**Сравнение:**

| Аспект | CSR | SSR | SSG | ISR |
|--------|-----|-----|-----|-----|
| TTFB | Быстрый | Медленный | Очень быстрый | Очень быстрый |
| FCP | Медленный | Быстрый | Очень быстрый | Очень быстрый |
| SEO | Плохой | Хороший | Отличный | Отличный |
| Динамические данные | Да | Да | Нет | Частично |
| Серверная нагрузка | Нет | Высокая | Нет | Низкая |
| Персонализация | Да | Да | Нет | Частично |

**Когда использовать:**
- **CSR:** dashboards, админки, приложения за авторизацией
- **SSR:** e-commerce, контент зависит от пользователя
- **SSG:** блоги, документация, маркетинговые страницы
- **ISR:** каталоги, новости (обновление раз в N минут)

**React Server Components (RSC):**
- Компоненты рендерятся только на сервере
- Нет JS на клиенте для этих компонентов
- Прямой доступ к базе данных

---

### 9. Testing Strategies

**Вопрос:** Какой подход к тестированию предпочтителен для React? Какие инструменты?

**Ответ:**

**Пирамида тестирования:**
```
         /\
        /  \  E2E (Cypress, Playwright)
       /----\  10-20%
      /      \
     /  Integ \  Integration (RTL)
    /----------\  30-40%
   /            \
  /    Unit      \  Unit (Jest, Vitest)
 /----------------\  40-60%
```

**Unit Tests:**
```javascript
// Тестируем изолированную логику
// utils, hooks, reducers

// Jest + React Testing Library
test('formats currency correctly', () => {
  expect(formatCurrency(1000)).toBe('$1,000.00');
});

// Custom hooks
const { result } = renderHook(() => useCounter());
act(() => result.current.increment());
expect(result.current.count).toBe(1);
```

**Integration Tests:**
```javascript
// Тестируем компонент с его зависимостями
// React Testing Library поощряет этот подход

test('user can submit form', async () => {
  render(<ContactForm />);
  
  await userEvent.type(
    screen.getByLabelText(/email/i),
    'test@example.com'
  );
  await userEvent.click(screen.getByRole('button', { name: /submit/i }));
  
  expect(await screen.findByText(/success/i)).toBeInTheDocument();
});
```

**E2E Tests:**
```javascript
// Тестируем реальный user flow
// Cypress или Playwright

test('user can complete checkout', async ({ page }) => {
  await page.goto('/products');
  await page.click('[data-testid="add-to-cart"]');
  await page.click('[data-testid="checkout"]');
  await page.fill('#email', 'test@example.com');
  await page.click('button[type="submit"]');
  await expect(page.locator('.confirmation')).toBeVisible();
});
```

**Инструменты:**

| Категория | Инструмент | Назначение |
|-----------|------------|------------|
| Test Runner | Jest, Vitest | Запуск тестов |
| UI Testing | React Testing Library | Рендер компонентов |
| E2E | Cypress, Playwright | Браузерные тесты |
| Mocking | MSW | API mocking |
| Visual | Storybook, Chromatic | UI компоненты |
| Coverage | Istanbul | Покрытие кода |

**Принципы RTL:**
- Тестируй как пользователь
- Избегай тестирования implementation details
- Используй accessible queries (getByRole, getByLabelText)

---

### 10. Browser APIs

**Вопрос:** Объясните Intersection Observer, ResizeObserver, MutationObserver. Приведите практические примеры.

**Ответ:**

**Intersection Observer:**
Отслеживает пересечение элемента с viewport или другим элементом.

```javascript
// Lazy loading изображений
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      observer.unobserve(img);
    }
  });
}, {
  rootMargin: '100px', // Начать загрузку за 100px до viewport
  threshold: 0.1       // Триггер при 10% видимости
});

document.querySelectorAll('img[data-src]').forEach(img => {
  observer.observe(img);
});
```

**Применение:**
- Lazy loading
- Infinite scroll
- Анимации при скролле
- Аналитика видимости

**ResizeObserver:**
Отслеживает изменение размеров элемента.

```javascript
// Responsive компонент
const observer = new ResizeObserver((entries) => {
  entries.forEach(entry => {
    const { width, height } = entry.contentRect;
    if (width < 600) {
      entry.target.classList.add('compact');
    } else {
      entry.target.classList.remove('compact');
    }
  });
});

observer.observe(document.querySelector('.container'));
```

**Применение:**
- Container queries (до нативной поддержки)
- Canvas resize
- Виртуализация

**MutationObserver:**
Отслеживает изменения DOM-дерева.

```javascript
// Отслеживание динамического контента
const observer = new MutationObserver((mutations) => {
  mutations.forEach(mutation => {
    mutation.addedNodes.forEach(node => {
      if (node.matches?.('.ad')) {
        node.remove(); // Ad blocker пример
      }
    });
  });
});

observer.observe(document.body, {
  childList: true,    // Отслеживать добавление/удаление детей
  subtree: true,      // Включая все поддерево
  attributes: true,   // Отслеживать изменение атрибутов
});
```

**Применение:**
- Отслеживание сторонних скриптов
- Автоматическая инициализация виджетов
- Undo/Redo системы

**React hooks:**
```javascript
function useIntersection(ref, options) {
  const [isVisible, setIsVisible] = useState(false);
  
  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      setIsVisible(entry.isIntersecting);
    }, options);
    
    if (ref.current) observer.observe(ref.current);
    return () => observer.disconnect();
  }, [ref, options]);
  
  return isVisible;
}
```

---

## 💻 Практика / Live Coding

### 1. Custom Hook (useDebounce / useLocalStorage)

**Задача:** Создайте custom hook `useDebounce` или `useLocalStorage`.

**Архитектура решения:**

```
┌─────────────────────────────────────────────────────────────┐
│                     useDebounce Hook                         │
│                                                              │
│  Input: value, delay                                         │
│  Output: debouncedValue                                      │
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ value changes → setTimeout(delay) → update debouncedValue││
│  │       ↑                                                 ││
│  │       └── clearTimeout при новом изменении              ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Компоненты:                                                 │
│  • useState для хранения debouncedValue                      │
│  • useEffect с таймером                                      │
│  • cleanup function для clearTimeout                         │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                   useLocalStorage Hook                       │
│                                                              │
│  Input: key, initialValue                                    │
│  Output: [value, setValue]                                   │
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ Инициализация:                                          ││
│  │   1. Попробовать прочитать из localStorage              ││
│  │   2. Если нет — использовать initialValue               ││
│  │   3. Lazy initialization в useState                     ││
│  │                                                         ││
│  │ При изменении:                                          ││
│  │   1. Обновить state                                     ││
│  │   2. Синхронизировать с localStorage                    ││
│  │   3. Обработать ошибки (quota exceeded)                 ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Edge cases:                                                 │
│  • SSR (window undefined)                                    │
│  • JSON parse errors                                         │
│  • Storage events (синхронизация между табами)               │
└─────────────────────────────────────────────────────────────┘
```

**API:**
```typescript
// useDebounce
const debouncedSearchTerm = useDebounce(searchTerm, 500);

// useLocalStorage
const [theme, setTheme] = useLocalStorage('theme', 'light');
```

---

### 2. Accordion / Modal Component

**Задача:** Создайте компонент Accordion или Modal с учетом accessibility и анимации.

**Архитектура решения:**

```
┌─────────────────────────────────────────────────────────────┐
│                      Modal Architecture                      │
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                    Portal (document.body)               ││
│  │  ┌───────────────────────────────────────────────────┐  ││
│  │  │              Overlay (backdrop)                   │  ││
│  │  │  ┌─────────────────────────────────────────────┐  │  ││
│  │  │  │              Modal Content                  │  │  ││
│  │  │  │  ┌───────────────────────────────────────┐  │  │  ││
│  │  │  │  │ Header (title, close button)         │  │  │  ││
│  │  │  │  ├───────────────────────────────────────┤  │  │  ││
│  │  │  │  │ Body (children)                      │  │  │  ││
│  │  │  │  ├───────────────────────────────────────┤  │  │  ││
│  │  │  │  │ Footer (actions)                     │  │  │  ││
│  │  │  │  └───────────────────────────────────────┘  │  │  ││
│  │  │  └─────────────────────────────────────────────┘  │  ││
│  │  └───────────────────────────────────────────────────┘  ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘

  Accessibility:
  ┌─────────────────────────────────────────────────────────┐
  │ • role="dialog" aria-modal="true"                       │
  │ • aria-labelledby → заголовок                           │
  │ • aria-describedby → описание                           │
  │ • Focus trap (Tab циклится внутри)                      │
  │ • Escape закрывает                                      │
  │ • Возврат фокуса при закрытии                           │
  │ • body overflow: hidden при открытии                    │
  └─────────────────────────────────────────────────────────┘

  Animation:
  ┌─────────────────────────────────────────────────────────┐
  │ • Overlay: opacity 0 → 1                                │
  │ • Content: transform scale(0.95) → scale(1)             │
  │ • Exit animation перед unmount                          │
  │ • Использовать CSS transitions или framer-motion        │
  └─────────────────────────────────────────────────────────┘
```

**Ключевые компоненты:**
- **createPortal** для рендера вне DOM-иерархии
- **useRef** для focus management
- **useEffect** для keyboard events и body scroll lock
- **useState/useCallback** для анимации exit

---

### 3. Infinite Scroll

**Задача:** Реализуйте компонент с бесконечной прокруткой используя Intersection Observer.

**Архитектура решения:**

```
┌─────────────────────────────────────────────────────────────┐
│                  Infinite Scroll Architecture                │
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                    Viewport                             ││
│  │  ┌───────────────────────────────────────────────────┐  ││
│  │  │ Item 1                                            │  ││
│  │  ├───────────────────────────────────────────────────┤  ││
│  │  │ Item 2                                            │  ││
│  │  ├───────────────────────────────────────────────────┤  ││
│  │  │ Item 3                                            │  ││
│  │  └───────────────────────────────────────────────────┘  ││
│  │  ┌───────────────────────────────────────────────────┐  ││
│  │  │           Sentinel Element                        │◄─┼── IntersectionObserver
│  │  └───────────────────────────────────────────────────┘  ││         watches this
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Data Flow:                                                  │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ Sentinel visible → fetchNextPage() → append items       ││
│  │                                                         ││
│  │ State:                                                  ││
│  │ • items: T[]                                            ││
│  │ • page: number                                          ││
│  │ • loading: boolean                                      ││
│  │ • hasMore: boolean                                      ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Оптимизации:                                                │
│  • rootMargin: "100px" — предзагрузка                        │
│  • Debounce fetch calls                                      │
│  • Loading indicator                                         │
│  • Error handling + retry                                    │
└─────────────────────────────────────────────────────────────┘
```

**Компоненты:**
1. **useInfiniteScroll hook** — логика Observer + fetch
2. **Sentinel ref** — элемент в конце списка
3. **Loading/Error states** — UI feedback
4. **Виртуализация** (опционально) — для очень длинных списков

---

### 4. Form Validation

**Задача:** Создайте форму с кастомной валидацией без использования библиотек.

**Архитектура решения:**

```
┌─────────────────────────────────────────────────────────────┐
│                  Form Validation Architecture                │
│                                                              │
│  State Structure:                                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ {                                                       ││
│  │   values: { email: '', password: '' },                  ││
│  │   errors: { email: null, password: null },              ││
│  │   touched: { email: false, password: false },           ││
│  │   isSubmitting: false,                                  ││
│  │   isValid: false                                        ││
│  │ }                                                       ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Validation Schema:                                          │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ {                                                       ││
│  │   email: [                                              ││
│  │     { test: required, message: 'Email required' },      ││
│  │     { test: isEmail, message: 'Invalid email' }         ││
│  │   ],                                                    ││
│  │   password: [                                           ││
│  │     { test: required, message: 'Password required' },   ││
│  │     { test: minLength(8), message: 'Min 8 chars' }      ││
│  │   ]                                                     ││
│  │ }                                                       ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Event Handlers:                                             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ onChange:                                               ││
│  │   1. Update values[field]                               ││
│  │   2. Validate field (debounced)                         ││
│  │   3. Update errors[field]                               ││
│  │                                                         ││
│  │ onBlur:                                                 ││
│  │   1. Set touched[field] = true                          ││
│  │   2. Validate field                                     ││
│  │   3. Show error if touched && error                     ││
│  │                                                         ││
│  │ onSubmit:                                               ││
│  │   1. Touch all fields                                   ││
│  │   2. Validate all                                       ││
│  │   3. If valid → submit, else show errors                ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

**API хука:**
```typescript
const { values, errors, touched, handleChange, handleBlur, handleSubmit, isValid } = 
  useForm({
    initialValues: { email: '', password: '' },
    validationSchema,
    onSubmit: async (values) => { /* ... */ }
  });
```

---

### 5. Data Fetching Component

**Задача:** Реализуйте компонент с загрузкой данных, обработкой ошибок и состоянием loading.

**Архитектура решения:**

```
┌─────────────────────────────────────────────────────────────┐
│                  Data Fetching Architecture                  │
│                                                              │
│  useFetch Hook State Machine:                                │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                                                         ││
│  │     ┌─────────┐   fetch()   ┌─────────┐                 ││
│  │     │  IDLE   │────────────►│ LOADING │                 ││
│  │     └─────────┘             └────┬────┘                 ││
│  │          ▲                       │                      ││
│  │          │              ┌────────┴────────┐             ││
│  │          │              ▼                 ▼             ││
│  │          │        ┌─────────┐       ┌─────────┐         ││
│  │          └────────│ SUCCESS │       │  ERROR  │         ││
│  │          refetch  └─────────┘       └────┬────┘         ││
│  │                                          │              ││
│  │                                    retry │              ││
│  │                                          ▼              ││
│  │                                    ┌─────────┐          ││
│  │                                    │ LOADING │          ││
│  │                                    └─────────┘          ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  State:                                                      │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ {                                                       ││
│  │   data: T | null,                                       ││
│  │   error: Error | null,                                  ││
│  │   status: 'idle' | 'loading' | 'success' | 'error',     ││
│  │   isLoading: boolean,                                   ││
│  │   isError: boolean,                                     ││
│  │   isSuccess: boolean                                    ││
│  │ }                                                       ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
│  Features:                                                   │
│  • AbortController для отмены при unmount                    │
│  • Retry с exponential backoff                               │
│  • Cache (опционально)                                       │
│  • Refetch interval                                          │
│  • Stale-while-revalidate                                    │
└─────────────────────────────────────────────────────────────┘
```

**API:**
```typescript
const { data, error, isLoading, refetch } = useFetch<User[]>('/api/users', {
  retries: 3,
  retryDelay: 1000,
  onSuccess: (data) => console.log('Loaded', data),
  onError: (error) => console.error('Failed', error)
});

// В компоненте
if (isLoading) return <Skeleton />;
if (error) return <ErrorMessage error={error} onRetry={refetch} />;
return <UserList users={data} />;
```

**Ключевые компоненты:**
- useReducer для state machine
- useEffect с AbortController
- Exponential backoff для retry
- Cleanup при unmount/dependency change
