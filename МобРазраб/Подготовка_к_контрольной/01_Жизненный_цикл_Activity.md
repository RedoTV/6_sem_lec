# Жизненный цикл Activity

Activity — это один из фундаментальных компонентов Android-приложения. Каждое Activity проходит через определённые состояния в течение своего жизненного цикла.

## Состояния Activity

| Состояние | Описание |
|-----------|----------|
| **Created** | Activity создана, но ещё не видна пользователю |
| **Started** | Activity видна, но не в фокусе (пользователь не может с ней взаимодействовать) |
| **Resumed** | Activity на переднем плане, пользователь может с ней взаимодействовать |
| **Paused** | Activity частично скрыта другим Activity (например, диалогом). Не получает ввод |
| **Stopped** | Activity полностью скрыта, но сохраняет состояние в памяти |
| **Destroyed** | Activity уничтожена, ресурсы освобождены |

## Основные callback-методы

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Инициализация, setContentView, настройка Compose
        // Это единственный обязательный метод
    }

    override fun onStart() {
        super.onStart()
        // Activity становится видимой
    }

    override fun onResume() {
        super.onResume()
        // Activity в фокусе, пользователь взаимодействует
    }

    override fun onPause() {
        super.onPause()
        // Activity теряет фокус (частично видна)
        // Здесь нужно сохранять критичные данные
    }

    override fun onStop() {
        super.onStop()
        // Activity больше не видна
    }

    override fun onRestart() {
        super.onRestart()
        // Activity перезапускается после Stopped
    }

    override fun onDestroy() {
        super.onDestroy()
        // Activity уничтожается
        // Освобождение ресурсов, отписка от подписок
    }
}
```

## Порядок вызовов

```
Запуск приложения:
  onCreate → onStart → onResume

Пользователь нажал "Назад":
  onPause → onStop → onDestroy

Пользователь открыл другое Activity:
  onPause → onStop

Возврат к Activity из Stopped:
  onRestart → onStart → onResume

Пользователь нажал "Home":
  onPause → onStop

Возврат к Activity после Home:
  onRestart → onStart → onResume

Входящий звонок (overlay):
  onPause → (звонок завершён) → onResume
```

## onSaveInstanceState и onRestoreInstanceState

Используются для сохранения и восстановления данных при уничтожении Activity системой (например, при повороте экрана).

```kotlin
override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
    outState.putString("key", "value")
}

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    if (savedInstanceState != null) {
        val value = savedInstanceState.getString("key")
    }
}
```

## Важные моменты

- `onCreate()` — единственный **обязательный** callback
- `onPause()` — гарантированно вызывается перед тем, как Activity покидает передний план. Используйте для сохранения важных данных
- `onDestroy()` — **не гарантирован** к вызову (система может убить процесс). Не полагайтесь на него для критичных операций
- Конфигурационные изменения (поворот экрана, смена языка) уничтожают и пересоздают Activity
- В Jetpack Compose состояние сохраняется через `rememberSaveable`
