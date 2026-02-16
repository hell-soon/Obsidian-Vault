## Суть
Обертка над `socket.io-client` для Nuxt/Vue приложений.
Решает 3 главные проблемы:
1. **Мульти-неймспейсинг:** Позволяет держать открытыми несколько соединений одновременно (например, `/chat` и `/notifications`).
2. **Реактивность:** Состояние подключения (`isConnected`) реактивно, UI обновляется сам.
3. **Типобезопасность:** Использует Generics для валидации имен событий и передаваемых данных.

## Код
**Зависимости:** `npm i socket.io-client`
```ts
// app/composables/useWebSocket.ts (или shared/composables)

import type { Socket } from 'socket.io-client'
import type { DeepReadonly } from 'vue'
// Не забудь создать файл с типами! См. ниже.
import type { ClientToServerEvents, ServerToClientEvents } from '~/shared/types/socketEvents'
import { io } from 'socket.io-client'

interface WebSocketState {
  isConnected: boolean
}

// Типизированный сокет
type AppSocket = Socket<ServerToClientEvents, ClientToServerEvents>

export interface UseWebSocketReturn {
  connect: (namespace: string, auth?: { token: string } | null) => void
  disconnect: (namespace: string) => void
  on: <Ev extends keyof ServerToClientEvents & string>(
    namespace: string,
    event: Ev,
    callback: ServerToClientEvents[Ev],
  ) => () => void // Возвращает функцию отписки
  emit: <Ev extends keyof ClientToServerEvents & string>(
    namespace: string,
    event: Ev,
    ...args: Parameters<ClientToServerEvents[Ev]>
  ) => void
  sockets: DeepReadonly<Map<string, { instance: AppSocket, state: WebSocketState }>>
  getSocketState: (namespace: string) => WebSocketState | undefined
}

type SocketMap = Map<string, { instance: AppSocket, state: WebSocketState }>

// Singleton State (один на всё приложение)
const sockets = reactive<SocketMap>(new Map())

export function useWebSocket(): UseWebSocketReturn {
  const config = useRuntimeConfig() // Лучше брать URL отсюда, а не напрямую из process.env

  const connect = (namespace: string, auth: { token: string } | null = null) => {
    if (sockets.has(namespace)) {
      console.warn(`[WebSocket] Connection to "${namespace}" already exists.`)
      return
    }

    const WEBSOCKET_URL = config.public.wsUrl || import.meta.env.VITE_WS_URL
    const fullUrl = `${WEBSOCKET_URL}${namespace}`

    const socket = io(fullUrl, {
      ...(auth && { auth }),
      autoConnect: false,
      transports: ['websocket', 'polling'],
    })

    const state: WebSocketState = reactive({ isConnected: false })
    sockets.set(namespace, { instance: socket, state })

    // System Events
    socket.on('connect', () => {
      state.isConnected = true
    })
    socket.on('disconnect', () => {
      state.isConnected = false
    })
    socket.on('connect_error', (err) => {
      console.error(`[WebSocket] Error "${namespace}":`, err.message)
      state.isConnected = false
      sockets.delete(namespace)
    })

    socket.connect()
  }

  const disconnect = (namespace: string) => {
    const socketData = sockets.get(namespace)
    if (socketData) {
      socketData.instance.disconnect()
      sockets.delete(namespace)
    }
  }

  const on = <Ev extends keyof ServerToClientEvents & string>(
    namespace: string,
    event: Ev,
    callback: ServerToClientEvents[Ev],
  ) => {
    const socketData = sockets.get(namespace)
    if (!socketData) return () => {}
    
    socketData.instance.on(event, callback as any)
    
    // Возвращаем cleanup функцию для onUnmounted
    return () => socketData.instance.off(event, callback as any)
  }

  const emit = <Ev extends keyof ClientToServerEvents & string>(
    namespace: string,
    event: Ev,
    ...args: Parameters<ClientToServerEvents[Ev]>
  ) => {
    const socketData = sockets.get(namespace)
    if (socketData?.state.isConnected) {
      socketData.instance.emit(event, ...args)
    }
  }

  return {
    connect,
    disconnect,
    on,
    emit,
    sockets: readonly(sockets),
    getSocketState: (ns) => sockets.get(ns)?.state,
  }
}
```

## Типизация событий
Чтобы IntelliSense работал, нужно описать контракт.
Файл: `app/shared/types/socketEvents.ts`
```ts
export interface ServerToClientEvents {
  'chat:message': (msg: { id: number, text: string }) => void
  'notification:new': (data: { title: string }) => void
}

export interface ClientToServerEvents {
  'chat:send': (text: string) => void
  'auth:verify': (token: string) => void
}
```

## Пример использования (в компоненте)
```vue
<script setup lang="ts">
const { connect, disconnect, on, emit, getSocketState } = useWebSocket()
const chatState = computed(() => getSocketState('/chat'))

onMounted(() => {
  // 1. Подключение
  connect('/chat', { token: 'user-token' })

  // 2. Подписка (авто-отписка нужна при unmount)
  const unsub = on('/chat', 'chat:message', (msg) => {
    console.log('New message:', msg.text) // Типизировано!
  })
  
  // Clean up listener if needed manually
  // unsub()
})

// 3. Отправка
function sendMessage() {
  emit('/chat', 'chat:send', 'Hello world') // TS подскажет, если аргументы не те
}

// 4. Очистка соединения при уходе со страницы
onUnmounted(() => {
  disconnect('/chat')
})
</script>

<template>
  <div>
    Status: {{ chatState?.isConnected ? '🟢' : '🔴' }}
  </div>
</template>
```

## Плюсы
- **Глобальное состояние:** Если ты перейдешь на другую страницу и вернешься, соединение не разорвется (если не вызвать `disconnect`).
- **DX (Developer Experience):** TypeScript защищает от опечаток в названиях событий (`chat:messaage` подчеркнет красным).

## Минусы (Risks)
- **Memory Leaks:** Если забыть вызвать `disconnect()` или отписаться от событий через возвращаемую функцию, слушатели будут копиться.
- **SSR:** Socket.io не работает на сервере Nuxt. Весь код вызова (`connect`) нужно оборачивать в `onMounted` или проверку `if (import.meta.client)`.