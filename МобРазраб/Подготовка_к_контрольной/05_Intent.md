# Intent (явные и неявные)

Intent — это объект-намерение, описывающий действие, которое нужно выполнить. Используется для:
- Запуска Activity
- Запуска Service
- Отправки BroadcastReceiver
- Взаимодействия между компонентами

## Типы Intent

### 1. Явные (Explicit) Intent

Указывается **конкретный класс** целевого компонента. Используется для навигации внутри приложения.

```kotlin
val intent = Intent(this, SecondActivity::class.java)
intent.putExtra("key", "value")
intent.putExtra("number", 42)
startActivity(intent)
```

Чтение данных в целевой Activity:
```kotlin
val value = intent.getStringExtra("key")
val number = intent.getIntExtra("number", 0)
```

### 2. Неявные (Implicit) Intent

Указывается **действие** (action), а система сама подбирает подходящее приложение. Используется для взаимодействия с другими приложениями.

```kotlin
// Открыть URL в браузере
val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://google.com"))
startActivity(intent)

// Позвонить
val intent = Intent(Intent.ACTION_DIAL, Uri.parse("tel:+79001234567"))
startActivity(intent)

// Отправить текст
val intent = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, "Текст для отправки")
}
startActivity(Intent.createChooser(intent, "Поделиться через"))
}

// Отправить email
val intent = Intent(Intent.ACTION_SENDTO).apply {
    data = Uri.parse("mailto:")
    putExtra(Intent.EXTRA_EMAIL, arrayOf("test@example.com"))
    putExtra(Intent.EXTRA_SUBJECT, "Тема")
    putExtra(Intent.EXTRA_TEXT, "Текст письма")
}
startActivity(intent)
```

## Основные атрибуты Intent

| Атрибут | Описание | Пример |
|---------|----------|--------|
| `action` | Действие для выполнения | `ACTION_VIEW`, `ACTION_SEND`, `ACTION_EDIT` |
| `data` | Данные (URI) | `Uri.parse("https://google.com")` |
| `type` | MIME-тип данных | `"text/plain"`, `"image/*"` |
| `extras` | Дополнительные данные | `putExtra("key", value)` |
| `category` | Категория компонента | `CATEGORY_LAUNCHER`, `CATEGORY_DEFAULT` |
| `flags` | Флаги поведения | `FLAG_ACTIVITY_NEW_TASK`, `FLAG_CLEAR_TOP` |

## Передача данных через Intent

### Базовые типы
```kotlin
intent.putExtra("string", "текст")
intent.putExtra("int", 42)
intent.putExtra("boolean", true)
intent.putExtra("double", 3.14)
intent.putExtra("float", 2.7f)
intent.putExtra("long", 100L)
```

### Передача объектов (Parcelable / Serializable)

```kotlin
// Serializable (проще, но медленнее)
data class User(val name: String, val age: Int) : Serializable

intent.putExtra("user", User("Иван", 25))
val user = intent.getSerializableExtra("user") as? User

// Parcelable (быстрее, рекомендуется)
@Parcelize
data class User(val name: String, val age: Int) : Parcelable

intent.putExtra("user", User("Иван", 25))
val user = intent.getParcelableExtra<User>("user")
```

### Передача массивов и списков
```kotlin
intent.putExtra("array", arrayOf("a", "b", "c"))
intent.putStringArrayListExtra("list", arrayListOf("a", "b", "c"))
```

## Получение результата от Activity (ActivityResult)

```kotlin
// Запускающая Activity
val launcher = registerForActivityResult(
    ActivityResultContracts.StartActivityForResult()
) { result ->
    if (result.resultCode == RESULT_OK) {
        val data = result.data?.getStringExtra("result")
    }
}

// Запуск
val intent = Intent(this, SecondActivity::class.java)
launcher.launch(intent)

// В SecondActivity — возврат результата
val resultIntent = Intent().apply {
    putExtra("result", "Данные от SecondActivity")
}
setResult(RESULT_OK, resultIntent)
finish()
```

### Часто используемые контракты

```kotlin
// Запрос разрешения
val permissionLauncher = registerForActivityResult(
    ActivityResultContracts.RequestPermission()
) { granted ->
    if (granted) { /* разрешение получено */ }
}

// Выбор изображения
val pickImage = registerForActivityResult(
    ActivityResultContracts.PickVisualMedia()
) { uri: Uri? ->
    // Работа с URI
}
```

## Intent Filter

Intent Filter объявляется в манифесте и указывает, какие Intent'ы может принимать компонент:

```xml
<activity android:name=".ShareActivity" android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
</activity>
```

## Intent Flags (часто используемые)

| Флаг | Описание |
|------|----------|
| `FLAG_ACTIVITY_NEW_TASK` | Запуск Activity в новой задаче |
| `FLAG_ACTIVITY_CLEAR_TOP` | Удалить все Activity над целевой в стеке |
| `FLAG_ACTIVITY_SINGLE_TOP` | Не создавать новый экземпляр, если Activity уже на вершине |
| `FLAG_ACTIVITY_NO_HISTORY` | Activity не сохраняется в стеке |

```kotlin
intent.flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
```

## Важные моменты

- Явные Intent — для навигации внутри приложения (указываем конкретный класс)
- Неявные Intent — для взаимодействия с другими приложениями (указываем действие)
- Всегда проверяйте наличие приложения для обработки неявного Intent:
```kotlin
val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://google.com"))
if (intent.resolveActivity(packageManager) != null) {
    startActivity(intent)
}
```
- Используйте `registerForActivityResult` вместо устаревшего `startActivityForResult`
- Для передачи объектов предпочтительнее `Parcelable` (быстрее `Serializable`)
