# Compose: Модификаторы (Modifier)

Модификаторы — это цепочки операций, которые настраивают внешний вид и поведение composable-элемента. Порядок модификаторов **имеет значение**.

## Правило порядка

Модификаторы применяются **наружу → внутрь**. Каждое следующее修饰ение оборачивает предыдущее.

```kotlin
// padding сначала, потом click — кликабельная область больше
Text("A", modifier = Modifier.clickable {}.padding(16.dp))

// click сначала, потом padding — кликабельна только внутренняя область
Text("B", modifier = Modifier.padding(16.dp).clickable {})
```

---

## Размер

```kotlin
Modifier.size(200.dp)                       // Фиксированный размер
Modifier.size(width = 200.dp, height = 100.dp)  // Фиксированный WxH
Modifier.fillMaxSize()                       // Занять всё пространство
Modifier.fillMaxWidth()                      // Занять всю ширину
Modifier.fillMaxHeight()                     // Занять всю высоту
Modifier.wrapContentSize()                   // Обернуть по размеру контента
Modifier.wrapContentWidth()                  // Обернуть по ширине контента
Modifier.wrapContentHeight()                 // Обернуть по высоте контента
Modifier.heightIn(min = 50.dp, max = 200.dp) // Ограничение высоты
Modifier.widthIn(min = 100.dp)               // Ограничение ширины
Modifier.defaultMinSize(minWidth = 100.dp, minHeight = 50.dp)
```

## Отступы

```kotlin
Modifier.padding(16.dp)                                // Со всех сторон
Modifier.padding(horizontal = 16.dp, vertical = 8.dp)  // Горизонт/вертик
Modifier.padding(start = 8.dp, top = 4.dp, end = 8.dp, bottom = 4.dp) // Каждая сторона
Modifier.padding(top = 16.dp)                          // Только сверху
```

## Фон

```kotlin
Modifier.background(Color.Red)
Modifier.background(Color.Blue, RoundedCornerShape(12.dp))
Modifier.background(
    Brush.linearGradient(
        colors = listOf(Color.Red, Color.Blue)
    )
)
Modifier.background(
    Brush.verticalGradient(
        colors = listOf(Color.Cyan, Color.Magenta)
    ),
    shape = RoundedCornerShape(topStart = 16.dp, topEnd = 16.dp)
)
```

## Обрезка (clip)

```kotlin
Modifier.clip(CircleShape)                    // Круг
Modifier.clip(RoundedCornerShape(16.dp))      // Скруглённые углы
Modifier.clip(RoundedCornerShape(topStart = 16.dp, topEnd = 16.dp)) // Только верхние
Modifier.clip(RectangleShape)                 // Прямоугольник (по умолчанию)
Modifier.clip(CutCornerShape(16.dp))          // Срезанные углы
```

## Рамка (border)

```kotlin
Modifier.border(2.dp, Color.Red)
Modifier.border(2.dp, Color.Blue, RoundedCornerShape(12.dp))
Modifier.border(BorderStroke(2.dp, Color.Green), CircleShape)
```

## Кликабельность

```kotlin
Modifier.clickable { /* обработка клика */ }
Modifier.clickable(
    interactionSource = remember { MutableInteractionSource() },
    indication = null // Убрать ripple-эффект
) { /* клик без визуальной обратной связи */ }

// Расширенный клик
Modifier.combinedClickable(
    onClick = { /* клик */ },
    onDoubleClick = { /* двойной клик */ },
    onLongClick = { /* длинное нажатие */ }
)
```

## Прокрутка

```kotlin
Modifier.verticalScroll(rememberScrollState())
Modifier.horizontalScroll(rememberScrollState())
Modifier.scrollable(
    state = rememberScrollableState { delta -> delta },
    orientation = Orientation.Vertical
)
```

## Отступы внутри/снаружи

```kotlin
// offset — смещение (не влияет на компоновку других элементов)
Modifier.offset(x = 10.dp, y = 20.dp)
Modifier.offset { IntOffset(0, scrollState.value) } // Динамический
```

