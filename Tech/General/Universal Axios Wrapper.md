## Суть
Абстракция над Axios, которая превращает хаос с ошибками в предсказуемую систему.
1. **Нормализация ошибок:** Преобразует любые ошибки (сеть, 404, 500) в единый интерфейс `ApiError`.
2. **Типизация:** Generic-методы `get<T>`, `post<T>` гарантируют, что мы знаем тип ответа.
3. **Изоляция:** UI-компоненты не импортируют `axios` напрямую, что позволяет легко менять настройки HTTP-клиента в одном месте.

## Код

Структура файлов (обычно лежит в `shared/services/api` или `shared/api`):

### 1. Конфигурация (`http.ts`)
Создание инстанса. Здесь подключаются interceptors (например, для добавления Bearer токена).

```ts
import axios from 'axios'

export const api = axios.create({
  // В Vite используем import.meta.env
  // В Nuxt лучше прокидывать через runtimeConfig, если нужен SSR
  baseURL: import.meta.env.VITE_API_BASE_URL, 
})

api.interceptors.request.use(
  (config) => config,
  (error) => Promise.reject(error),
)
```

### 2. Типы (`api.types.ts`)
Контракты для ошибок и конфигов.

```ts
import type { AxiosError, AxiosRequestConfig } from 'axios'

export interface BackendErrorDetail {
  message: string
  [key: string]: any
}

// Единый формат ошибки для всего приложения
export interface ApiError {
  message: string
  status?: number
  details?: BackendErrorDetail | any
  config?: RequestConfig
  raw: AxiosError
}

export type ErrorNotificationStrategy = 'none' | 'toast' | 'custom'

export interface NotificationOptions {
  strategy: ErrorNotificationStrategy
  message?: string
}

export interface RequestConfig extends AxiosRequestConfig {
  suppressErrorNotify?: NotificationOptions
}
```

### 3. Сервис (`api.services.ts`)
Основная логика нормализации и CRUD методы.

```ts
import type { AxiosError, Method } from 'axios'
import type { ApiError, BackendErrorDetail, RequestConfig } from './api.types'
import { api } from './http'

// Type Guard: проверка, пришла ли ошибка от бэкенда в ожидаемом формате
function isBackendError(data: any): data is BackendErrorDetail {
  return data && typeof data.message === 'string'
}

function normalizeError(error: AxiosError): ApiError {
  const apiError: ApiError = {
    message: 'Произошла неизвестная ошибка',
    status: error.response?.status,
    config: error.config as RequestConfig,
    raw: error,
  }

  if (error.response) {
    // Ошибка от сервера (4xx, 5xx)
    const errorData = error.response.data
    apiError.details = errorData
    
    if (isBackendError(errorData)) {
      apiError.message = errorData.message
    } else {
      apiError.message = error.message
    }
  } 
  else if (error.request) {
    // Запрос ушел, но ответа нет (Network Error)
    apiError.message = 'Сервер не отвечает. Проверьте подключение к сети.'
  } 
  else {
    // Ошибка настройки запроса
    apiError.message = error.message
  }

  return apiError
}

// Generic-обертка над запросом
async function request<T>(
  method: Method,
  url: string,
  config: RequestConfig = {},
  data?: any,
): Promise<T> {
  try {
    const response = await api.request<T>({
      method,
      url,
      data,
      ...config,
    })
    return response.data
  } catch (error) {
    throw normalizeError(error as AxiosError)
  }
}

// Public API
export const apiService = {
  get<T>(url: string, config?: RequestConfig): Promise<T> {
    return request<T>('GET', url, config)
  },

  post<T>(url: string, data?: any, config?: RequestConfig): Promise<T> {
    return request<T>('POST', url, config, data)
  },

  put<T>(url: string, data?: any, config?: RequestConfig): Promise<T> {
    return request<T>('PUT', url, config, data)
  },

  delete<T>(url: string, config?: RequestConfig): Promise<T> {
    return request<T>('DELETE', url, config)
  },
}
```

### 4. Barrel (`index.ts`)
```ts
export * from './api.services'
export * from './api.types'
```

## Плюсы
- **Единая точка правды:** Логика обработки ошибок находится в `normalizeError`, а не размазана по сотням `catch` блоков в компонентах.
- **Чистый код в компонентах:**
  ```ts
  // Было
  try { ... } catch (e) { alert(e.response?.data?.message || e.message || 'Error') }
  
  // Стало
  try { ... } catch (e) { alert(e.message) } // e гарантированно ApiError
  ```
- **Универсальность:** Работает везде, где есть JS (Vue 2/3, React, Nuxt, Node.js).

## Минусы / Nuxt Specific
- **Nuxt SSR:** В Nuxt 3/4 есть встроенный `useFetch` / `$fetch`, который лучше оптимизирован для SSR (уменьшает дублирование запросов, работает с `useAsyncData`).
- **Axios в Nuxt:** Если использовать этот враппер в Nuxt на стороне сервера (SSR), нужно внимательно следить за передачей Cookies и заголовков от клиента к серверу API, так как Axios сам это не сделает (в отличие от проксирующего `useFetch`).

## Когда использовать
- В SPA (Single Page Applications) на Vue 3 / Vite.
- При миграции легаси-проектов на новые рельсы.
- Если бэкенд отдает специфичные структуры ошибок, которые `useFetch` обрабатывает неудобно.