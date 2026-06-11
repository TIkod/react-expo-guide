# Тема 5: Хуки — useState, useEffect, useRef, useMemo, useCallback

## Что такое хуки?

Хуки — это функции с префиксом `use`, которые дают компонентам доступ к возможностям React (состояние, жизненный цикл и т.д.).

**Правила хуков:**
1. Вызывай только на верхнем уровне компонента (не внутри if/for/функций)
2. Вызывай только из React-компонентов или кастомных хуков

## useState — уже знаем (быстрый recap)

```tsx
const [count, setCount] = useState(0);
const [name, setName] = useState('');
const [user, setUser] = useState<User | null>(null);
```

Функциональное обновление (когда новый state зависит от старого):
```tsx
// Лучше так:
setCount(prev => prev + 1);

// Чем так (может быть баг при пакетных обновлениях):
setCount(count + 1);
```

## useEffect — побочные эффекты

`useEffect` запускает код **после** рендера компонента. Используется для:
- Загрузки данных (fetch)
- Подписок (таймеры, события)
- Синхронизации с внешними системами

### Базовый синтаксис

```tsx
useEffect(() => {
  // Код эффекта
  console.log('Компонент отрендерился');
});
// Без массива зависимостей = запускается ПОСЛЕ КАЖДОГО рендера
```

```tsx
useEffect(() => {
  console.log('Запустится один раз при монтировании');
}, []);
// Пустой массив = запускается один раз
```

```tsx
useEffect(() => {
  console.log('Запустится при изменении userId');
  // Загрузить данные пользователя...
}, [userId]);
// Массив зависимостей — запускается при изменении любого из них
```

### Загрузка данных

```tsx
import { useState, useEffect } from 'react';
import { View, Text, ActivityIndicator } from 'react-native';

type Post = { id: number; title: string; body: string };

function PostsScreen() {
  const [posts, setPosts] = useState<Post[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function loadPosts() {
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/posts?_limit=10');
        const data = await response.json();
        setPosts(data);
      } catch (err) {
        setError('Ошибка загрузки');
      } finally {
        setLoading(false);
      }
    }

    loadPosts();
  }, []);  // один раз при монтировании

  if (loading) return <ActivityIndicator size="large" />;
  if (error) return <Text style={{ color: 'red' }}>{error}</Text>;

  return (
    <View>
      {posts.map(post => (
        <Text key={post.id}>{post.title}</Text>
      ))}
    </View>
  );
}
```

### Функция очистки (cleanup)

Когда компонент размонтируется или перед следующим запуском эффекта — вызывается cleanup:

```tsx
useEffect(() => {
  const interval = setInterval(() => {
    setCount(prev => prev + 1);
  }, 1000);

  // Cleanup — очищает интервал при размонтировании:
  return () => clearInterval(interval);
}, []);
```

```tsx
useEffect(() => {
  const subscription = someEventEmitter.subscribe(handler);

  return () => subscription.unsubscribe();  // отписываемся
}, []);
```

### Зависимости — важно!

```tsx
function SearchResults({ query }: { query: string }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    // Запускается при изменении query
    fetch(`/api/search?q=${query}`)
      .then(res => res.json())
      .then(data => setResults(data));
  }, [query]);  // query в зависимостях!

  // ...
}
```

**Правило:** Всё что используешь внутри useEffect из области снаружи — добавь в массив зависимостей.

## useRef — ссылки без перерисовки

`useRef` хранит значение между рендерами, но **не вызывает перерисовку** при изменении.

### Два применения:

**1. Ссылка на компонент/DOM-элемент:**
```tsx
import { useRef } from 'react';
import { TextInput } from 'react-native';

function LoginForm() {
  const emailRef = useRef<TextInput>(null);
  const passwordRef = useRef<TextInput>(null);

  return (
    <View>
      <TextInput
        ref={emailRef}
        placeholder="Email"
        returnKeyType="next"
        onSubmitEditing={() => passwordRef.current?.focus()}
        // При нажатии Enter — фокус переходит на следующее поле
      />
      <TextInput
        ref={passwordRef}
        placeholder="Пароль"
        secureTextEntry
      />
      <Pressable onPress={() => emailRef.current?.focus()}>
        <Text>Перейти к Email</Text>
      </Pressable>
    </View>
  );
}
```

**2. Хранение значения без перерисовки:**
```tsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef<ReturnType<typeof setInterval> | null>(null);

  const start = () => {
    intervalRef.current = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
  };

  const stop = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
    }
  };

  return (
    <View>
      <Text>{seconds}с</Text>
      <Pressable onPress={start}><Text>Старт</Text></Pressable>
      <Pressable onPress={stop}><Text>Стоп</Text></Pressable>
    </View>
  );
}
```

