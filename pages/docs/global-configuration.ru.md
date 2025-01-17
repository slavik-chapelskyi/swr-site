# Глобальная конфигурация

Контекст `SWRConfig` может предоставить глобальные конфигурации ([опции](/docs/api#опции)) для всех SWR хуков.

```jsx
<SWRConfig value={options}>
  <Component/>
</SWRConfig>
```

В этом примере все SWR хуки будут использовать один и тот же fetcher, предоставленный для загрузки данных JSON, и по умолчанию обновляться каждые 3 секунды:

```jsx
import useSWR, { SWRConfig } from 'swr'

function Dashboard () {
  const { data: events } = useSWR('/api/events')
  const { data: projects } = useSWR('/api/projects')
  const { data: user } = useSWR('/api/user', { refreshInterval: 0 }) // переопределение

  // ...
}

function App () {
  return (
    <SWRConfig
      value={{
        refreshInterval: 3000,
        fetcher: (resource, init) => fetch(resource, init).then(res => res.json())
      }}
    >
      <Dashboard />
    </SWRConfig>
  )
}
```

## Вложение конфигураций

`SWRConfig` объединяет конфигурацию из родительского контекста. Он может принимать либо объект, либо функциональную конфигурацию. Функциональная получает в качестве аргумента родительскую конфигурацию и возвращает новую конфигурацию, которую вы можете настроить самостоятельно.

### Пример объектной конфигурации

```jsx
import { SWRConfig, useSWRConfig } from 'swr'

function App() {
  return (
    <SWRConfig
      value={{
        dedupingInterval: 100,
        refreshInterval: 100,
        fallback: { a: 1, b: 1 },
      }}
    >
      <SWRConfig
        value={{
          dedupingInterval: 200, // переопределит родительское значение, поскольку значение является примитивным
          fallback: { a: 2, c: 2 }, // будет сливаться с родительским значением, поскольку значение является объединяемым объектом
        }}
      >
        <Page />
      </SWRConfig>
    </SWRConfig>
  )
}

function Page() {
  const config = useSWRConfig()
  // {
  //   dedupingInterval: 200,
  //   refreshInterval: 100,
  //   fallback: { a: 2,  b: 1, c: 2 },
  // }
}
```

### Пример функциональной конфигурации

```jsx
import { SWRConfig, useSWRConfig } from 'swr'

function App() {
  return (
    <SWRConfig
      value={{
        dedupingInterval: 100,
        refreshInterval: 100,
        fallback: { a: 1, b: 1 },
      }}
    >
      <SWRConfig
        value={parent => ({
          dedupingInterval: parent.dedupingInterval * 5,
          fallback: { a: 2, c: 2 },
        })}
      >
        <Page />
      </SWRConfig>
    </SWRConfig>
  )
}

function Page() {
  const config = useSWRConfig()
  // {
  //   dedupingInterval: 500,
  //   fallback: { a: 2, c: 2 },
  // }
}
```

## Дополнительные API

### Провайдер кеша

Помимо всех перечисленных [опций](/docs/api#опции), `SWRConfig` также принимает опциональную функцию `provider`.
Пожалуйста, обратитесь к разделу [Кеш](/docs/advanced/cache) для более подробной информации.

```jsx
<SWRConfig value={{ provider: () => new Map() }}>
  <Dashboard />
</SWRConfig>
```

### Доступ к глобальным конфигурациям

Вы можете использовать ловушку `useSWRConfig` для получения глобальных конфигураций,
а также [`mutate`](/docs/mutation) и [`cache`](/docs/advanced/cache):

```jsx
import { useSWRConfig } from 'swr'

function Component () {
  const { refreshInterval, mutate, cache, ...restConfig } = useSWRConfig()

  // ...
}
```

Вложенные конфигурации будут расширены. Если не используется `<SWRConfig>`, вернётся значение по умолчанию.
