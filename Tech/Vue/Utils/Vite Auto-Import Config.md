##  Суть
Настройка Vite, которая эмулирует поведение Nuxt в обычном Vue 3 проекте.
Автоматически импортирует:
1. **Core API:** `vue`, `vue-router`, `pinia` (не нужно писать `import { ref } ...`).
2. **UI Kit:** Компоненты из слоя `01.kit`.
3. **API Services:** Сервисный слой.

Генерирует `.d.ts` файлы, чтобы TypeScript и VS Code понимали эту "магию".

##  Код (vite.config.ts)
**Необходимые пакеты:**
`npm i -D unplugin-auto-import unplugin-vue-components`
```ts
import { fileURLToPath, URL } from 'node:url'
import { defineConfig } from 'vite'

// Плагины авто-импорта
import autoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'

export default defineConfig({
  plugins: [
    // 1. Авто-импорт функций (ref, computed, useRoute, apiService)
    autoImport({
      imports: [
        'vue',
        'vue-router',
        'pinia',
        // Можно добавить '@vueuse/core' если используется
      ],
      // Кастомные директории с логикой
      dirs: [
        './src/shared/services/api', // API слой
        './src/components/01.kit/*', // Утилиты UI кита (если там есть .ts файлы)
      ],
      // Куда сохранять файл с типами
      dts: './src/types/dts/auto-imports.d.ts',
    }),

    // 2. Авто-импорт компонентов (Button, Input)
    Components({
      // Откуда брать компоненты без явного импорта
      dirs: ['src/components/01.kit'],
      
      // Ищем .ts файлы (если используется Barrel Pattern index.ts)
      // или добавь 'vue', если компоненты лежат просто файлами
      extensions: ['ts', 'vue'], 
      
      deep: true, // Искать во вложенных папках
      dts: 'src/types/dts/components.d.ts',
      
      // Исключения, чтобы не сканировать лишнее
      exclude: [
        /[\\/]node_modules[\\/]/,
        /[\\/]\.git[\\/]/,
        /[\\/]models[\\/]/, // Исключаем модели данных
      ],
    }),
  ],
})
```
##  Плюсы
- **Чистота кода:** Файлы становятся короче на 10-15 строк импортов.
- **Скорость:** Не тратишь время на написание `import { ref } from 'vue'`.
- **Единообразие:** Весь UI Kit доступен глобально, что подталкивает использовать именно его, а не писать стили с нуля.

##  Минусы
- **"Магия":** Новичок может не понять, откуда берется переменная, если не знает про плагины.
- **Конфликты имен:** Если создать переменную `ref` локально, она перекроет глобальный импорт (хотя это скорее плюс).
- **Tooling:** Иногда нужно перезапустить Volar (VS Code Server), чтобы он подхватил новые компоненты.

##  Когда использовать
- В любых **Vue 3 (Vite)** проектах, если это не Nuxt.
- Особенно полезно при архитектуре FSD/VSA, чтобы сделать слой `Shared/UI` (Kit) глобально доступным.