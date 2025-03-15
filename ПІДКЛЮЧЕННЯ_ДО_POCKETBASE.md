# 🔌 Як підключитися до PocketBase

Привіт! Тут ти дізнаєшся, як підключити твій Next.js додаток до бази даних PocketBase. Це дуже просто! 😊

## 📦 Що таке PocketBase?

PocketBase - це як чарівна скринька, яка зберігає всі дані твого додатку:
- Завдання, які ти створюєш
- Інформацію про користувачів
- Налаштування додатку

## 🔧 Крок 1: Створення файлу для підключення

Спочатку створимо файл, який буде відповідати за підключення до PocketBase:

1. Відкрий свій проект у Visual Studio Code
2. Створи нову папку `lib` в корені проекту
3. Створи новий файл `lib/pocketbase.ts`
4. Скопіюй цей код у файл:

```typescript
// Імпортуємо PocketBase з бібліотеки
import PocketBase from 'pocketbase';

// Створюємо змінну для URL нашої бази даних
// Це адреса, за якою працює PocketBase на твоєму комп'ютері
const POCKETBASE_URL = 'http://127.0.0.1:8090';

// Створюємо клієнт PocketBase
// Це як пульт керування для нашої бази даних
const pb = new PocketBase(POCKETBASE_URL);

// Експортуємо клієнт, щоб інші файли могли його використовувати
export default pb;
```

## 🚀 Крок 2: Використання PocketBase в компонентах

Тепер ми можемо використовувати PocketBase в наших компонентах. Ось приклад, як отримати список завдань:

```typescript
// Створи файл app/todos/page.tsx
import pb from '@/lib/pocketbase';
import { useEffect, useState } from 'react';

// Тип для нашого завдання
type Todo = {
  id: string;
  title: string;
  completed: boolean;
};

// Компонент для сторінки завдань
export default function TodosPage() {
  // Створюємо стан для зберігання списку завдань
  const [todos, setTodos] = useState<Todo[]>([]);
  // Створюємо стан для відстеження завантаження
  const [loading, setLoading] = useState(true);

  // Ця функція виконується один раз, коли компонент завантажується
  useEffect(() => {
    // Асинхронна функція для отримання завдань
    async function fetchTodos() {
      try {
        // Отримуємо список завдань з PocketBase
        const records = await pb.collection('todos').getList(1, 50);
        // Зберігаємо завдання в стані
        setTodos(records.items as unknown as Todo[]);
      } catch (error) {
        // Якщо сталася помилка, показуємо її в консолі
        console.error('Помилка при отриманні завдань:', error);
      } finally {
        // В будь-якому випадку, завантаження завершено
        setLoading(false);
      }
    }

    // Викликаємо функцію для отримання завдань
    fetchTodos();
  }, []);

  // Якщо дані ще завантажуються, показуємо повідомлення
  if (loading) {
    return <div>Завантаження завдань...</div>;
  }

  return (
    <div>
      <h1>Мої завдання</h1>
      
      {/* Показуємо список завдань */}
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>
            <input 
              type="checkbox" 
              checked={todo.completed} 
              readOnly 
            />
            <span>{todo.title}</span>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## 🎮 Крок 3: Додавання нового завдання

Ось як додати нове завдання до бази даних:

```typescript
// Компонент форми для додавання завдання
import { useState } from 'react';
import pb from '@/lib/pocketbase';

