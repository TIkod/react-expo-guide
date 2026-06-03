# Разработано организацией TICODE (https://ticode.online/)

# Тема 2: Основы компонентов и JSX

## Что такое компонент?

Компонент — это функция, которая возвращает JSX (описание интерфейса). Это строительный блок любого React Native приложения.

```tsx
// Простейший компонент
function Greeting() {
  return <Text>Привет!</Text>;
}
```

Компоненты можно вкладывать друг в друга, передавать в них данные, переиспользовать.

## JSX — что это?

JSX — это синтаксис, похожий на HTML, но это не HTML. Под капотом JSX превращается в вызовы функций:

```tsx
// Что ты пишешь:
<Text style={{ color: 'red' }}>Привет</Text>

// Что это на самом деле:
React.createElement(Text, { style: { color: 'red' } }, 'Привет')
```

### Правила JSX

**1. Один корневой элемент**
```tsx
// Ошибка:
return (
  <Text>Первый</Text>
  <Text>Второй</Text>
);

// Правильно — обернуть в View:
return (
  <View>
    <Text>Первый</Text>
    <Text>Второй</Text>
  </View>
);

// Или использовать Fragment (не создаёт лишний элемент):
return (
  <>
    <Text>Первый</Text>
    <Text>Второй</Text>
  </>
);
```

**2. Самозакрывающиеся теги**
```tsx
// Если нет children — закрывай сразу:
<Image source={require('./photo.png')} />
<TextInput placeholder="Введи текст" />
```

**3. JavaScript внутри `{}`**
```tsx
const name = 'Алексей';
const age = 25;

return (
  <View>
    <Text>Привет, {name}!</Text>
    <Text>Тебе {age} лет</Text>
    <Text>Через 10 лет будет {age + 10}</Text>
  </View>
);
```

**4. Условный рендеринг**
```tsx
const isLoggedIn = true;

return (
  <View>
    {/* Тернарный оператор */}
    {isLoggedIn ? <Text>Добро пожаловать!</Text> : <Text>Войдите в систему</Text>}

    {/* Короткое замыкание — рендерит только если true */}
    {isLoggedIn && <Text>Кнопка для авторизованных</Text>}
  </View>
);
```

**5. Рендеринг списков**
```tsx
const fruits = ['Яблоко', 'Банан', 'Вишня'];

return (
  <View>
    {fruits.map((fruit, index) => (
      <Text key={index}>{fruit}</Text>  // key обязателен!
    ))}
  </View>
);
```

## Основные встроенные компоненты

### View
Аналог `<div>`. Контейнер для других компонентов.

```tsx
import { View } from 'react-native';

<View style={{ flex: 1, backgroundColor: '#fff' }}>
  {/* дочерние компоненты */}
</View>
```

### Text
Весь текст ДОЛЖЕН быть внутри `<Text>`. Нельзя просто написать текст в `<View>`.

```tsx
import { Text } from 'react-native';

<Text>Обычный текст</Text>
<Text style={{ fontSize: 24, fontWeight: 'bold' }}>Заголовок</Text>
<Text numberOfLines={2}>Текст который обрежется после двух строк...</Text>
```

### Image
Изображения из локальных файлов или URL.

**`require()` — встроен в React Native, ничего подключать не нужно.**

Путь в `require()` — относительный от текущего файла. Рекомендуется хранить картинки в `assets/images/`:

```
my-app/
├── assets/
│   └── images/
│       └── logo.png   ← сюда
├── app/
└── components/
```

**Важно: всегда указывай `width` и `height`, иначе картинка не отобразится.**

```tsx
import { Image } from 'react-native';

// Локальный файл — путь относительный от текущего файла
// Если ты в components/Card.tsx, а фото в assets/images/logo.png:
<Image source={require('../assets/images/logo.png')} style={{ width: 100, height: 100 }} />

// Если картинка лежит рядом с компонентом:
<Image source={require('./photo.png')} style={{ width: 100, height: 100 }} />

// URL — require не нужен, используй объект с uri:
<Image
  source={{ uri: 'https://example.com/photo.jpg' }}
  style={{ width: 200, height: 200 }}
/>
```

### TextInput
Поле для ввода текста.

```tsx
import { TextInput } from 'react-native';

<TextInput
  placeholder="Введи имя"
  style={{ borderWidth: 1, padding: 8 }}
  onChangeText={(text) => console.log(text)}
/>
```

### Pressable (и TouchableOpacity)
Любой кликабельный элемент.

