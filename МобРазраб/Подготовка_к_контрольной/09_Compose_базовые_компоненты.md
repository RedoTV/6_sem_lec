# Compose: Базовые компоненты (Text, TextField, Button, Image)

Jetpack Compose — декларативный UI-фреймворк. UI описывается Kotlin-функциями с аннотацией `@Composable`.

## Общая структура Compose Activity

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyAppTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    Greeting("Android")
                }
            }
        }
    }
}

@Composable
fun Greeting(name: String) {
    Text(text = "Hello $name!")
}
```

---

## Text

Отображение текста.

```kotlin
// Простой текст
Text("Привет, мир!")

// Стилизация
Text(
    text = "Заголовок",
    color = Color.Red,
    fontSize = 24.sp,
    fontWeight = FontWeight.Bold,
    fontStyle = FontStyle.Italic,
    fontFamily = FontFamily.Serif,
    letterSpacing = 2.sp,
    textAlign = TextAlign.Center,
    maxLines = 2,
    overflow = TextOverflow.Ellipsis,
    lineHeight = 28.sp,
    textDecoration = TextDecoration.Underline
)

// С разными стилями в одном тексте
Text(
    buildAnnotatedString {
        append("Обычный текст, ")
        withStyle(SpanStyle(fontWeight = FontWeight.Bold, color = Color.Red)) {
            append("жирный красный")
        }
        append(", и снова обычный")
    }
)

// Выбираемый текст
SelectionContainer {
    Text("Этот текст можно выделить и скопировать")
}
```

## TextField / OutlinedTextField

Поле ввода текста.

```kotlin
// Состояние для хранения текста
var text by remember { mutableStateOf("") }

// Basic TextField
TextField(
    value = text,
    onValueChange = { text = it },
    label = { Text("Введите имя") },
    placeholder = { Text("Подсказка") },
    singleLine = true,
    leadingIcon = { Icon(Icons.Default.Search, "Поиск") },
    trailingIcon = { Icon(Icons.Default.Clear, "Очистить") },
    isError = text.length > 20,
    supportingText = { Text("${text.length}/20") },
    keyboardOptions = KeyboardOptions(
        keyboardType = KeyboardType.Email,
        imeAction = ImeAction.Done
    ),
    visualTransformation = PasswordVisualTransformation(),
    enabled = true,
    readOnly = false,
    colors = TextFieldDefaults.colors(
        focusedContainerColor = Color.White,
        unfocusedContainerColor = Color.LightGray
    )
)

// OutlinedTextField (с обводкой)
OutlinedTextField(
    value = text,
    onValueChange = { text = it },
    label = { Text("Email") },
    modifier = Modifier.fillMaxWidth()
)

// Многострочное поле
var multiText by remember { mutableStateOf("") }
TextField(
    value = multiText,
    onValueChange = { multiText = it },
    label = { Text("Описание") },
    minLines = 3,
    maxLines = 5
)
```

## Button

Кнопка.

```kotlin
// Обычная кнопка
Button(
    onClick = { /* действие */ },
    enabled = true,
    colors = ButtonDefaults.buttonColors(
        containerColor = Color.Blue,
        contentColor = Color.White,
        disabledContainerColor = Color.Gray
    ),
    shape = RoundedCornerShape(12.dp)
) {
    Text("Нажми меня")
}

// Кнопка с иконкой
Button(onClick = { }) {
    Icon(Icons.Default.Add, contentDescription = "Добавить")
    Spacer(modifier = Modifier.width(8.dp))
    Text("Добавить")
}

// OutlinedButton (с обводкой)
OutlinedButton(onClick = { }) {
    Text("Outlined")
}

// TextButton (только текст)
TextButton(onClick = { }) {
    Text("Text Button")
}

// IconButton (только иконка)
IconButton(onClick = { }) {
    Icon(Icons.Default.Favorite, contentDescription = "Избранное")
}

// FilledTonalButton
FilledTonalButton(onClick = { }) {
    Text("Tonal")
}

// ElevatedButton
ElevatedButton(onClick = { }) {
    Text("Elevated")
}

// FloatingActionButton
FloatingActionButton(
    onClick = { },
    containerColor = MaterialTheme.colorScheme.primary
) {
    Icon(Icons.Default.Add, contentDescription = "Добавить")
}
```

## Image

Отображение изображений.

### Из ресурсов

```kotlin
Image(
    painter = painterResource(R.drawable.my_image),
    contentDescription = "Описание изображения",
    modifier = Modifier.size(200.dp),
    contentScale = ContentScale.Crop,
    alignment = Alignment.Center,
    alpha = 0.8f
)
```

### С обрезкой (круг, скруглённые углы)

```kotlin
Image(
    painter = painterResource(R.drawable.avatar),
    contentDescription = "Аватар",
    modifier = Modifier
        .size(100.dp)
        .clip(CircleShape),
    contentScale = ContentScale.Crop
)

Image(
    painter = painterResource(R.drawable.photo),
    contentDescription = "Фото",
    modifier = Modifier
        .size(200.dp)
        .clip(RoundedCornerShape(16.dp)),
    contentScale = ContentScale.Crop
)
```

### Из URL (нужна библиотека Coil)

```kotlin
// Зависимость: implementation("io.coil-kt:coil-compose:2.5.0")
AsyncImage(
    model = "https://example.com/image.jpg",
    contentDescription = "Из интернета",
    modifier = Modifier.size(200.dp),
    contentScale = ContentScale.Crop,
    placeholder = painterResource(R.drawable.placeholder),
    error = painterResource(R.drawable.error_image)
)
```

### Иконки из Material Icons

```kotlin
Icon(
    imageVector = Icons.Default.Home,
    contentDescription = "Главная",
    tint = Color.Blue,
    modifier = Modifier.size(48.dp)
)

// Доступные: Icons.Default.Home, .Search, .Settings, .Add, .Delete,
// .Favorite, .ArrowBack, .Menu, .Close, .Edit, .Person, .Email, .Phone...
```

## Общие принципы Compose

- **Composable-функция** — функция с `@Composable`, описывает элемент UI
- Компоненты **не возвращают** значение, они _испускают_ UI
- `modifier` — всегда первый именованный параметр (по конвенции)
- Состояние управляется через `remember` и `mutableStateOf`
- `Column`, `Row`, `Box` — контейнеры для компоновки