## Видимость и прозрачность

```kotlin
Modifier.alpha(0.5f)    // Полупрозрачность
Modifier.visible(true)  // Видимость (Compose 1.7+)
```

## Z-порядок (elevation / shadow)

```kotlin
Modifier.shadow(8.dp)
Modifier.shadow(8.dp, RoundedCornerShape(16.dp))
Modifier.shadow(8.dp, shape = CircleShape, clip = true)
```

## Вес (в Row/Column)

```kotlin
// В Row — распределение ширины
Row {
    Text("A", modifier = Modifier.weight(1f))
    Text("B", modifier = Modifier.weight(2f)) // В 2 раза шире
    Text("C") // Фиксированный размер
}

// В Column — распределение высоты
Column {
    Text("Top", modifier = Modifier.weight(1f))
    Text("Bottom", modifier = Modifier.weight(3f))
}
```

## Аспектное отношение

```kotlin
Modifier.aspectRatio(16f / 9f)  // Широкое видео
Modifier.aspectRatio(1f)         // Квадрат
```

## Включение/отключение

```kotlin
Modifier.enabled(false) // Отключить взаимодействие
```

## Фокус и ввод

```kotlin
Modifier.focusable()
Modifier.clickable { /* ... */ }
Modifier.pointerInput(Unit) {
    detectTapGestures(
        onPress = { /* нажатие */ },
        onDoubleTap = { /* двойной тап */ },
        onLongPress = { /* длинное нажатие */ },
        onTap = { /* тап */ }
    )
}
Modifier.pointerInput(Unit) {
    detectDragGestures { change, dragAmount ->
        // Обработка перетаскивания
    }
}
```

## Тестирование

```kotlin
Modifier.testTag("myButton")     // Тег для тестирования
Modifier.semantics { contentDescription = "Кнопка входа" } // Для accessibility
```

## Комбинирование модификаторов

```kotlin
val commonModifier = Modifier
    .fillMaxWidth()
    .padding(horizontal = 16.dp, vertical = 8.dp)

// Использование
Text("Текст", modifier = commonModifier)
Button(onClick = { }, modifier = commonModifier) {
    Text("Кнопка")
}

// Расширение существующего
Text("Текст", modifier = commonModifier.background(Color.LightGray))
```

## Кастомный модификатор

```kotlin
// Простой модификатор
fun Modifier.myStyle() = this
    .padding(16.dp)
    .background(Color.White, RoundedCornerShape(8.dp))
    .border(1.dp, Color.Gray, RoundedCornerShape(8.dp))

// Использование
Text("Привет", modifier = Modifier.myStyle())
```

## Частые паттерны

```kotlin
// Карточка
Card(
    modifier = Modifier
        .fillMaxWidth()
        .padding(16.dp),
    shape = RoundedCornerShape(12.dp),
    elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
) {
    Column(modifier = Modifier.padding(16.dp)) {
        Text("Заголовок", fontWeight = FontWeight.Bold)
        Text("Описание", color = Color.Gray)
    }
}

// Круглая кнопка
Box(
    modifier = Modifier
        .size(56.dp)
        .clip(CircleShape)
        .background(Color.Blue)
        .clickable { },
    contentAlignment = Alignment.Center
) {
    Icon(Icons.Default.Add, "", tint = Color.White)
}

// Элемент списка с ripple
Row(
    modifier = Modifier
        .fillMaxWidth()
        .clickable { }
        .padding(16.dp)
) {
    Text("Item")
}
```

## Важные моменты

- Порядок модификаторов **критически важен**: `padding → clickable` ≠ `clickable → padding`
- `Modifier` — неизменяемый, каждая функция возвращает **новый** Modifier
- Модификаторы — первый именованный параметр composable-функции (по конвенции)
- Используйте цепочку модификаторов для описания всего визуального поведения элемента
