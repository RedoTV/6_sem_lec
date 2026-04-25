# Compose: Обработка событий (лямбды в composable)

В Jetpack Compose события обрабатываются через **лямбда-выражения**, передаваемые как параметры composable-функций.

---

## Click — обычный клик

```kotlin
Button(onClick = {
    // Действие при клике
    Log.d("TAG", "Кнопка нажата")
}) {
    Text("Нажми")
}

// clickable-модификатор (для любого элемента)
Text(
    text = "Нажми на текст",
    modifier = Modifier.clickable {
        Log.d("TAG", "Текст нажат")
    }
)

// combinedClickable (длинное нажатие, двойной клик)
Text(
    text = "Нажми",
    modifier = Modifier.combinedClickable(
        onClick = { /* обычный клик */ },
        onDoubleClick = { /* двойной клик */ },
        onLongClick = { /* длинное нажатие */ }
    )
)
```

## TextField — изменение текста

```kotlin
var text by remember { mutableStateOf("") }

TextField(
    value = text,
    onValueChange = { newText ->
        text = newText
        // Дополнительная логика
        if (newText.length > 10) {
            Log.d("TAG", "Текст слишком длинный")
        }
    }
)
```

## Переключатели (Switch, Checkbox, RadioButton)

```kotlin
// Switch
var isOn by remember { mutableStateOf(false) }
Switch(
    checked = isOn,
    onCheckedChange = { newState ->
        isOn = newState
    }
)

// Checkbox
var isChecked by remember { mutableStateOf(false) }
Checkbox(
    checked = isChecked,
    onCheckedChange = { isChecked = it }
)

// TriStateCheckbox (три состояния)
var state by remember { mutableStateOf(ToggleableState.Indeterminate) }
TriStateCheckbox(
    state = state,
    onClick = {
        state = when (state) {
            ToggleableState.On -> ToggleableState.Off
            ToggleableState.Off -> ToggleableState.Indeterminate
            ToggleableState.Indeterminate -> ToggleableState.On
        }
    }
)

// RadioButton
var selected by remember { mutableStateOf(0) }
Column {
    (0..2).forEach { index ->
        Row(
            modifier = Modifier.clickable { selected = index },
            verticalAlignment = Alignment.CenterVertically
        ) {
            RadioButton(
                selected = selected == index,
                onClick = { selected = index }
            )
            Text("Вариант ${index + 1}")
        }
    }
}
```

## Slider

```kotlin
var sliderValue by remember { mutableStateOf(0.5f) }
Slider(
    value = sliderValue,
    onValueChange = { sliderValue = it },
    valueRange = 0f..100f,
    steps = 9, // 10 сегментов
    onValueChangeFinished = {
        Log.d("TAG", "Финальное значение: $sliderValue")
    }
)
```

## Scroll — прокрутка

```kotlin
// Вертикальный скролл
val scrollState = rememberScrollState()
Column(
    modifier = Modifier.verticalScroll(scrollState)
) {
    repeat(100) {
        Text("Элемент $it", modifier = Modifier.padding(16.dp))
    }
}

// Программная прокрутка
LaunchedEffect(Unit) {
    scrollState.animateScrollTo(500)
}

// Горизонтальный скролл
val hScrollState = rememberScrollState()
Row(modifier = Modifier.horizontalScroll(hScrollState)) {
    repeat(50) {
        Text("Item $it", modifier = Modifier.padding(16.dp))
    }
}
```

## Snackbar

```kotlin
val snackbarHostState = remember { SnackbarHostState() }
val scope = rememberCoroutineScope()

Scaffold(snackbarHost = { SnackbarHost(snackbarHostState) }) {
    Column(modifier = Modifier.padding(it)) {
        Button(onClick = {
            scope.launch {
                snackbarHostState.showSnackbar(
                    message = "Действие выполнено",
                    actionLabel = "Отменить",
                    duration = SnackbarDuration.Short
                )
            }
        }) {
            Text("Показать Snackbar")
        }
    }
}
```

## Диалоги

```kotlin
var showDialog by remember { mutableStateOf(false) }

if (showDialog) {
    AlertDialog(
        onDismissRequest = { showDialog = false },
        title = { Text("Подтверждение") },
        text = { Text("Вы уверены, что хотите удалить?") },
        confirmButton = {
            TextButton(onClick = {
                // Действие
                showDialog = false
            }) {
                Text("Удалить")
            }
        },
        dismissButton = {
            TextButton(onClick = { showDialog = false }) {
                Text("Отмена")
            }
        }
    )
}

Button(onClick = { showDialog = true }) {
    Text("Показать диалог")
}
```

## Всплывающее меню (DropdownMenu)

```kotlin
var expanded by remember { mutableStateOf(false) }
var selectedOption by remember { mutableStateOf("Выберите") }

Box {
    Button(onClick = { expanded = true }) {
        Text(selectedOption)
    }
    DropdownMenu(
        expanded = expanded,
        onDismissRequest = { expanded = false }
    ) {
        DropdownMenuItem(
            text = { Text("Вариант 1") },
            onClick = {
                selectedOption = "Вариант 1"
                expanded = false
            }
        )
        DropdownMenuItem(
            text = { Text("Вариант 2") },
            onClick = {
                selectedOption = "Вариант 2"
                expanded = false
            }
        )
    }
}
```

## Навигация

```kotlin
// Зависимость: implementation("androidx.navigation:navigation-compose:2.7.6")

val navController = rememberNavController()

NavHost(navController, startDestination = "home") {
    composable("home") {
        HomeScreen(
            onNavigateToDetail = { id ->
                navController.navigate("detail/$id")
            }
        )
    }
    composable(
        "detail/{id}",
        arguments = listOf(navArgument("id") { type = NavType.IntType })
    ) { backStackEntry ->
        val id = backStackEntry.arguments?.getInt("id") ?: 0
        DetailScreen(id, onBack = { navController.popBackStack() })
    }
}

// Переход
navController.navigate("detail/42")

// Назад
navController.popBackStack()
```

## Важные моменты

- Все события — **лямбды**, передаваемые в composable-функции
- Состояние UI хранится в `remember { mutableStateOf(...) }` или во `ViewModel`
- `onClick` — главный обработчик кликов
- `onValueChange` — обработчик изменения значения (TextField, Slider, Switch и т.д.)
- `onDismissRequest` — обработчик закрытия (Dialog, DropdownMenu)
- Для побочных эффектов (snackbar, навигация после события) используйте `LaunchedEffect` и `rememberCoroutineScope()`