```tsx
import { Pressable, Text } from 'react-native';

<Pressable onPress={() => console.log('нажали!')}>
  <Text>Нажми меня</Text>
</Pressable>

// С визуальной обратной связью через стиль:
<Pressable
  style={({ pressed }) => ({
    opacity: pressed ? 0.5 : 1,
    backgroundColor: 'blue',
    padding: 12,
  })}
  onPress={() => alert('Привет!')}
>
  <Text style={{ color: 'white' }}>Кнопка</Text>
</Pressable>
```

### ScrollView
Прокручиваемый контейнер. Используй когда контент может не влезть на экран.

```tsx
import { ScrollView, Text } from 'react-native';

<ScrollView>
  <Text>Много текста...</Text>
  <Text>Ещё текст...</Text>
  {/* ... */}
</ScrollView>
```

**Важно:** ScrollView загружает все элементы сразу. Для длинных списков — используй `FlatList` (Тема 7).

## Создание своего компонента

### Базовая структура файла компонента

```tsx
// components/Card.tsx

import { View, Text, StyleSheet } from 'react-native';

// Типизируем пропсы (TypeScript)
type CardProps = {
  title: string;
  description: string;
};

export default function Card({ title, description }: CardProps) {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>{title}</Text>
      <Text style={styles.description}>{description}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
    borderRadius: 8,
    padding: 16,
    marginBottom: 12,
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
  },
  description: {
    fontSize: 14,
    color: '#666',
    marginTop: 4,
  },
});
```

### Использование компонента

```tsx
// app/(tabs)/index.tsx

import { View } from 'react-native';
import Card from '@/components/Card';

export default function HomeScreen() {
  return (
    <View style={{ flex: 1, padding: 16 }}>
      <Card title="Первая карточка" description="Описание первой карточки" />
      <Card title="Вторая карточка" description="Описание второй карточки" />
    </View>
  );
}
```

## Children — дочерние элементы

Компонент может принимать дочерние элементы через `children`:

```tsx
// components/Container.tsx

import { View } from 'react-native';
import { ReactNode } from 'react';

type ContainerProps = {
  children: ReactNode;
};

export default function Container({ children }: ContainerProps) {
  return (
    <View style={{ flex: 1, padding: 16, backgroundColor: '#f5f5f5' }}>
      {children}
    </View>
  );
}

// Использование:
<Container>
  <Text>Этот текст будет внутри Container</Text>
  <Card title="Карточка" description="Тоже внутри Container" />
</Container>
```

## Именование и организация

- Компоненты — с большой буквы: `Card`, `UserAvatar`, `LoginButton`
- Файлы компонентов — с большой буквы: `Card.tsx`, `UserAvatar.tsx`
- Файлы экранов (в папке `app/`) — с маленькой: `index.tsx`, `profile.tsx`
- Один компонент — один файл

---

## Практика

### Задание 2.1 — Компонент ProfileCard

Создай компонент `components/ProfileCard.tsx` который показывает:
- Аватар (можно любую картинку)
- Имя пользователя
- Email
- Кнопку "Написать"

Пропсы: `name: string`, `email: string`, `avatarUrl: string`

Используй его на главном экране с двумя разными пользователями.

### Задание 2.2 — Условный рендеринг

Создай компонент `components/StatusBadge.tsx`:
- Принимает `status: 'online' | 'offline' | 'busy'`
- Показывает точку нужного цвета (зелёный/серый/красный) и текст статуса

```tsx
// Пример использования:
<StatusBadge status="online" />   // зелёная точка + "В сети"
<StatusBadge status="offline" />  // серая точка + "Не в сети"
<StatusBadge status="busy" />     // красная точка + "Занят"
```

---

## Дополнительно (попробуй сам)

**Задача:** Создай компонент `Button.tsx` — кастомную кнопку.

Требования:
- Пропсы: `title: string`, `onPress: () => void`, `variant: 'primary' | 'secondary'`
- `primary` — синяя кнопка с белым текстом
- `secondary` — белая кнопка с синей обводкой и синим текстом
- При нажатии должна слегка уменьшать прозрачность (используй `Pressable`)

Это паттерн который ты будешь использовать в каждом проекте.

---

## Итог темы

- Компонент = функция, возвращающая JSX
- JSX — не HTML, но похож синтаксически
- Основные компоненты: `View`, `Text`, `Image`, `TextInput`, `Pressable`, `ScrollView`
- `{}` внутри JSX — для JavaScript выражений
- `key` обязателен при рендере списков через `.map()`
- Типизируй пропсы через TypeScript тип или интерфейс

**Следующая тема:** [03-styles.md](./03-styles.md)
