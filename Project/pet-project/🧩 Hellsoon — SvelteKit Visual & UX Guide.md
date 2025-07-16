
---

## 🌌 Общая концепция
Это визитка-разработка в мрачном, техно-оккультном стиле. Используется стек SvelteKit + Tailwind + REST API с NestJS.

---

## 🎨 Цветовая палитра (Voidborn UI)

| Назначение        | Цвет         | HEX      |
|-------------------|--------------|----------|
| Фон                | `#0d0d0d`    | Абсолютная тьма |
| Вторичный фон      | `#161616`    | Тёмные блоки |
| Основной акцент    | `#8f43ff`    | Глиф-сиреневый |
| Вторичный акцент   | `#00c3ff`    | Холодный неон |
| Ошибка / красный   | `#ff3c6a`    | Кровавая магия |
| Успех / зелёный    | `#00ffae`    | Призрачный неон |
| Текст основной     | `#e0e0e0`    | Светлый |
| Текст вторичный    | `#7f7f7f`    | Подписи |

---

## ✨ Анимации и визуальные фишки

- `transition:fade` + `scale` при появлении блоков
- `GlitchText` — SVG/clip-path/JS глитч-заголовки
- Неоновые свечения через `box-shadow` и Tailwind glow
- Кастомный курсор (SVG-глиф)
- Страница 404: «Свиток потерян в Пустоте»
- WebGL/Canvas живой фон (опционально)
- Терминальный ввод (мини CLI-интерфейс)

---

## 📂 Структура проекта (SvelteKit)

```
/src
  /routes
    /                 — Главная: портал в Архив
    /whoami           — Легенда о Hellsoon
    /artifacts        — Проекты (Артефакты)
    /codex            — Блог / Кодексы знаний
    /summon           — Контактная форма (Призыв)
    /admin            — Панель управления
  /lib
    /components       — Визуальные и логические блоки
    /stores           — Глобальное состояние
    /styles           — Глобальные стили и анимации
  /assets
    /glyphs.svg       — Глифы
    /backgrounds      — Темные текстуры / noise
```


---

## 🧱 Tailwind-конфиг (пример)

```js
// tailwind.config.js
theme: {
  extend: {
    colors: {
      void: '#0d0d0d',
      shadow: '#161616',
      glyph: '#8f43ff',
      ice: '#00c3ff',
      blood: '#ff3c6a',
      spirit: '#00ffae',
    },
    fontFamily: {
      gothic: ['UnifrakturCook', 'serif'],
      mono: ['Share Tech Mono', 'monospace'],
    },
  }
}
```

---

## 🔗 Названия и ссылки

| Название раздела   | URL            | Описание |
|--------------------|----------------|----------|
| 🏠 Главная          | `/`            | Вступление. Терминал |
| 👤 Кто есть Hellsoon | `/whoami`      | Биография в легенде |
| 🗂️ Артефакты         | `/artifacts`   | Проекты |
| 📖 Кодексы           | `/codex`       | Посты, статьи |
| 📡 Призвать          | `/summon`      | Контактная форма |
| 🧿 Тайный канал      | `/admin`       | Админка для управления |

---

## 🧩 Компоненты (Svelte)

- `TerminalIntro.svelte` — Ввод команды при входе
- `GlitchText.svelte` — Заголовки с эффектами
- `SummonForm.svelte` — Контактная форма в стиле ритуала
- `ArtifactCard.svelte` — Превью проектов
- `DarkLayout.svelte` — Базовый тёмный layout

---

## 📦 Шрифты и ресурсы

- [UnifrakturCook](https://fonts.google.com/specimen/UnifrakturCook) — готический
- [Share Tech Mono](https://fonts.google.com/specimen/Share+Tech+Mono) — техно
- [Three.js](https://threejs.org) / [Zdog](https://zzz.dog) — фоновые WebGL-эффекты

---

## ⚙️ Будущие улучшения

- Темная и светлая форма (день/ночь)
- Чат в виде демона/тени (AI mock)
- Markdown-блог из `/codex`
- Управление контентом через NestJS API
- Докеризация всего проекта