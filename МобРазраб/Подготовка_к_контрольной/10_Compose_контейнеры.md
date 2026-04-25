# Compose: Контейнеры (Column, Row, Box)

Контейнеры (layout-компоненты) определяют расположение дочерних элементов.

---

## Column — вертикальное расположение

Располагает дочерние элементы **сверху вниз** (по вертикали).

```kotlin
Column(
    modifier = Modifier.fillMaxSize(),
    verticalArrangement = Arrangement.SpaceBetween,
    horizontalAlignment = Alignment.CenterHorizontally
) {
    Text("Первый")
    Text("Второй")
    Text("Третий")
}
```

### Параметры Column

| Параметр | Описание |
|----------|----------|
| `verticalArrangement` | Расположение по вертикали |
| `horizontalAlignment` | Выравнивание по горизонтали |
| `modifier` | Модификатор |

### verticalArrangement

```kotlin
Arrangement.Top           // Сверху (по умолчанию)
Arrangement.Bottom        // Снизу
Arrangement.Center        // По центру
Arrangement.SpaceBetween  // Равные промежутки между элементами
Arrangement.SpaceAround   // Промежутки + половина по краям
Arrangement.SpaceEvenly   // Равные промежутки везде
Arrangement.spacedBy(16.dp) // Фиксированный отступ между элементами
```

### horizontalAlignment

```kotlin
Alignment.Start          // По левому краю
Alignment.End            // По правому краю
Alignment.CenterHorizontally // По центру
```

### Пример: форма входа

```kotlin
Column(
    modifier = Modifier
        .fillMaxSize()
        .padding(24.dp),
    verticalArrangement = Arrangement.Center,
    horizontalAlignment = Alignment.CenterHorizontally
) {
    Text("Вход", fontSize = 28.sp, fontWeight = FontWeight.Bold)
    Spacer(modifier = Modifier.height(16.dp))

    var login by remember { mutableStateOf("") }
    var password by remember { mutableStateOf("") }

    OutlinedTextField(
        value = login,
        onValueChange = { login = it },
        label = { Text("Логин") },
        modifier = Modifier.fillMaxWidth()
    )
    Spacer(modifier = Modifier.height(8.dp))

    OutlinedTextField(
        value = password,
        onValueChange = { password = it },
        label = { Text("Пароль") },
        visualTransformation = PasswordVisualTransformation(),
        modifier = Modifier.fillMaxWidth()
    )
    Spacer(modifier = Modifier.height(24.dp))

    Button(onClick = { }, modifier = Modifier.fillMaxWidth()) {
        Text("Войти")
    }
}
```

---

## Row — горизонтальное расположение

Располагает дочерние элементы **слева направо** (по горизонтали).

```kotlin
Row(
    modifier = Modifier.fillMaxWidth(),
    horizontalArrangement = Arrangement.SpaceBetween,
    verticalAlignment = Alignment.CenterVertically
) {
    Text("Левый")
    Text("Центр")
    Text("Правый")
}
```

### Параметры Row

| Параметр | Описание |
|----------|----------|
| `horizontalArrangement` | Расположение по горизонтали |
| `verticalAlignment` | Выравнивание по вертикали |

### verticalAlignment

```kotlin
Alignment.Top            // По верхнему краю
Alignment.Bottom         // По нижнему краю
Alignment.CenterVertically // По центру
```

### Пример: элемент списка

```kotlin
Row(
    modifier = Modifier
        .fillMaxWidth()
        .padding(16.dp),
    horizontalArrangement = Arrangement.SpaceBetween,
    verticalAlignment = Alignment.CenterVertically
) {
    Row(verticalAlignment = Alignment.CenterVertically) {
        Image(
            painter = painterResource(R.drawable.avatar),
            contentDescription = null,
            modifier = Modifier.size(48.dp).clip(CircleShape),
            contentScale = ContentScale.Crop
        )
        Spacer(modifier = Modifier.width(12.dp))
        Column {
            Text("Иван Иванов", fontWeight = FontWeight.Bold)
            Text("Последнее сообщение", color = Color.Gray, fontSize = 14.sp)
        }
    }
    Text("12:30", color = Color.Gray, fontSize = 12.sp)
}
```

---

## Box — наложение (stack)

Располагает дочерние элементы **один поверх другого** (как FrameLayout).

