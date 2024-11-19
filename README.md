# Codelab de Room en Android

## 1. Introducción
   - Qué es **Room** y sus ventajas.
   - Por qué utilizar Room sobre SQLite.
   - Componentes clave de Room.

## 2. Preparativos Iniciales
   - Configuración del proyecto en Android Studio.
   - Añadir las dependencias necesarias en el archivo `build.gradle`:
     ```gradle
     dependencies {
         implementation "androidx.room:room-runtime:2.5.0"
         kapt "androidx.room:room-compiler:2.5.0"
     }
     ```
   - Para Kotlin, asegúrate de habilitar `kapt`:
     ```gradle
     apply plugin: 'kotlin-kapt'
     ```

## 3. Creación de la Entidad
   - Definición de la clase de entidad con anotaciones de Room.
   - Anotaciones más importantes: `@Entity`, `@PrimaryKey`, `@ColumnInfo`.
   - Ejemplo de la entidad `User`:
     ```kotlin
     @Entity(tableName = "user_table")
     data class User(
         @PrimaryKey(autoGenerate = true) val id: Int,
         @ColumnInfo(name = "first_name") val firstName: String,
         @ColumnInfo(name = "last_name") val lastName: String
     )
     ```

## 4. Creación del DAO (Data Access Object)
   - Definir las funciones de acceso a los datos con Room.
   - Anotaciones clave: `@Dao`, `@Insert`, `@Query`.
   - Ejemplo de DAO:
     ```kotlin
     @Dao
     interface UserDao {
         @Insert
         suspend fun insert(user: User)

         @Query("SELECT * FROM user_table")
         fun getAllUsers(): LiveData<List<User>>
     }
     ```

## 5. Creación de la Base de Datos
   - Crear una clase abstracta para la base de datos, heredada de `RoomDatabase`.
   - Usar la anotación `@Database` para definir la versión y las entidades.
   - Ejemplo de la base de datos:
     ```kotlin
     @Database(entities = [User::class], version = 1, exportSchema = false)
     abstract class UserDatabase : RoomDatabase() {
         abstract fun userDao(): UserDao
     }
     ```

## 6. Inicialización de la Base de Datos
   - Instanciar la base de datos en un objeto singleton para evitar múltiples instancias.
   - Ejemplo de inicialización:
     ```kotlin
     @Volatile
     private var INSTANCE: UserDatabase? = null

     fun getDatabase(context: Context): UserDatabase {
         return INSTANCE ?: synchronized(this) {
             val instance = Room.databaseBuilder(
                 context.applicationContext,
                 UserDatabase::class.java,
                 "user_database"
             ).build()
             INSTANCE = instance
             instance
         }
     }
     ```

## 7. Uso de Room en la UI
   - Cómo acceder a la base de datos desde actividades o fragmentos.
   - Mostrar los resultados usando `LiveData` y `Observer`.
   - Ejemplo de uso en una `Activity`:
     ```kotlin
     val userDao = UserDatabase.getDatabase(applicationContext).userDao()
     userDao.getAllUsers().observe(this, Observer { users ->
         // Actualizar UI con la lista de usuarios
     })
     ```

## 8. Operaciones en la Base de Datos
   - Realizar inserciones, actualizaciones, eliminaciones y consultas de datos.
   - Ejemplo de inserción de un usuario:
     ```kotlin
     val user = User(0, "John", "Doe")
     userDao.insert(user)
     ```
