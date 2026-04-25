# Java/XML: Атрибуты для стилизации XML

## Общие атрибуты View

### Размеры

```xml
android:layout_width="match_parent"     <!-- Заполнить родителя -->
android:layout_width="wrap_content"     <!-- По содержимому -->
android:layout_width="200dp"            <!-- Фиксированный -->

android:layout_height="match_parent"
android:layout_height="wrap_content"
android:layout_height="100dp"

android:minWidth="50dp"
android:minHeight="50dp"
```

### Отступы

```xml
android:padding="16dp"                              <!-- Со всех сторон -->
android:paddingStart="8dp"
android:paddingEnd="8dp"
android:paddingTop="4dp"
android:paddingBottom="4dp"
android:paddingHorizontal="16dp"                    <!-- API 26+ -->
android:paddingVertical="8dp"                       <!-- API 26+ -->
```

### Внешние отступы

```xml
android:layout_margin="16dp"
android:layout_marginStart="8dp"
android:layout_marginEnd="8dp"
android:layout_marginTop="4dp"
android:layout_marginBottom="4dp"
android:layout_marginHorizontal="16dp"              <!-- API 26+ -->
android:layout_marginVertical="8dp"                 <!-- API 26+ -->
```

### Фон

```xml
android:background="#FF6200EE"                       <!-- Цвет -->
android:background="@color/purple"                   <!-- Из ресурсов -->
android:background="@drawable/my_bg"                 <!-- Drawable -->
android:background="@drawable/gradient_bg"           <!-- Gradient drawable -->
android:backgroundTint="#80000000"                   <!-- Наложение цвета -->
android:backgroundTintMode="src_over"                <!-- Режим наложения -->
```

### Видимость

```xml
android:visibility="visible"      <!-- Видим (по умолчанию) -->
android:visibility="invisible"    <!-- Невидим, но занимает место -->
android:visibility="gone"         <!-- Невидим и НЕ занимает место -->
```

Из кода:
```java
view.setVisibility(View.VISIBLE);
view.setVisibility(View.INVISIBLE);
view.setVisibility(View.GONE);

boolean isVisible = view.getVisibility() == View.VISIBLE;
```

### Прозрачность

```xml
android:alpha="0.5"    <!-- 0.0 = полностью прозрачный, 1.0 = непрозрачный -->
```

```java
view.setAlpha(0.5f);
```

### Включение/отключение

```xml
android:enabled="false"
android:clickable="true"
android:focusable="true"
```

```java
view.setEnabled(false);
view.setClickable(true);
```

### Elevation (тень)

```xml
android:elevation="8dp"
android:translationZ="4dp"
```

```java
view.setElevation(8f);
```

---

## Стили (styles.xml)

Стиль — набор атрибутов, который можно применить к View.

```xml
<!-- res/values/styles.xml -->
<style name="MyButtonStyle">
    <item name="android:layout_width">match_parent</item>
    <item name="android:layout_height">wrap_content</item>
    <item name="android:textColor">#FFFFFF</item>
    <item name="android:textSize">16sp</item>
    <item name="android:backgroundTint">#6200EE</item>
    <item name="android:textAllCaps">false</item>
    <item name="android:padding">12dp</item>
</style>
```

Применение:
```xml
<Button
    style="@style/MyButtonStyle"
    android:text="Кнопка" />
```

### Наследование стилей

```xml
<!-- Наследует от MyButtonStyle и переопределяет цвет -->
<style name="MyButtonStyle.Red">
    <item name="android:backgroundTint">#FF0000</item>
</style>

<!-- Наследование от системного стиля -->
<style name="MyTextView" parent="Widget.AppCompat.TextView">
    <item name="android:textSize">18sp</item>
    <item name="android:textColor">#333333</item>
</style>
```

---

## Темы (themes.xml)

Тема — стиль, применяемый ко всему Activity или приложению.

```xml
<!-- res/values/themes.xml -->
<style name="Theme.MyApp" parent="Theme.MaterialComponents.DayNight.DarkActionBar">
    <item name="colorPrimary">#6200EE</item>
    <item name="colorPrimaryVariant">#3700B3</item>
    <item name="colorOnPrimary">#FFFFFF</item>
    <item name="colorSecondary">#03DAC5</item>
    <item name="colorSecondaryVariant">#018786</item>
    <item name="colorOnSecondary">#000000</item>
    <item name="android:statusBarColor">#3700B3</item>
    <item name="android:navigationBarColor">#000000</item>
    <item name="android:windowBackground">#FFFFFF</item>
</style>
```

### Цвета темы

