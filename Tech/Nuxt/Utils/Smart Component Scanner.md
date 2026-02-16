## Суть
Стандартный авто-импорт Nuxt сканирует все `.vue` файлы и генерирует имена на основе вложенности папок (напр. `<FeaturesTaskUiCard>`).
Этот скрипт меняет поведение: он ищет файлы `index.ts` внутри папок и регистрирует их как компоненты. Это позволяет "срезать" длинные префиксы и инкапсулировать сложные компоненты, экспортируя только то, что нужно.

## Код (nuxt.config.ts)
```ts
import { readdirSync } from 'node:fs'
import { resolve, basename } from 'node:path'

/**
 * Рекурсивно ищет папки, содержащие index.ts, 
 * чтобы зарегистрировать их как изолированные компоненты.
 */
export function findComponentDirs(startPath: string): any[] {
  const basePath = resolve(__dirname, startPath)
  const results: any[] = []

  function recurse(currentPath: string) {
    const entries = readdirSync(currentPath, { withFileTypes: true })
    
    // Ищем маркер "публичного API" компонента
    const hasIndex = entries.some(entry => entry.isFile() && entry.name === 'index.ts')

    if (hasIndex) {
      const folderName = basename(currentPath)
      
      results.push({
        path: currentPath,
        // Опционально: можно задать prefix, если нужно пространство имен
        prefix: folderName, 
        
        // Важно: говорим Nuxt не генерировать префикс от пути к файлу
        pathPrefix: false, 
        
        // Nuxt будет искать экспорты именно в этом файле
        pattern: 'index.ts', 
        extensions: ['.ts', '.vue'] 
      })
    }

    // Идем глубже
    for (const entry of entries) {
      if (entry.isDirectory()) {
        recurse(resolve(currentPath, entry.name))
      }
    }
  }

  try {
    recurse(basePath)
  } catch (error) {
    console.warn(`Could not scan directory: ${basePath}`, error)
  }

  return results
}
```

## Интеграция (nuxt.config.ts)
```ts
import { findComponentDirs } from './nuxt.config.utils'

export default defineNuxtConfig({
  hooks: {
    'components:dirs': (dirs) => {
      // Сканируем UI-кит или виджеты
      const widgetDirs = findComponentDirs('./app/components/02.shared')
      
      // Добавляем найденное в конфиг Nuxt
      dirs.push(...widgetDirs)
    }
  }
})
```

## Пример структуры
Файловая система:
```text
app/components/02.shared/
  UserCard/
    ├── UserAvatar.vue  (Внутренний компонент, Nuxt его НЕ увидит глобально)
    ├── UserInfo.vue    (Тоже приватный)
    └── index.ts        (Public API)
```

Содержимое `index.ts`:
```ts
// Экспортируем собранный компонент
// В шаблонах он будет доступен просто как <UserCard />
export { default as UserCard } from './UserCardWrapper.vue'
// или так
export { default } from './UserCardWrapper.vue'
```

## Плюсы
- **Чистые имена:** `<UserCard />` вместо `<SharedUserCardWrapper>`.
- **Инкапсуляция:** Вспомогательные компоненты (`UserAvatar`) не засоряют глобальное пространство имен.
- **Контроль:** Ты явно решаешь, что доступно извне, а что — приватная кухня компонента.
## Минусы
- **Ручная работа:** Нужно создавать `index.ts` для каждого глобального компонента.
- **HMR:** Иногда при изменении экспортов в `index.ts` требуется перезагрузка Nuxt.

## Когда использовать
- Для **UI Kit** и **Shared Widgets**.
- Когда компонент состоит из 5-10 под-файлов, и ты не хочешь, чтобы они все "светились" в авто-импортах.