export default function TodoForm() {
  // Стан для зберігання тексту завдання
  const [title, setTitle] = useState('');
  // Стан для відстеження процесу додавання
  const [adding, setAdding] = useState(false);

  // Функція для обробки відправки форми
  async function handleSubmit(e: React.FormEvent) {
    // Запобігаємо стандартній поведінці форми (перезавантаження сторінки)
    e.preventDefault();
    
    // Перевіряємо, чи не порожній текст завдання
    if (!title.trim()) return;
    
    // Позначаємо, що почався процес додавання
    setAdding(true);
    
    try {
      // Створюємо нове завдання в PocketBase
      await pb.collection('todos').create({
        title: title,
        completed: false,
        // Тут потрібно додати ID користувача, коли ми додамо авторизацію
        user: 'КОРИСТУВАЧ_ID'
      });
      
      // Очищаємо поле вводу після успішного додавання
      setTitle('');
      
      // Тут можна додати код для оновлення списку завдань
      // або показати повідомлення про успіх
      
    } catch (error) {
      // Якщо сталася помилка, показуємо її в консолі
      console.error('Помилка при додаванні завдання:', error);
      // Тут можна показати повідомлення про помилку користувачу
    } finally {
      // В будь-якому випадку, процес додавання завершено
      setAdding(false);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Що потрібно зробити?"
        disabled={adding}
      />
      <button type="submit" disabled={adding || !title.trim()}>
        {adding ? 'Додаємо...' : 'Додати'}
      </button>
    </form>
  );
}
```

## 🔄 Крок 4: Оновлення та видалення завдань

Ось як оновити статус завдання або видалити його:

```typescript
// Функція для позначення завдання як виконаного/невиконаного
async function toggleTodoStatus(id: string, completed: boolean) {
  try {
    // Оновлюємо завдання в PocketBase
    await pb.collection('todos').update(id, {
      completed: !completed
    });
    
    // Тут можна додати код для оновлення списку завдань
    // або показати повідомлення про успіх
    
  } catch (error) {
    console.error('Помилка при оновленні завдання:', error);
  }
}

// Функція для видалення завдання
async function deleteTodo(id: string) {
  try {
    // Видаляємо завдання з PocketBase
    await pb.collection('todos').delete(id);
    
    // Тут можна додати код для оновлення списку завдань
    // або показати повідомлення про успіх
    
  } catch (error) {
    console.error('Помилка при видаленні завдання:', error);
  }
}
```

## 🔐 Крок 5: Авторизація користувачів

Щоб користувачі могли входити в свій акаунт, ми можемо використати вбудовану авторизацію PocketBase:

```typescript
// Функція для реєстрації нового користувача
async function registerUser(email: string, password: string, passwordConfirm: string) {
  try {
    // Створюємо нового користувача в PocketBase
    const user = await pb.collection('users').create({
      email,
      password,
      passwordConfirm
    });
    
    // Автоматично входимо після реєстрації
    await pb.collection('users').authWithPassword(email, password);
    
    return user;
  } catch (error) {
    console.error('Помилка при реєстрації:', error);
    throw error;
  }
}

// Функція для входу користувача
async function loginUser(email: string, password: string) {
  try {
    // Входимо в акаунт через PocketBase
    const user = await pb.collection('users').authWithPassword(email, password);
    return user;
  } catch (error) {
    console.error('Помилка при вході:', error);
    throw error;
  }
}

// Функція для виходу з акаунту
function logoutUser() {
  // Виходимо з акаунту
  pb.authStore.clear();
}

// Перевірка, чи користувач увійшов в акаунт
function isUserLoggedIn() {
  return pb.authStore.isValid;
}

// Отримання інформації про поточного користувача
function getCurrentUser() {
  return pb.authStore.model;
}
```

## 🎉 Готово!

Тепер ти знаєш, як підключити твій Next.js додаток до PocketBase і виконувати основні операції:
- Отримувати список завдань
- Додавати нові завдання
- Оновлювати статус завдань
- Видаляти завдання
- Реєструвати користувачів і входити в акаунт

Використовуй ці приклади як основу для твого Todo List додатку і експериментуй з ними! 🚀

## 🌟 Додаткові можливості

PocketBase має багато інших можливостей, які ти можеш використати в своєму додатку:
- Завантаження файлів (наприклад, аватарок користувачів)
- Реальночасові оновлення (щоб бачити зміни одразу)
- Фільтрація і сортування даних
- Пагінація (розділення великих списків на сторінки)

Коли ти освоїш основи, спробуй додати ці можливості до свого додатку! 😊 