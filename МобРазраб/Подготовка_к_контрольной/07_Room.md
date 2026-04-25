# Работа с данными. Room

Room — это библиотека-обёртка над SQLite, входящая в Android Jetpack. Предоставляет абстракцию для работы с локальной базой данных.

## Добавление зависимости

```kotlin
// build.gradle.kts (app)
dependencies {
    val roomVersion = "2.6.1"
    implementation("androidx.room:room-runtime:$roomVersion")
    implementation("androidx.room:room-ktx:$roomVersion")
    ksp("androidx.room:room-compiler:$roomVersion")
}
```

## Основные компоненты Room

| Компонент | Описание |
|-----------|----------|
| **@Entity** | Описание таблицы (класс данных) |
| **@Dao** | Интерфейс для SQL-запросов |
| **@Database** | Класс базы данных, связывающий Entity и Dao |

## 1. Entity — сущность (таблица)

```kotlin
@Entity(tableName = "users")
data class User(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,

    @ColumnInfo(name = "first_name")
    val firstName: String,

    @ColumnInfo(name = "last_name")
    val lastName: String,

    @ColumnInfo(name = "age")
    val age: Int,

    @ColumnInfo(name = "email")
    val email: String
)
```

### Составной первичный ключ
```kotlin
@Entity(primaryKeys = ["first_name", "last_name"])
data class User(
    @ColumnInfo(name = "first_name") val firstName: String,
    @ColumnInfo(name = "last_name") val lastName: String,
    val age: Int
)
```

### Внешние ключи
```kotlin
@Entity(
    tableName = "orders",
    foreignKeys = [
        ForeignKey(
            entity = User::class,
            parentColumns = ["id"],
            childColumns = ["user_id"],
            onDelete = ForeignKey.CASCADE
        )
    ]
)
data class Order(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    @ColumnInfo(name = "user_id") val userId: Int,
    val amount: Double,
    val description: String
)
```

### Игнорирование поля
```kotlin
@Entity(tableName = "users")
data class User(
    @PrimaryKey val id: Int,
    val name: String,
    @Ignore val tempData: String // Не будет в таблице
)
```

## 2. DAO — Data Access Object

```kotlin
@Dao
interface UserDao {

    // INSERT
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(user: User): Long

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(users: List<User>)

    // UPDATE
    @Update
    suspend fun update(user: User)

    // DELETE
    @Delete
    suspend fun delete(user: User)

    // QUERY — получение всех
    @Query("SELECT * FROM users")
    suspend fun getAll(): List<User>

    // QUERY — Flow (реактивное обновление)
    @Query("SELECT * FROM users")
    fun getAllFlow(): Flow<List<User>>

    // QUERY — по ID
    @Query("SELECT * FROM users WHERE id = :id")
    suspend fun getById(id: Int): User?

    // QUERY — фильтрация
    @Query("SELECT * FROM users WHERE age > :minAge ORDER BY age DESC")
    suspend fun getOlderThan(minAge: Int): List<User>

    // QUERY — поиск
    @Query("SELECT * FROM users WHERE first_name LIKE '%' || :query || '%'")
    suspend fun searchByName(query: String): List<User>

    // QUERY — удаление
    @Query("DELETE FROM users WHERE id = :id")
    suspend fun deleteById(id: Int)

    // QUERY — подсчёт
    @Query("SELECT COUNT(*) FROM users")
    suspend fun count(): Int
}
```

### OnConflictStrategy
- `ABORT` — отменить транзакцию (по умолчанию)
- `REPLACE` — заменить существующую запись
- `IGNORE` — проигнорировать конфликт
- `FAIL` — транзакция завершается с ошибкой

## 3. Database — база данных

```kotlin
@Database(
    entities = [User::class, Order::class],
    version = 1,
    exportSchema = false
)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
    abstract fun orderDao(): OrderDao
}
```

### Создание экземпляра (Singleton)

```kotlin
// Через Application класс
class MyApp : Application() {
    val database: AppDatabase by lazy {
        Room.databaseBuilder(
            applicationContext,
            AppDatabase::class.java,
            "my_database"
        )
        // .fallbackToDestructiveMigration() // Удалить данные при смене версии
        .build()
    }
}
```

## Использование в ViewModel

```kotlin
class UserViewModel(application: Application) : AndroidViewModel(application) {
    private val dao = (application as MyApp).database.userDao()

    val users = dao.getAllFlow().stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5000),
        initialValue = emptyList()
    )

    fun addUser(name: String, age: Int) {
        viewModelScope.launch(Dispatchers.IO) {
            dao.insert(User(firstName = name, lastName = "", age = age, email = ""))
        }
    }

    fun deleteUser(id: Int) {
        viewModelScope.launch(Dispatchers.IO) {
            dao.deleteById(id)
        }
    }
}
```

## Использование в Compose

```kotlin
@Composable
fun UserScreen(viewModel: UserViewModel = viewModel()) {
    val users by viewModel.users.collectAsStateWithLifecycle()

    Column {
        LazyColumn {
            items(users) { user ->
                Text("${user.firstName}, возраст: ${user.age}")
            }
        }

        Button(onClick = {
            viewModel.addUser("Новый пользователь", 25)
        }) {
            Text("Добавить")
        }
    }
}
```

## Миграции

```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(db: SupportSQLiteDatabase) {
        db.execSQL("ALTER TABLE users ADD COLUMN phone TEXT DEFAULT ''")
    }
}

Room.databaseBuilder(context, AppDatabase::class.java, "db")
    .addMigrations(MIGRATION_1_2)
    .build()
```

## TypeConverter (для сложных типов)

```kotlin
class Converters {
    @TypeConverter
    fun fromTimestamp(value: Long?): Date? = value?.let { Date(it) }

    @TypeConverter
    fun dateToTimestamp(date: Date?): Long? = date?.time
}

@TypeConverters(Converters::class)
abstract class AppDatabase : RoomDatabase() { ... }
```

## Важные моменты

- Все операции с БД должны быть **в фоновом потоке** (suspend функции, Dispatchers.IO)
- Room **не позволяет** выполнять БД-операции в главном потоке (выбросит исключение)
- Используйте `Flow` для автоматического обновления UI при изменении данных
- `@PrimaryKey(autoGenerate = true)` — автоинкремент ID
- `@Entity` = таблица, `@Dao` = запросы, `@Database` = точка входа
- Версия БД (`version`) увеличивается при изменении схемы — нужен `Migration` или `fallbackToDestructiveMigration`
