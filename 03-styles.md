# Разработано организацией TICODE (https://ticode.online/)

# Тема 3: Стили и StyleSheet

## Как работают стили в React Native

В React Native нет CSS-файлов. Стили — это JavaScript объекты. Свойства написаны в camelCase.

```tsx
// CSS:
// background-color: red;
// font-size: 16px;

// React Native:
{ backgroundColor: 'red', fontSize: 16 }
```

**Единицы измерения:** Нет `px`, `em`, `rem` — только числа. Числа — это density-independent pixels (dp), автоматически масштабируются под плотность экрана.

## StyleSheet.create

```tsx
import { View, Text, StyleSheet } from 'react-native';

export default function MyScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Заголовок</Text>
      <Text style={styles.subtitle}>Подзаголовок</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#1a1a1a',
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
    marginTop: 8,
  },
});
```

`StyleSheet.create` — это не обязательно, но рекомендуется:
- Валидирует стили в dev-режиме
- Оптимизирует производительность

## Инлайн стили

Можно передавать объект прямо в `style`:

```tsx
<Text style={{ fontSize: 18, color: 'blue' }}>Текст</Text>
```

Удобно для динамических стилей, но не злоупотребляй — каждый рендер создаёт новый объект.

## Комбинирование стилей

```tsx
// Массив стилей — последний побеждает
<Text style={[styles.text, styles.bold, { color: 'red' }]}>
  Текст
</Text>

// Условный стиль
<View style={[styles.box, isActive && styles.activeBox]}>
```

## Flexbox в React Native

React Native использует Flexbox для всех лейаутов. Отличие от веба:

| Свойство | Веб (CSS) | React Native |
|----------|-----------|--------------|
| `flexDirection` | `row` по умолчанию | **`column` по умолчанию** |
| `alignContent` | `stretch` | `flex-start` |
| Все элементы | не flex | **flex по умолчанию** |

### flex: 1

Самый важный приём — растянуть элемент на всё доступное пространство:

```tsx
<View style={{ flex: 1 }}>
  {/* Займёт всю высоту родителя */}
</View>
```

### Основная ось (flexDirection)

```tsx
// Вертикально (по умолчанию):
<View style={{ flexDirection: 'column' }}>
  <Text>Сверху</Text>
  <Text>Снизу</Text>
</View>

// Горизонтально:
<View style={{ flexDirection: 'row' }}>
  <Text>Слева</Text>
  <Text>Справа</Text>
</View>
```

### justifyContent — выравнивание по основной оси

```tsx
<View style={{ flex: 1, justifyContent: 'center' }}>
  {/* Дети по центру вертикально (при flexDirection: 'column') */}
</View>

// Значения:
// 'flex-start'    — в начало (по умолчанию)
// 'flex-end'      — в конец
// 'center'        — по центру
// 'space-between' — равные отступы между
// 'space-around'  — равные отступы вокруг
// 'space-evenly'  — полностью равные отступы
```

### alignItems — выравнивание по поперечной оси

```tsx
<View style={{ flex: 1, flexDirection: 'row', alignItems: 'center' }}>
  {/* Дети выровнены по центру вертикально */}
</View>

// Значения:
// 'stretch'    — растянуть (по умолчанию)
// 'flex-start' — в начало
// 'flex-end'   — в конец
// 'center'     — по центру
```

### Типичные лейауты

```tsx
// Центрировать по экрану:
<View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
  <Text>По центру экрана</Text>
</View>

// Шапка + контент + подвал:
<View style={{ flex: 1 }}>
  <View style={{ height: 60, backgroundColor: '#007AFF' }}>
    {/* Шапка */}
  </View>
  <View style={{ flex: 1 }}>
    {/* Контент — займёт всё оставшееся место */}
  </View>
  <View style={{ height: 80 }}>
    {/* Подвал */}
  </View>
</View>

// Горизонтальный ряд с иконкой и текстом:
<View style={{ flexDirection: 'row', alignItems: 'center', gap: 12 }}>
  <Image source={...} style={{ width: 40, height: 40 }} />
  <Text>Имя пользователя</Text>
</View>
```

## Свойства отступов и размеров

```tsx
{
  // Внешние отступы:
  margin: 16,           // все стороны
  marginTop: 8,
  marginBottom: 8,
  marginLeft: 16,
  marginRight: 16,
  marginHorizontal: 16, // left + right
  marginVertical: 8,    // top + bottom

  // Внутренние отступы:
  padding: 16,
  paddingTop: 8,
  paddingHorizontal: 16,
  // (аналогично margin)

  // Размеры:
  width: 100,
  height: 50,
  minWidth: 50,
  maxWidth: 300,

  // Процентные размеры:
  width: '50%',   // 50% от родителя
  height: '100%',

  // gap (между дочерними элементами):
  gap: 12,        // и по горизонтали, и по вертикали
  rowGap: 8,
  columnGap: 16,
}
```