## useMemo — кэширование вычислений

Кэширует результат вычисления. Пересчитывает только когда меняются зависимости.

```tsx
import { useMemo } from 'react';

function ProductList({ products, filter }: Props) {
  // Без useMemo — фильтрация выполняется при каждом рендере
  // С useMemo — только когда меняются products или filter
  const filteredProducts = useMemo(() => {
    return products.filter(p =>
      p.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [products, filter]);

  return (
    <FlatList data={filteredProducts} ... />
  );
}
```

**Когда использовать:** Тяжёлые вычисления (сортировка/фильтрация больших массивов, сложная математика). Не злоупотребляй — useMemo сам имеет стоимость.

## useCallback — кэширование функций

Кэширует функцию, чтобы её референс не менялся при каждом рендере.

```tsx
import { useCallback } from 'react';

function Parent() {
  const [items, setItems] = useState([...]);

  // Без useCallback — новая функция при каждом рендере Parent
  // С useCallback — та же функция пока items не изменится
  const handleDelete = useCallback((id: number) => {
    setItems(prev => prev.filter(item => item.id !== id));
  }, []);  // пустой массив т.к. используем функциональный setState

  return <ItemList items={items} onDelete={handleDelete} />;
}
```

**Когда использовать:** Когда передаёшь функцию в дочерний компонент обёрнутый в `React.memo`, или как зависимость в useEffect.

## Кастомные хуки

Можно вынести логику с хуками в отдельную функцию — это кастомный хук.

```tsx
// hooks/useCounter.ts
import { useState } from 'react';

function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(prev => prev + 1);
  const decrement = () => setCount(prev => prev - 1);
  const reset = () => setCount(initialValue);

  return { count, increment, decrement, reset };
}

// Использование:
function CounterScreen() {
  const { count, increment, decrement, reset } = useCounter(10);

  return (
    <View>
      <Text>{count}</Text>
      <Pressable onPress={increment}><Text>+</Text></Pressable>
      <Pressable onPress={decrement}><Text>-</Text></Pressable>
      <Pressable onPress={reset}><Text>Сброс</Text></Pressable>
    </View>
  );
}
```

### Кастомный хук для загрузки данных

```tsx
// hooks/useFetch.ts
import { useState, useEffect } from 'react';

function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    setLoading(true);
    fetch(url)
      .then(res => {
        if (!res.ok) throw new Error('Ошибка сети');
        return res.json();
      })
      .then(data => setData(data))
      .catch(err => setError(err.message))
      .finally(() => setLoading(false));
  }, [url]);

  return { data, loading, error };
}

// Использование:
function UserScreen({ userId }: { userId: number }) {
  const { data: user, loading, error } = useFetch<User>(
    `https://api.example.com/users/${userId}`
  );

  if (loading) return <ActivityIndicator />;
  if (error) return <Text>{error}</Text>;
  if (!user) return null;

  return <Text>{user.name}</Text>;
}
```

---

## Практика

### Задание 5.1 — Секундомер

Создай компонент секундомера:
- Отображает время в формате `MM:SS`
- Кнопки: Старт, Стоп, Сброс
- Используй `useRef` для хранения intervalId
- Используй `useState` для времени

### Задание 5.2 — Поиск с debounce

Создай экран поиска:
- TextInput для ввода запроса
- Через 500мс после последнего ввода — делай fetch к `https://jsonplaceholder.typicode.com/posts?title_like={query}`
- Показывай список результатов или "Ничего не найдено"
- Во время загрузки показывай ActivityIndicator

Подсказка: используй `useEffect` + `setTimeout` + cleanup для debounce.

---

## Дополнительно (попробуй сам)

**Задача:** Создай кастомный хук `useLocalStorage` (замена AsyncStorage для веб-части) или `useDebounce`:

```tsx
// hooks/useDebounce.ts
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// Использование:
const debouncedQuery = useDebounce(query, 500);

useEffect(() => {
  // fetchData(debouncedQuery);
}, [debouncedQuery]);
```

---

## Итог темы

- `useEffect` — запуск кода после рендера: загрузка данных, подписки, таймеры
- Массив зависимостей контролирует когда эффект запускается
- Cleanup функция — очищает ресурсы при размонтировании
- `useRef` — значение без перерисовки + ссылки на компоненты
- `useMemo` — кэширование тяжёлых вычислений
- `useCallback` — кэширование функций для стабильного референса
- Кастомный хук — функция `use*` выносящая переиспользуемую логику

**Следующая тема:** [06-navigation.md](./06-navigation.md)
