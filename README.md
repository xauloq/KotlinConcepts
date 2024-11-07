# KotlinConcepts

Room Database in Kotlin
Overview
Room provides an abstraction layer over SQLite to facilitate database access while maintaining the full capabilities of SQLite. It simplifies database operations by reducing boilerplate code and provides a modern API for database management in Kotlin.

Main Components of Room
Entity: Represents a table within the database.
DAO (Data Access Object): Defines the methods for accessing the database.
Database: The main database class that provides the RoomDatabase instance and connects the DAO and Entity.
Implementation Guide
1. Creating the Entity (User.kt)
The Entity class represents a table structure within the database.

kotlin
Copy code
@Entity(tableName = "user_table")
data class User(
    @PrimaryKey(autoGenerate = true) val id: Int,
    val name: String,
    val age: Int
)
@Entity: Indicates that this class represents a table in the database.
@PrimaryKey(autoGenerate = true): Specifies the primary key field with auto-increment enabled.
2. Defining the DAO (UserDao.kt)
The DAO interface contains methods for database operations.

kotlin
Copy code
@Dao
interface UserDao {
    @Insert(onConflict = OnConflictStrategy.IGNORE)
    suspend fun addUser(user: User)

    @Query("SELECT * FROM user_table ORDER BY id ASC")
    fun readAllData(): LiveData<List<User>>
}
@Dao: Marks this interface as a Data Access Object.
@Insert: Defines a method to insert a new record with conflict resolution strategy set to IGNORE.
@Query: Allows custom SQL queries, such as selecting all records and ordering them by id.
3. Creating the Database Class (UserDatabase.kt)
The UserDatabase class serves as the main access point to the database.

kotlin
Copy code
@Database(entities = [User::class], version = 1, exportSchema = false)
abstract class UserDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao

    companion object {
        @Volatile
        private var INSTANCE: UserDatabase? = null

        fun getDatabase(context: Context): UserDatabase {
            val tempInstance = INSTANCE
            if (tempInstance != null) {
                return tempInstance
            }
            synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    UserDatabase::class.java,
                    "user_database"
                ).build()
                INSTANCE = instance
                return instance
            }
        }
    }
}
@Database: Annotates this class as the Room Database. The entities array lists the tables used in the database.
@Volatile: Ensures the value of INSTANCE is always up-to-date and visible to all threads.
synchronized block: Prevents multiple threads from creating multiple instances of the database.
Example Usage
To use the UserDatabase, call getDatabase(context) to obtain an instance and use the userDao() method for accessing the database operations.

kotlin
Copy code
val userDatabase = UserDatabase.getDatabase(context)
val userDao = userDatabase.userDao()

// Example usage
CoroutineScope(Dispatchers.IO).launch {
    userDao.addUser(User(0, "John Doe", 25))
}