## Текстовые стили

```tsx
{
  fontSize: 16,
  fontWeight: 'bold',     // '100' до '900' или 'bold', 'normal'
  fontStyle: 'italic',
  color: '#333',
  textAlign: 'center',    // 'left', 'right', 'center', 'justify'
  lineHeight: 24,
  letterSpacing: 1,
  textDecorationLine: 'underline', // 'none', 'underline', 'line-through'
  textTransform: 'uppercase',      // 'none', 'uppercase', 'lowercase', 'capitalize'
}
```

## Границы и скругления

```tsx
{
  borderWidth: 1,
  borderColor: '#ccc',
  borderRadius: 8,        // все углы
  borderTopLeftRadius: 16,

  // Только одна сторона:
  borderTopWidth: 1,
  borderBottomColor: '#eee',

  // Тень (iOS):
  shadowColor: '#000',
  shadowOffset: { width: 0, height: 2 },
  shadowOpacity: 0.1,
  shadowRadius: 4,

  // Тень (Android):
  elevation: 4,

  // Используй оба для кроссплатформенности
}
```

## Адаптивные размеры с Dimensions

```tsx
import { Dimensions } from 'react-native';

const { width, height } = Dimensions.get('window');

const styles = StyleSheet.create({
  halfScreen: {
    width: width / 2,
    height: height / 3,
  },
});
```

## useWindowDimensions (предпочтительно)

```tsx
import { useWindowDimensions } from 'react-native';

function MyComponent() {
  const { width, height } = useWindowDimensions();
  // Автоматически обновляется при повороте экрана

  return (
    <View style={{ width: width * 0.9 }}>
      <Text>90% ширины экрана</Text>
    </View>
  );
}
```

## Platform — разные стили для iOS и Android

```tsx
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  header: {
    paddingTop: Platform.OS === 'ios' ? 44 : 24,
    backgroundColor: Platform.select({
      ios: '#007AFF',
      android: '#6200EE',
    }),
  },
});
```

## Тема и цвета — хорошая практика

Создай файл `constants/Colors.ts`:

```ts
export const Colors = {
  primary: '#007AFF',
  secondary: '#5856D6',
  background: '#F2F2F7',
  surface: '#FFFFFF',
  text: '#1C1C1E',
  textSecondary: '#8E8E93',
  border: '#C6C6C8',
  error: '#FF3B30',
  success: '#34C759',
};
```

Используй везде через импорт — так легко менять тему.

---

## Практика

### Задание 3.1 — Карточка пользователя со стилями

Стилизуй компонент `ProfileCard` из Темы 2:
- Белый фон, скругление 12, тень
- Аватар — круглый (borderRadius = половина размера)
- Имя — жирный шрифт 18px
- Email — серый 14px
- Кнопка "Написать" — синяя с белым текстом, скруглённая

### Задание 3.2 — Экран входа (макет)

Создай экран `app/login.tsx` только с вёрсткой (без логики):
- Логотип по центру сверху
- Два поля: Email и Пароль
- Кнопка "Войти" на всю ширину
- Текст "Нет аккаунта? Зарегистрируйтесь" снизу по центру

Используй `flex: 1, justifyContent: 'center'` для центрирования.

---

## Дополнительно (попробуй сам)

**Задача:** Адаптивная сетка карточек.

Сделай экран с сеткой карточек (2 в ряд на узком экране, 3 на широком):

```tsx
import { useWindowDimensions } from 'react-native';

function GridScreen() {
  const { width } = useWindowDimensions();
  const columns = width > 600 ? 3 : 2;
  const cardWidth = (width - 16 * (columns + 1)) / columns;

  // Рендери карточки с этой шириной
}
```

---

## Итог темы

- Стили = JavaScript объекты с camelCase свойствами
- `StyleSheet.create` — рекомендуемый способ создания стилей
- Flexbox — единственный способ лейаута, `column` по умолчанию
- `flex: 1` — растянуть на всё доступное место
- `justifyContent` — основная ось, `alignItems` — поперечная
- `Platform.OS` / `Platform.select` — разные стили под платформы
- Вынеси цвета в константы

**Следующая тема:** [04-state-props.md](./04-state-props.md)
