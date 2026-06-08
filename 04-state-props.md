# Разработано организацией TICODE (https://ticode.online/)

# Тема 4: State и Props

## Props — данные снаружи

Props (properties) — это данные, которые компонент получает от родителя. Родитель передаёт, дочерний компонент читает. Props **неизменяемы** внутри компонента.

```tsx
// Определяем компонент с пропсами:
type GreetingProps = {
  name: string;
  age?: number;  // ? = необязательный
};

function Greeting({ name, age }: GreetingProps) {
  return (
    <Text>
      Привет, {name}! {age ? `Тебе ${age} лет.` : ''}
    </Text>
  );
}

// Используем:
<Greeting name="Алёша" age={25} />
<Greeting name="Маша" />  // age не передан — undefined
```

### Дефолтные значения пропсов

```tsx
type ButtonProps = {
  title: string;
  color?: string;
  size?: 'small' | 'medium' | 'large';
};

function Button({ title, color = '#007AFF', size = 'medium' }: ButtonProps) {
  const fontSize = size === 'small' ? 12 : size === 'large' ? 20 : 16;

  return (
    <Pressable style={{ backgroundColor: color, padding: 12, borderRadius: 8 }}>
      <Text style={{ color: '#fff', fontSize }}>{title}</Text>
    </Pressable>
  );
}
```

### Функции как пропсы (callbacks)

```tsx
type ItemProps = {
  title: string;
  onPress: () => void;
  onLongPress?: () => void;
};

function Item({ title, onPress, onLongPress }: ItemProps) {
  return (
    <Pressable onPress={onPress} onLongPress={onLongPress}>
      <Text>{title}</Text>
    </Pressable>
  );
}

// Использование:
<Item
  title="Удалить"
  onPress={() => console.log('нажали')}
  onLongPress={() => console.log('долгое нажатие')}
/>
```

## State — данные внутри

State — это данные которые компонент хранит и изменяет сам. При изменении state компонент **перерисовывается**.

```tsx
import { useState } from 'react';
import { View, Text, Pressable } from 'react-native';

function Counter() {
  const [count, setCount] = useState(0);  // начальное значение = 0

  return (
    <View>
      <Text>Счётчик: {count}</Text>
      <Pressable onPress={() => setCount(count + 1)}>
        <Text>+1</Text>
      </Pressable>
      <Pressable onPress={() => setCount(count - 1)}>
        <Text>-1</Text>
      </Pressable>
      <Pressable onPress={() => setCount(0)}>
        <Text>Сброс</Text>
      </Pressable>
    </View>
  );
}
```

### Анатомия useState

```tsx
const [value, setValue] = useState(initialValue);
//     ↑           ↑              ↑
//  текущее    функция         начальное
//  значение   изменения       значение
```

### State с объектом

```tsx
type User = {
  name: string;
  email: string;
  age: number;
};

function ProfileEditor() {
  const [user, setUser] = useState<User>({
    name: 'Иван',
    email: 'ivan@example.com',
    age: 30,
  });

  const updateName = (name: string) => {
    // Важно: всегда создавай новый объект, не мутируй старый!
    setUser({ ...user, name });
  };

  return (
    <View>
      <TextInput value={user.name} onChangeText={updateName} />
      <Text>Email: {user.email}</Text>
    </View>
  );
}
```

### State с массивом

```tsx
function TodoList() {
  const [todos, setTodos] = useState<string[]>([]);
  const [input, setInput] = useState('');

  const addTodo = () => {
    if (input.trim()) {
      setTodos([...todos, input]);  // новый массив!
      setInput('');
    }
  };

  const removeTodo = (index: number) => {
    setTodos(todos.filter((_, i) => i !== index));
  };

  return (
    <View>
      <TextInput value={input} onChangeText={setInput} />
      <Pressable onPress={addTodo}>
        <Text>Добавить</Text>
      </Pressable>
      {todos.map((todo, index) => (
        <Pressable key={index} onPress={() => removeTodo(index)}>
          <Text>{todo}</Text>
        </Pressable>
      ))}
    </View>
  );
}
```

## Поднятие состояния (Lifting State Up)

Когда двум компонентам нужен один и тот же state — выноси его в их общего родителя.

```tsx
// Плохо — у каждого компонента свой state:
function Parent() {
  return (
    <View>
      <ComponentA />  {/* свой count */}
      <ComponentB />  {/* свой count, не знает про A */}
    </View>
  );
}

// Хорошо — state в родителе:
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <View>
      <ComponentA count={count} onIncrement={() => setCount(count + 1)} />
      <ComponentB count={count} />  {/* видит тот же count */}
    </View>
  );
}

function ComponentA({ count, onIncrement }: { count: number; onIncrement: () => void }) {
  return (
    <View>
      <Text>{count}</Text>
      <Pressable onPress={onIncrement}><Text>+1</Text></Pressable>
    </View>
  );
}

function ComponentB({ count }: { count: number }) {
  return <Text>Итого: {count}</Text>;
}
```

