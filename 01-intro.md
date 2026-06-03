# Разработано организацией TICODE (https://ticode.online/)

# Тема 1: Введение в React Native и Expo

## Что такое React Native?

React Native — это фреймворк от Meta для создания мобильных приложений на JavaScript/TypeScript с использованием синтаксиса React. Ключевое отличие от веба:

- В браузере React рендерит `<div>`, `<span>`, `<p>`
- В React Native рендерятся **нативные** компоненты iOS и Android (`UIView`, `android.view.View`)

Ты пишешь один JavaScript-код, а приложение работает на обеих платформах как настоящее нативное приложение — не WebView, не оболочка.

```
Твой JS/TS код
      ↓
  React Native Bridge / JSI
      ↓
Нативные компоненты
iOS (Swift/ObjC)   Android (Kotlin/Java)
```

## Что такое Expo?

Expo — это платформа поверх React Native, которая:
1. Убирает необходимость настраивать Xcode / Android Studio с нуля
2. Предоставляет готовые API (камера, геолокация, уведомления и т.д.)
3. Даёт CLI для быстрого старта и запуска
4. Позволяет тестировать на телефоне через приложение **Expo Go**

**Expo SDK 52** (2026) — актуальная версия.

### Два режима Expo

| Режим | Описание | Когда использовать |
|-------|----------|-------------------|
| Managed Workflow | Expo управляет нативным кодом | Большинство приложений, Junior-уровень |
| Bare Workflow | Есть доступ к нативному коду (ios/, android/) | Когда нужны кастомные нативные модули |

На этом курсе используем **Managed Workflow**.

## Установка окружения

### Требования
- Node.js 20+ (LTS)
- npm или yarn или pnpm

### Установка Expo CLI

```bash
npm install -g expo-cli
# или используй npx (рекомендуется, не нужно глобально)
```

### Создание нового проекта

```bash
npx create-expo-app@latest my-app
cd my-app
```

При создании выбери шаблон:
- **Default** — с TypeScript и Expo Router (рекомендуется)
- Blank — минимальный, без навигации

### Структура проекта (Default шаблон)

```
my-app/
├── app/                  ← папка для экранов (Expo Router)
│   ├── (tabs)/
│   │   ├── index.tsx     ← главный экран
│   │   └── explore.tsx
│   └── _layout.tsx       ← корневой layout
├── assets/               ← шрифты, картинки
├── components/           ← переиспользуемые компоненты
├── constants/            ← цвета, константы
├── hooks/                ← кастомные хуки
├── app.json              ← конфиг приложения
├── package.json
└── tsconfig.json
```

## Запуск проекта

```bash
cd my-app
npx expo start
```

После запуска увидишь QR-код в терминале:
- На телефоне: установи **Expo Go** (iOS/Android) и сканируй QR
- В браузере: нажми `w` — откроется веб-версия
- В эмуляторе: `i` — iOS симулятор, `a` — Android эмулятор

## Как работает Expo Go

```
Телефон с Expo Go         Твой компьютер
       ↑                        |
  Wi-Fi / USB            npx expo start
       ↑                        |
   QR-код  ←──────────  Metro Bundler
                         (сборщик кода)
```

Metro Bundler собирает твой JS-код и отправляет на телефон. При изменении файла — автообновление (Hot Reload).

## Разница: веб vs React Native

| Веб (HTML/CSS) | React Native |
|----------------|--------------|
| `<div>` | `<View>` |
| `<p>`, `<span>` | `<Text>` |
| `<img>` | `<Image>` |
| `<input>` | `<TextInput>` |
| `<button>` | `<TouchableOpacity>` / `<Pressable>` |
| `<ul>`, `<li>` | `<FlatList>` |
| CSS классы | StyleSheet объекты |
| Flexbox (строки) | Flexbox (колонки по умолчанию!) |

**Важно:** В React Native Flexbox работает вертикально по умолчанию (`flexDirection: 'column'`). В CSS по умолчанию — горизонтально.

## Первый экран — что видишь в шаблоне

Открой `app/(tabs)/index.tsx`. Там что-то вроде:

```tsx
import { Text, View } from 'react-native';

export default function HomeScreen() {
  return (
    <View>
      <Text>Привет, мир!</Text>
    </View>
  );
}
```

Это и есть компонент — функция, возвращающая JSX с нативными элементами.

---

## Практика

### Задание 1.1 — Первый запуск

1. Установи Node.js 20 LTS если ещё не установлен
2. Создай новый проект: `npx create-expo-app@latest my-first-app`
3. Запусти: `npx expo start`
4. Открой на телефоне через Expo Go или в браузере
5. Измени текст в `app/(tabs)/index.tsx` на своё имя, убедись что изменение отображается сразу

### Задание 1.2 — Изучи структуру

Открой файлы и ответь себе на вопросы:
- Что делает `app/_layout.tsx`?
- Что находится в `app.json` — найди поле `name` и поменяй название приложения
- Что за папка `assets/`?

---

## Дополнительно (попробуй сам)

**Задача:** Создай второй проект с нуля, но на этот раз выбери **Blank** шаблон.

```bash
npx create-expo-app@latest my-blank-app --template blank
```

Сравни структуру с Default шаблоном:
- Что есть в Default, чего нет в Blank?
- Попробуй добавить экран — создай файл `app/about.tsx` с простым текстом
- Перейди по URL `localhost:8081/about` в браузере — что видишь?

Это первое знакомство с Expo Router (подробно — в Теме 6).

---

## Итог темы

- React Native рендерит нативные компоненты, не HTML
- Expo упрощает разработку и даёт готовые API
- `npx create-expo-app` — команда для старта
- `npx expo start` — запуск сервера разработки
- Expo Go — приложение для тестирования на телефоне
- Основные компоненты: `View`, `Text`, `Image`, `TextInput`, `Pressable`

**Следующая тема:** [02-components-jsx.md](./02-components-jsx.md)
