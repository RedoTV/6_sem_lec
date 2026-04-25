# Compose: Анимации

Jetpack Compose предоставляет несколько уровней API для анимаций: от высокоуровневых (простых) до низкоуровневых (полный контроль).

---

## 1. AnimatedVisibility — появление/исчезновение

```kotlin
var visible by remember { mutableStateOf(true) }

Column {
    Button(onClick = { visible = !visible }) {
        Text("Переключить")
    }

    AnimatedVisibility(
        visible = visible,
        enter = fadeIn() + expandVertically(),
        exit = fadeOut() + shrinkVertically()
    ) {
        Text("Я появляюсь и исчезаю!")
    }
}
```

### Варианты enter/exit

```kotlin
// Появление
enter = fadeIn(initialAlpha = 0f)
enter = slideInVertically(initialOffsetY = { it }) // Снизу
enter = slideInHorizontally(initialOffsetX = { -it }) // Слева
enter = expandVertically(expandFrom = Alignment.Top)
enter = expandHorizontally()
enter = scaleIn(initialScale = 0f)
enter = fadeIn() + expandVertically() // Комбинирование

// Исчезновение
exit = fadeOut(targetAlpha = 0f)
exit = slideOutVertically(targetOffsetY = { it })
exit = slideOutHorizontally(targetOffsetX = { it })
exit = shrinkVertically(shrinkTowards = Alignment.Top)
exit = scaleOut(targetScale = 0f)
exit = fadeOut() + shrinkVertically()
```

### AnimatedVisibility для отдельных элементов (в Row/Column)

```kotlin
Row {
    AnimatedVisibility(visible = showIcon) {
        Icon(Icons.Default.Star, "star")
    }
    Text("Текст")
}
```

---

## 2. animateContentSize — анимация изменения размера

```kotlin
var expanded by remember { mutableStateOf(false) }

Column(
    modifier = Modifier
        .fillMaxWidth()
        .animateContentSize(
            animationSpec = spring(
                dampingRatio = Spring.DampingRatioLowBouncy,
                stiffness = Spring.StiffnessLow
            )
        )
        .background(Color.LightGray, RoundedCornerShape(8.dp))
        .padding(16.dp)
) {
    Text("Заголовок", fontWeight = FontWeight.Bold)
    if (expanded) {
        Text(
            "Дополнительный текст, который появляется при раскрытии. "
            + "Размер контейнера анимированно увеличивается."
        )
    }
    Button(onClick = { expanded = !expanded }) {
        Text(if (expanded) "Свернуть" else "Развернуть")
    }
}
```

---

## 3. animate*AsState — анимация значения

```kotlin
var isOn by remember { mutableStateOf(false) }

val color by animateColorAsState(
    targetValue = if (isOn) Color.Green else Color.Red,
    animationSpec = tween(durationMillis = 500)
)

val size by animateDpAsState(
    targetValue = if (isOn) 100.dp else 50.dp,
    animationSpec = spring(dampingRatio = Spring.DampingRatioHighBouncy)
)

val alpha by animateFloatAsState(
    targetValue = if (isOn) 1f else 0.3f,
    animationSpec = tween(300)
)

Column(horizontalAlignment = Alignment.CenterHorizontally) {
    Box(
        modifier = Modifier
            .size(size)
            .background(color, CircleShape)
            .alpha(alpha)
    )
    Spacer(modifier = Modifier.height(16.dp))
    Button(onClick = { isOn = !isOn }) {
        Text("Переключить")
    }
}
```

### Доступные animateAsState

| Функция | Тип значения |
|---------|-------------|
| `animateFloatAsState` | Float |
| `animateDpAsState` | Dp |
| `animateIntAsState` | Int |
| `animateColorAsState` | Color |
| `animateOffsetAsState` | Offset |
| `animateRectAsState` | Rect |
| `animateSizeAsState` | Size |
| `animateValueAsState<T>` | Любой тип (нужен TwoWayConverter) |

---

## 4. AnimatedContent — переключение контента с анимацией

```kotlin
var count by remember { mutableStateOf(0) }

AnimatedContent(
    targetState = count,
    transitionSpec = {
        slideIntoContainer(AnimatedContentTransitionScope.SlideDirection.Up) togetherWith
            slideOutOfContainer(AnimatedContentTransitionScope.SlideDirection.Down)
    },
    label = "counter"
) { targetCount ->
    Text(
        text = "$targetCount",
        fontSize = 48.sp,
        fontWeight = FontWeight.Bold
    )
}

Button(onClick = { count++ }) {
    Text("+1")
}
```

### Направления слайдов

```kotlin
SlideDirection.Left
SlideDirection.Right
SlideDirection.Up
SlideDirection.Down
```

---

## 5. updateTransition — управление несколькими анимациями

