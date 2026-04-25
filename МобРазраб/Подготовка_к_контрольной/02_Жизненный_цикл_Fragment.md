# Жизненный цикл Fragment

Fragment — это модульная часть UI внутри Activity. Имеет собственный жизненный цикл, который тесно связан с жизненным циклом хост-Activity.

## Состояния Fragment

| Состояние | Описание |
|-----------|----------|
| **INITIALIZED** | Fragment создан через конструктор, но ещё не добавлен в FragmentManager |
| **CREATED** | Fragment добавлен в FragmentManager, прошли onCreateView/onViewStateRestored |
| **STARTED** | Fragment виден, но не в фокусе |
| **RESUMED** | Fragment видим и в фокусе (пользователь может взаимодействовать) |
| **DESTROYED** | Fragment удалён из FragmentManager |

## Основные callback-методы

```kotlin
class MyFragment : Fragment() {

    override fun onAttach(context: Context) {
        super.onAttach(context)
        // Fragment привязан к Activity
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Инициализация, НЕ связанная с UI
    }

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        // Создание и возврат View фрагмента
        return inflater.inflate(R.layout.fragment_my, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        // View создана, здесь настраиваем UI, адаптеры, слушатели
    }

    override fun onViewStateRestored(savedInstanceState: Bundle?) {
        super.onViewStateRestored(savedInstanceState)
        // Состояние View восстановлено
    }

    override fun onStart() {
        super.onStart()
        // Fragment виден пользователю
    }

    override fun onResume() {
        super.onResume()
        // Fragment в фокусе, пользователь взаимодействует
    }

    override fun onPause() {
        super.onPause()
        // Fragment теряет фокус
    }

    override fun onStop() {
        super.onStop()
        // Fragment больше не виден
    }

    override fun onDestroyView() {
        super.onDestroyView()
        // View фрагмента уничтожена, освободить ссылки на View
    }

    override fun onDestroy() {
        super.onDestroy()
        // Fragment уничтожается
    }

    override fun onDetach() {
        super.onDetach()
        // Fragment отвязан от Activity
    }
}
```

## Порядок вызовов

```
Добавление Fragment:
  onAttach → onCreate → onCreateView → onViewCreated →
  onViewStateRestored → onStart → onResume

Удаление Fragment:
  onPause → onStop → onDestroyView → onDestroy → onDetach

Замена Fragment (replace):
  onPause → onStop → onDestroyView → (новый fragment: onAttach → onCreate → ...)
  При addToBackStack: onDestroyView (View уничтожена, fragment в стеке)
  Возврат из back stack: onCreateView → onViewCreated → onViewStateRestored → onStart → onResume
```

## Связь с жизненным циклом Activity

```
Activity onCreate  → Fragment onAttach, onCreate, onCreateView, onViewCreated
Activity onStart   → Fragment onStart
Activity onResume  → Fragment onResume
Activity onPause   → Fragment onPause
Activity onStop    → Fragment onStop
Activity onDestroy → Fragment onDestroyView, onDestroy, onDetach
```

## FragmentManager и FragmentTransaction

```kotlin
// Добавление фрагмента
supportFragmentManager.commit {
    add(R.id.fragment_container, MyFragment::class.java, null)
}

// Замена фрагмента с добавлением в back stack
supportFragmentManager.commit {
    replace(R.id.fragment_container, MyFragment::class.java, null)
    addToBackStack("tag")
}

// Поиск фрагмента
val fragment = supportFragmentManager.findFragmentById(R.id.fragment_container)
```

## Передача данных во Fragment

```kotlin
// Через Bundle
val fragment = MyFragment().apply {
    arguments = Bundle().apply {
        putString("key", "value")
    }
}

// Чтение во Fragment
val value = arguments?.getString("key")
```

## Fragment Result API (рекомендуемый способ)

```kotlin
// В принимающем фрагменте
setFragmentResultListener("requestKey") { key, bundle ->
    val result = bundle.getString("resultKey")
}

// В отправляющем фрагменте
setFragmentResult("requestKey", Bundle().apply {
    putString("resultKey", "value")
})
```

## Важные моменты

- Fragment **всегда** живёт внутри Activity, его ЖЦ зависит от ЖЦ Activity
- `onDestroyView` вызывается без `onDestroy`, если Fragment в back stack (View уничтожена, но Fragment жив)
- `onCreate` — для инициализации, не связанной с UI (например, загрузка данных)
- `onViewCreated` — для настройки UI (слушатели, адаптеры)
- `onDestroyView` — очищайте ссылки на View, чтобы избежать утечек памяти