```kotlin
Box(
    modifier = Modifier.size(200.dp),
    contentAlignment = Alignment.BottomEnd
) {
    Image(painter = painterResource(R.drawable.bg), contentDescription = null)
    Text("Подпись", color = Color.White, modifier = Modifier.padding(8.dp))
}
```

### contentAlignment

```kotlin
Alignment.TopStart       // Верх-лево
Alignment.TopCenter      // Верх-центр
Alignment.TopEnd         // Верх-право
Alignment.CenterStart    // Середина-лево
Alignment.Center         // Центр
Alignment.CenterEnd      // Середина-право
Alignment.BottomStart    // Низ-лево
Alignment.BottomCenter   // Низ-центр
Alignment.BottomEnd      // Низ-право
```

### Пример: карточка с наложением

```kotlin
Box(
    modifier = Modifier
        .fillMaxWidth()
        .height(200.dp)
        .clip(RoundedCornerShape(16.dp))
) {
    Image(
        painter = painterResource(R.drawable.banner),
        contentDescription = null,
        modifier = Modifier.fillMaxSize(),
        contentScale = ContentScale.Crop
    )
    Box(
        modifier = Modifier
            .fillMaxSize()
            .background(
                Brush.verticalGradient(
                    colors = listOf(Color.Transparent, Color.Black.copy(alpha = 0.7f))
                )
            )
    )
    Text(
        text = "Заголовок",
        color = Color.White,
        fontSize = 24.sp,
        fontWeight = FontWeight.Bold,
        modifier = Modifier
            .align(Alignment.BottomStart)
            .padding(16.dp)
    )
}
```

---

## Spacer

Отступ между элементами.

```kotlin
Spacer(modifier = Modifier.height(16.dp))  // Вертикальный
Spacer(modifier = Modifier.width(16.dp))   // Горизонтальный
```

---

## LazyColumn и LazyRow

Прокручиваемые списки (замена RecyclerView).

```kotlin
// Вертикальный список
LazyColumn(
    verticalArrangement = Arrangement.spacedBy(8.dp),
    contentPadding = PaddingValues(16.dp)
) {
    items(userList) { user ->
        UserCard(user)
    }

    items(10) { index ->
        Text("Элемент $index")
    }

    item {
        Text("Одиночный элемент")
    }

    // С ключами для лучшей производительности
    items(items = userList, key = { it.id }) { user ->
        UserCard(user)
    }
}

// Горизонтальный список
LazyRow(
    horizontalArrangement = Arrangement.spacedBy(8.dp),
    contentPadding = PaddingValues(16.dp)
) {
    items(photoList) { photo ->
        Image(
            painter = painterResource(photo),
            contentDescription = null,
            modifier = Modifier.size(150.dp).clip(RoundedCornerShape(8.dp)),
            contentScale = ContentScale.Crop
        )
    }
}
```

---

## Scaffold

Базовая структура экрана с Material-компонентами.

```kotlin
Scaffold(
    topBar = {
        TopAppBar(
            title = { Text("Приложение") },
            navigationIcon = {
                IconButton(onClick = { }) {
                    Icon(Icons.Default.ArrowBack, "Назад")
                }
            },
            actions = {
                IconButton(onClick = { }) {
                    Icon(Icons.Default.Search, "Поиск")
                }
            }
        )
    },
    bottomBar = {
        NavigationBar {
            NavigationBarItem(
                selected = true,
                onClick = { },
                icon = { Icon(Icons.Default.Home, "Главная") },
                label = { Text("Главная") }
            )
            NavigationBarItem(
                selected = false,
                onClick = { },
                icon = { Icon(Icons.Default.Person, "Профиль") },
                label = { Text("Профиль") }
            )
        }
    },
    floatingActionButton = {
        FloatingActionButton(onClick = { }) {
            Icon(Icons.Default.Add, "Добавить")
        }
    },
    snackbarHost = { SnackbarHost(snackbarHostState) }
) { paddingValues ->
    Column(modifier = Modifier.padding(paddingValues)) {
        Text("Контент")
    }
}
```

---

## Важные моменты

- `Column` — вертикальная компоновка (аналог LinearLayout vertical)
- `Row` — горизонтальная компоновка (аналог LinearLayout horizontal)
- `Box` — наложение элементов (аналог FrameLayout/ConstraintLayout)
- `LazyColumn` / `LazyRow` — списки с ленивой загрузкой (аналог RecyclerView)
- `Scaffold` — структура экрана с TopBar, BottomBar, FAB, Snackbar
- `Spacer` — для отступов между элементами