```kotlin
enum class BoxState { Collapsed, Expanded }

var boxState by remember { mutableStateOf(BoxState.Collapsed) }
val transition = updateTransition(targetState = boxState, label = "transition")

val size by transition.animateDp(
    transitionSpec = { tween(500) },
    label = "size"
) { state ->
    when (state) {
        BoxState.Collapsed -> 50.dp
        BoxState.Expanded -> 200.dp
    }
}

val color by transition.animateColor(
    transitionSpec = { tween(500) },
    label = "color"
) { state ->
    when (state) {
        BoxState.Collapsed -> Color.Red
        BoxState.Expanded -> Color.Green
    }
}

val corner by transition.animateFloat(
    transitionSpec = { tween(500) },
    label = "corner"
) { state ->
    when (state) {
        BoxState.Collapsed -> 0f
        BoxState.Expanded -> 50f
    }
}

Box(
    modifier = Modifier
        .size(size)
        .background(color, RoundedCornerShape(corner))
        .clickable {
            boxState = when (boxState) {
                BoxState.Collapsed -> BoxState.Expanded
                BoxState.Expanded -> BoxState.Collapsed
            }
        }
)
```

---

## 6. rememberInfiniteTransition — бесконечная анимация

```kotlin
val infiniteTransition = rememberInfiniteTransition(label = "pulse")

val scale by infiniteTransition.animateFloat(
    initialValue = 1f,
    targetValue = 1.2f,
    animationSpec = infiniteRepeatable(
        animation = tween(1000, easing = LinearEasing),
        repeatMode = RepeatMode.Reverse
    ),
    label = "scale"
)

val alpha by infiniteTransition.animateFloat(
    initialValue = 0.3f,
    targetValue = 1f,
    animationSpec = infiniteRepeatable(
        animation = tween(800, easing = FastOutSlowInEasing),
        repeatMode = RepeatMode.Reverse
    ),
    label = "alpha"
)

Box(
    modifier = Modifier
        .size(80.dp * scale)
        .alpha(alpha)
        .background(Color.Red, CircleShape)
)
```

---

## 7. Animatable — низкоуровневая анимация

```kotlin
val scale = remember { Animatable(1f) }
val scope = rememberCoroutineScope()

LaunchedEffect(Unit) {
    scale.animateTo(
        targetValue = 1.5f,
        animationSpec = tween(500)
    )
    scale.animateTo(
        targetValue = 1f,
        animationSpec = spring(dampingRatio = Spring.DampingRatioHighBouncy)
    )
}

Box(
    modifier = Modifier
        .size(100.dp * scale.value)
        .background(Color.Blue, CircleShape)
        .clickable {
            scope.launch {
                scale.animateTo(1.5f, tween(200))
                scale.animateTo(1f, spring(dampingRatio = Spring.DampingRatioMediumBouncy))
            }
        }
)
```

---

## AnimationSpec — спецификации анимации

### tween (линейная/с easing)

```kotlin
tween<Float>(
    durationMillis = 300,
    delayMillis = 0,
    easing = FastOutSlowInEasing // по умолчанию
)

// Easing-функции:
FastOutSlowInEasing    // Быстрый старт, медленный конец
LinearOutSlowInEasing  // Линейный старт, медленный конец
FastOutLinearInEasing  // Быстрый старт, линейный конец
LinearEasing           // Равномерно
```

### spring (пружина)

```kotlin
spring<Float>(
    dampingRatio = Spring.DampingRatioNoBouncy,    // Без отскока
    // dampingRatio = Spring.DampingRatioLowBouncy,  // Много отскоков
    // dampingRatio = Spring.DampingRatioMediumBouncy,
    // dampingRatio = Spring.DampingRatioHighBouncy,  // Мало отскоков
    stiffness = Spring.StiffnessLow,                // Медленная
    // stiffness = Spring.StiffnessMedium,
    // stiffness = Spring.StiffnessHigh               // Быстрая
)
```

### keyframes (ключевые кадры)

```kotlin
keyframes<Float> {
    durationMillis = 1000
    0f at 0 using LinearEasing
    0.5f at 500 using FastOutSlowInEasing
    1f at 1000
}
```

### repeatable / infiniteRepeatable

```kotlin
repeatable<Float>(
    iterations = 3,
    animation = tween(500),
    repeatMode = RepeatMode.Restart  // Или RepeatMode.Reverse
)

infiniteRepeatable<Float>(
    animation = tween(500),
    repeatMode = RepeatMode.Reverse
)
```

### snap (мгновенная)

```kotlin
snap<Float>()
```

---

## 8. Modifier.graphicsLayer — анимация трансформаций

```kotlin
var rotation by remember { mutableStateOf(0f) }

Box(
    modifier = Modifier
        .size(100.dp)
        .background(Color.Blue)
        .graphicsLayer {
            rotationZ = rotation
            scaleX = 1f
            scaleY = 1f
            translationX = 0f
            translationY = 0f
            alpha = 1f
            shadowElevation = 0f
            shape = CircleShape
            clip = true
        }
        .clickable { rotation += 45f }
)

// Анимированная версия
val animatedRotation by animateFloatAsState(
    targetValue = rotation,
    animationSpec = tween(300)
)

Box(
    modifier = Modifier
        .graphicsLayer { rotationZ = animatedRotation }
)
```

---

## Важные моменты

- **AnimatedVisibility** — самый простой способ анимировать появление/исчезновение
- **animateContentSize** — для плавного изменения размера
- **animate*AsState** — анимация одного значения при изменении состояния
- **AnimatedContent** — анимация переключения между разными composable
- **updateTransition** — координация нескольких анимаций одновременно
- **rememberInfiniteTransition** — для повторяющихся (пульсирующих) анимаций
- **spring** — используется по умолчанию, даёт физически правдоподобную анимацию
- **tween** — для точного контроля длительности и easing