## Управляемые компоненты (Controlled)

TextInput в React Native должен быть **управляемым** — его значение хранится в state:

```tsx
function SearchBar() {
  const [query, setQuery] = useState('');

  return (
    <View>
      <TextInput
        value={query}          // ← берём из state
        onChangeText={setQuery} // ← обновляем state при вводе
        placeholder="Поиск..."
      />
      {query.length > 0 && (
        <Pressable onPress={() => setQuery('')}>
          <Text>Очистить</Text>
        </Pressable>
      )}
      <Text>Запрос: {query}</Text>
    </View>
  );
}
```

## Производный state — не хранить лишнее

Если значение можно вычислить из существующего state — вычисляй при рендере, не храни отдельно:

```tsx
// Плохо:
const [items, setItems] = useState([...]);
const [filteredItems, setFilteredItems] = useState([...]);  // дублирование!

// Хорошо:
const [items, setItems] = useState([...]);
const [filter, setFilter] = useState('');

const filteredItems = items.filter(item =>
  item.name.toLowerCase().includes(filter.toLowerCase())
);
// filteredItems пересчитывается при каждом рендере — это ок
```

## Практический пример — Корзина покупок

```tsx
type Product = {
  id: number;
  name: string;
  price: number;
};

type CartItem = Product & { quantity: number };

function ShopScreen() {
  const products: Product[] = [
    { id: 1, name: 'Яблоко', price: 15 },
    { id: 2, name: 'Банан', price: 10 },
    { id: 3, name: 'Вишня', price: 50 },
  ];

  const [cart, setCart] = useState<CartItem[]>([]);

  const addToCart = (product: Product) => {
    const existing = cart.find(item => item.id === product.id);
    if (existing) {
      setCart(cart.map(item =>
        item.id === product.id
          ? { ...item, quantity: item.quantity + 1 }
          : item
      ));
    } else {
      setCart([...cart, { ...product, quantity: 1 }]);
    }
  };

  const total = cart.reduce((sum, item) => sum + item.price * item.quantity, 0);

  return (
    <View style={{ flex: 1, padding: 16 }}>
      <Text style={{ fontSize: 20, fontWeight: 'bold' }}>Магазин</Text>

      {products.map(product => (
        <View key={product.id} style={{ flexDirection: 'row', justifyContent: 'space-between', paddingVertical: 8 }}>
          <Text>{product.name} — {product.price}₽</Text>
          <Pressable onPress={() => addToCart(product)} style={{ backgroundColor: '#007AFF', padding: 8, borderRadius: 6 }}>
            <Text style={{ color: '#fff' }}>В корзину</Text>
          </Pressable>
        </View>
      ))}

      <Text style={{ fontSize: 18, fontWeight: 'bold', marginTop: 24 }}>
        Корзина: {total}₽ ({cart.reduce((sum, i) => sum + i.quantity, 0)} товаров)
      </Text>
    </View>
  );
}
```

---

## Практика

### Задание 4.1 — Форма регистрации

Создай экран `app/register.tsx` с управляемой формой:
- Поля: Имя, Email, Пароль, Подтверждение пароля
- Кнопка "Зарегистрироваться" — активна только если все поля заполнены и пароли совпадают
- Под кнопкой показывай текст ошибки если пароли не совпадают
- При нажатии на кнопку (если всё верно) — покажи `Alert.alert('Успех', 'Вы зарегистрированы!')`

### Задание 4.2 — Список задач

Создай приложение TodoList:
- Поле ввода + кнопка "Добавить"
- Список задач с возможностью отметить как выполненную (strikethrough текст)
- Кнопка "Удалить" у каждой задачи
- Счётчик: "Выполнено X из Y"

---

## Дополнительно (попробуй сам)

**Задача:** Добавь к TodoList фильтрацию без дополнительного state:

- Кнопки: "Все", "Активные", "Выполненные"
- Хранить только `filter: 'all' | 'active' | 'completed'`
- Отфильтрованный список — производный (вычисляй при рендере)

---

## Итог темы

- **Props** — данные снаружи, неизменяемы, задают поведение компонента
- **State** — данные внутри, при изменении перерисовывает компонент
- `useState(initial)` → `[value, setValue]`
- Не мутируй state напрямую — создавай новый объект/массив
- Если несколько компонентов нужен один state — подними его в родителя
- Производный state — вычисляй при рендере, не храни отдельно

**Следующая тема:** [05-hooks.md](./05-hooks.md)