| Атрибут | Описание |
|---------|----------|
| `colorPrimary` | Основной цвет (Toolbar, кнопки) |
| `colorPrimaryVariant` | Тёмный вариант основного |
| `colorOnPrimary` | Цвет текста/иконок на основном |
| `colorSecondary` | Дополнительный цвет (FAB, Switch) |
| `colorSecondaryVariant` | Тёмный вариант дополнительного |
| `colorOnSecondary` | Цвет текста на дополнительном |
| `colorSurface` | Цвет поверхностей (Card, Dialog) |
| `colorOnSurface` | Цвет текста на поверхности |
| `colorError` | Цвет ошибок |
| `colorOnError` | Цвет текста на ошибке |

### Применение темы

В манифесте:
```xml
<application android:theme="@style/Theme.MyApp">
<!-- Или для конкретного Activity -->
<activity android:theme="@style/Theme.MyApp.NoActionBar">
```

### Тема без ActionBar

```xml
<style name="Theme.MyApp.NoActionBar">
    <item name="windowActionBar">false</item>
    <item name="windowNoTitle">true</item>
</style>
```

### Ночная тема

```xml
<!-- res/values-night/themes.xml -->
<style name="Theme.MyApp" parent="Theme.MaterialComponents.DayNight.DarkActionBar">
    <item name="colorPrimary">#BB86FC</item>
    <item name="android:windowBackground">#121212</item>
</style>
```

---

## TextAppearance — стили для текста

```xml
<style name="TextAppearance.MyApp.Headline" parent="TextAppearance.AppCompat.Headline">
    <item name="android:textSize">24sp</item>
    <item name="android:textColor">#000000</item>
    <item name="android:fontFamily">sans-serif-medium</item>
    <item name="android:letterSpacing">0.01</item>
</style>
```

```xml
<TextView
    android:textAppearance="@style/TextAppearance.MyApp.Headline"
    android:text="Заголовок" />
```

---

## Drawable (графика)

### Shape Drawable (фигура)

```xml
<!-- res/drawable/bg_rounded.xml -->
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">

    <solid android:color="#6200EE" />
    <corners android:radius="12dp" />
    <stroke android:width="2dp" android:color="#3700B3" />
    <padding android:left="8dp" android:right="8dp" android:top="4dp" android:bottom="4dp" />

</shape>
```

Виды shape: `rectangle`, `oval`, `line`, `ring`.

```xml
<!-- Овал (круг при одинаковых размерах) -->
<shape android:shape="oval">
    <solid android:color="#FF0000" />
</shape>

<!-- Градиент -->
<shape android:shape="rectangle">
    <gradient
        android:startColor="#6200EE"
        android:endColor="#03DAC5"
        android:angle="135"
        android:type="linear" />
    <corners android:radius="16dp" />
</shape>
```

### Ripple Drawable

```xml
<!-- res/drawable/ripple_effect.xml -->
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="#33FFFFFF">
    <item>
        <shape android:shape="rectangle">
            <solid android:color="#6200EE" />
            <corners android:radius="12dp" />
        </shape>
    </item>
</ripple>
```

### Selector (состояния)

```xml
<!-- res/drawable/button_selector.xml -->
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true">
        <shape android:shape="rectangle">
            <solid android:color="#3700B3" />
            <corners android:radius="8dp" />
        </shape>
    </item>

    <item android:state_enabled="false">
        <shape android:shape="rectangle">
            <solid android:color="#CCCCCC" />
            <corners android:radius="8dp" />
        </shape>
    </item>

    <item>
        <shape android:shape="rectangle">
            <solid android:color="#6200EE" />
            <corners android:radius="8dp" />
        </shape>
    </item>
</selector>
```

Состояния:
- `state_pressed` — нажат
- `state_enabled` — включён
- `state_focused` — в фокусе
- `state_selected` — выбран
- `state_checked` — отмечен (CheckBox, Switch)

### Layer List (слои)

```xml
<!-- res/drawable/bg_card.xml -->
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Тень -->
    <item android:top="4dp">
        <shape android:shape="rectangle">
            <solid android:color="#10000000" />
            <corners android:radius="12dp" />
        </shape>
    </item>
    <!-- Фон -->
    <item android:bottom="4dp">
        <shape android:shape="rectangle">
            <solid android:color="#FFFFFF" />
            <corners android:radius="12dp" />
        </shape>
    </item>
</layer-list>
```

---

## Сравнение: padding vs margin

```
margin — отступ СНАРУЖИ элемента (между элементом и соседями)
padding — отступ ВНУТРИ элемента (между границей и содержимым)

┌──────── margin ────────┐
│  ┌──── padding ────┐   │
│  │                  │   │
│  │    Контент       │   │
│  │                  │   │
│  └──────────────────┘   │
└─────────────────────────┘
```

## Важные моменты

- **Стили** — набор атрибутов для одного View (`style="@style/..."`)
- **Темы** — стиль для всего Activity/приложения (`android:theme="..."`)
- **Drawable** — графические ресурсы (shape, selector, ripple, layer-list)
- `padding` — внутренний отступ, `margin` — внешний
- `visibility="gone"` — элемент не виден и не занимает место
- `visibility="invisible"` — не виден, но занимает место
