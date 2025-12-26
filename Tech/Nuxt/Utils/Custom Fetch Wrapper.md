## Суть
Обертка над нативным `useFetch` в Nuxt.
Цель: упростить вызовы API, автоматически подставлять `BaseURL` и токен авторизации из кук, а также типизировать ответ.
## Код дададад
Файл: `app/shared/api/fetch.ts` (или `composables/useApi.ts`)

```ts
// Nuxt auto-imports: useFetch, useCookie, createError, import.meta.env

async function fetch<Res>(url: string, options?: any, headers?: any) {
  try {
    // 1. Сборка URL
    const reqUrl = import.meta.env.VITE_API_URL + url

    // 2. Авто-инъекция токена
    // useCookie работает и на сервере, и на клиенте
    const token = useCookie('access') 
    const customHeaders = { 
      access: token.value, 
      ...headers 
    }

    // 3. Вызов Nuxt Fetch
    const { data, error, status } = await useFetch(reqUrl, { 
      ...options, 
      headers: customHeaders 
    })

    const result = data.value as Res

    // 4. Обработка ошибок
    if (status.value !== 'success') {
      throw createError({
        statusCode: 500,
        statusMessage: reqUrl,
        message: error?.value?.message || 'Server Internal Error',
      })
    }
    return result
  }
  catch (err) {
    return Promise.reject(err)
  }
}

// --- CRUD Helpers ---

export function getReq<Res>(url: string, params?: any, headers?: any) {
  return fetch<Res>(url, { method: 'get', params }, headers)
}

export function postReq<Res>(url: string, params?: any, headers?: any) {
  return fetch<Res>(url, { method: 'post', body: params }, headers)
}

export function putReq<Res>(url: string, params?: any, headers?: any) {
  return fetch<Res>(url, { method: 'put', body: params }, headers)
}

export function delReq<Res>(url: string, params?: any, headers?: any) {
  return fetch<Res>(url, { method: 'delete', params }, headers)
}
```

##  Плюсы текущей версии
- **Быстрый старт:** Меньше бойлерплейта в компонентах.
- **Типизация:** `<Res>` позволяет явно указать, что вернет бэкенд.
- **SSR Support:** `useCookie` корректно прокидывает токен при серверном рендеринге.

##  Архитектурные риски (To Do)
1.  **useFetch vs $fetch:**
    - `useFetch` предназначен для инициализации данных при загрузке страницы (setup).
    - Для действий пользователя (отправка формы, клик кнопки) правильнее использовать `$fetch` напрямую, так как там не нужна реактивность.
    - *План:* Сделать разделение на `useApi` (для GET при загрузке) и `apiClient` (для POST/PUT действий).
2.  **Config:**
    - Вместо `import.meta.env` лучше использовать `useRuntimeConfig().public.apiUrl`, чтобы можно было менять URL через environment variables в Docker без пересборки.
3.  **Локализация:**
    - Хардкод сообщения об ошибке ('服务器内部错误') нужно заменить на i18n ключ.

##  Когда использовать
- В текущем виде — для быстрого прототипирования.
- В будущем — отрефакторить под использование `$fetch` внутри `fetch` функции, чтобы избежать overhead от создания реактивных объектов `useFetch` там, где они не нужны.

### На будущее
`useFetch` возвращает `{ data: Ref<T> }`. Тебе приходится писать `.value`, чтобы достать данные.
`$fetch` возвращает просто `Promise<T>` (сами данные).
