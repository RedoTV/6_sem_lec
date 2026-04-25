# Запуск Activity и передача данных между ними

## Способы запуска Activity

### 1. Явный запуск

```kotlin
val intent = Intent(this, SecondActivity::class.java)
startActivity(intent)
```

### 2. С передачей данных

```kotlin
val intent = Intent(this, SecondActivity::class.java).apply {
    putExtra("username", "Иван")
    putExtra("age", 25)
    putExtra("scores", arrayListOf(90, 85, 95))
}
startActivity(intent)
```

### 3. С ожиданием результата

```kotlin
val launcher = registerForActivityResult(
    ActivityResultContracts.StartActivityForResult()
) { result ->
    when (result.resultCode) {
        RESULT_OK -> {
            val data = result.data?.getStringExtra("result_key")
        }
        RESULT_CANCELED -> {
            // Пользователь отменил
        }
    }
}

// Запуск
val intent = Intent(this, SecondActivity::class.java)
launcher.launch(intent)
```

## Чтение данных в целевой Activity

```kotlin
class SecondActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val username = intent.getStringExtra("username") ?: ""
        val age = intent.getIntExtra("age", 0)
        val scores = intent.getStringArrayListExtra("scores") ?: arrayListOf()

        setContent {
            Text("Привет, $username! Возраст: $age")
        }
    }
}
```

## Возврат результата

```kotlin
// В SecondActivity
button.setOnClickListener {
    val resultIntent = Intent().apply {
        putExtra("result_key", "Результат обработки")
    }
    setResult(RESULT_OK, resultIntent)
    finish() // Закрыть Activity
}

// Или просто отменить
setResult(RESULT_CANCELED)
finish()
```

## Передача сложных объектов

### Parcelable (рекомендуется)

```kotlin
@Parcelize
data class User(
    val id: Int,
    val name: String,
    val email: String
) : Parcelable

// Отправка
val intent = Intent(this, SecondActivity::class.java).apply {
    putExtra("user", User(1, "Иван", "ivan@mail.ru"))
}
startActivity(intent)

// Получение
val user = intent.getParcelableExtra<User>("user")
```

### Serializable

```kotlin
data class User(
    val id: Int,
    val name: String
) : Serializable

// Отправка
intent.putExtra("user", User(1, "Иван"))

// Получение
val user = intent.getSerializableExtra("user") as? User
```

## Передача данных через Bundle

```kotlin
val bundle = Bundle().apply {
    putString("username", "Иван")
    putInt("age", 25)
    putParcelable("user", User(1, "Иван", "ivan@mail.ru"))
}

val intent = Intent(this, SecondActivity::class.java).apply {
    putExtras(bundle)
}
startActivity(intent)

// Получение
val bundle = intent.extras
val username = bundle?.getString("username")
```

## Передача данных во Fragment

```kotlin
val fragment = MyFragment().apply {
    arguments = Bundle().apply {
        putString("key", "value")
    }
}

// Во Fragment
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    val value = arguments?.getString("key")
}
```

## Возврат на предыдущую Activity

```kotlin
// Просто закрыть текущую
finish()

// Закрыть с результатом
val intent = Intent().apply { putExtra("key", "value") }
setResult(RESULT_OK, intent)
finish()

// Вернуться к конкретной Activity (очистить стек)
val intent = Intent(this, MainActivity::class.java).apply {
    flags = Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_SINGLE_TOP
}
startActivity(intent)
finish()
```

## Важные моменты

- `putExtra` перегружен для всех примитивных типов, String, Parcelable, Serializable
- Всегда предоставляйте значение по умолчанию при чтении: `getIntExtra("key", 0)`, `getStringExtra("key") ?: ""`
- Размер данных в Intent ограничен (~1 МБ). Для больших данных используйте БД, файлы или ViewModel
- `Parcelable` быстрее `Serializable` (рекомендуется Android)
- `registerForActivityResult` нужно вызывать **до** `onCreate` завершения (до `setContentView` / `setContent`), обычно на уровне поля класса
