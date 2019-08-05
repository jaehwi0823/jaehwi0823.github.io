---
layout: article
title: Android - Giới thiệu Room Persistence Library
tags: [android, java]
---
## 1. Giới thiệu
### 1.1. Room database là gì?
Room là một Persistence Library được google giới thiệu trong sự kiện google I/O mới đây, nó là một abstract layer cung cấp cách thức truy câp thao tác với dữ liệu trong cơ sở dữ liệu SQLite. Bình thường để tạo được database bạn cần viết các câu lệnh rất dài, mà viết sai một chút thôi là lại phải ngồi rà soát lại ngay.
```
public class DatabaseHelper extends SQLiteOpenHelper {

    public static final int DATABASE_VERSION = 1;
    public static final String DATABASE_NAME = "Contact.db";
    private static final String SQL_CREATE_CONTACTS = "CREATE TABLE "
            + ContactContract.ContactEntry.TABLE_NAME
            + " ("
            + ContactContract.ContactEntry._ID
            + " INTEGER PRIMARY KEY,"
            + ContactContract.ContactEntry.COLUMN_NAME
            + " TEXT,"
            + ContactContract.ContactEntry.COLUMN_PHONE_NUMBER
            + " TEXT,"
            + ContactContract.ContactEntry.COLUMN_ADDRESS
            + " TEXT)";

    private static final String SQL_DELETE_CONTACTS =
            "DROP TABLE IF EXISTS " + ContactContract.ContactEntry.TABLE_NAME;

    public DatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase) {
        sqLiteDatabase.execSQL(SQL_CREATE_CONTACTS);
    }

    @Override
    public void onUpgrade(SQLiteDatabase sqLiteDatabase, int i, int i1) {
        sqLiteDatabase.execSQL(SQL_DELETE_CONTACTS);
        sqLiteDatabase.execSQL(SQL_CREATE_CONTACTS);
    }
}
```
Còn với Room thì sao nhỉ?
```
@Entity(tableName = "users")
public class User {
    private static final String DEFAULT_PW = "12345678";
    @NonNull
    @PrimaryKey(autoGenerate = true)
    @ColumnInfo(name = "id")
    private int mId;
    @ColumnInfo(name = "first_name")
    private String mFirstName;
    @ColumnInfo(name = "last_name")
    private String mLastName;
    @ColumnInfo(name = "password")
    private String mPassword;
    @Embedded
    private Place mPlace
 }
```
Các ứng dụng sử dụng một lượng lớn dữ liệu có cấu trúc có thể hưởng lợi lớn từ việc lưu lại dữ liệu trên local thông qua Room Database. Trường hợp thường gặp nhất là chỉ cache những dữ liệu có liên quan. Nếu làm vậy thì khi thiết bị không có kết nối internet thì user vẫn có thể truy cập data đấy khi đang offline. Mọi dữ liệu được phát sinh hay thay đổi do user sau đó sẽ được đồng bộ với server khi họ online trở lại.
### 1.2. Đặc điểm của Room database
Framework chính (Sqlite Database) cung cấp các built-in support cho các trường hợp làm việc với các nội dung SQL thô. Mặc dù các API này khá mạnh mẽ nhưng chúng lại tương đối low-level và yêu cầu khá nhiều thời gian và nỗ lực để sử dụng:
- Không có xác thực các câu truy vấn SQL ở thời điểm compile-time. Khi data graph thay đổi thì dev sẽ phải cập nhật lại các câu truy vấn SQL thủ công. Việc này khá mất thời gian và xác suất gặp lỗi trong quá trình khá lớn.
- Sẽ phải dùng nhiều code khung để chuyển đổi giữa truy vấn SQL với các Java data object (Phần này chắc ai làm việc với DB nhiều chắc chắn hiểu rõ)
### 1.3. Cách sử dụng Room database
**1. Mở build.gradle (Root project) và thêm vào dòng lệnh sau**
```
allprojects {
    repositories {
        jcenter()
        maven { url 'https://maven.google.com' }
    }
}
```
**2. Mở build.gradle (app) và thêm dòng lệnh sau trong dependencies**
```
implementation "android.arch.persistence.room:runtime:1.0.0-beta2"
annotationProcessor "android.arch.persistence.room:compiler:1.0.0-beta2"
```
### 1.4. Các thành phần chính trong Room
![Thành phần chính trong Room](/assets/images/a7afdaea-db14-45a2-b0c4-7f7917e86edc.png)

Có 3 thành phần chính trong Room:
- **Database**
Có thể dùng componenet này để tạo database holder. Annotation sẽ cung cấp danh sách các thực thể và nội dung class sẽ định nghĩa danh sách các DAO (đối tượng truy cập CSDL) của CSDL. Nó cũng là điểm truy cập chính cho các kết nối phía dưới. Annotated class nên để là lớp abstract extends RoomDatabase. Tại thời điểm runtime thì dev có thể nhận được một instance của nó bằng cách gọi Room.databaseBuilder() hoặc Room.inMemoryDatabaseBuilder().
```
@Database(entities = {User.class}, version = DATABASE_VERSION)
public abstract class UserDatabase extends RoomDatabase{
    private static UserDatabase sUserDatabase;

    public static final int DATABASE_VERSION = 2;
    public static final String DATABASE_NAME = "Room-database";

    public abstract UserDAO userDAO();

    public static UserDatabase getInstance(Context context) {
        if (sUserDatabase == null) {
            sUserDatabase = Room.databaseBuilder(context, UserDatabase.class, DATABASE_NAME)
                .fallbackToDestructiveMigration()
                .build();
        }
        return sUserDatabase;
    }
}
```
- **DAO (Data Access Objects)**
Đây là component đại diện cho lớp hoặc interface như một đối tượng truy cập dữ liệu (DAO). DAO là thành phần chính của Room là chịu trách nhiệm trong việc định nghĩa các phương thức truy cập CSDL. Các lớp được chú thích với @Database phải chứa một phương thức trừu tượng có số lượng đối số truyền vào là 0 và đối tượng trả về là đối tượng của lớp được chú thích bởi @Dao. Khi code được sinh ra ở thời điểm biên dịch thì Room sẽ tạo một implementation của class này. •	Note: Bằng cách truy cập database sử dụng lớp DAO thay vì query builder hoặc queries trực tiếp thì dev có thể cô lập các thành phần khác nhau của kiến trúc database. Hơn nữa, DAO cho phép dev dễ dàng mock truy cập database khi dev test app.
```
@Dao
public interface UserDAO {
    @Query("SELECT * FROM users WHERE id = :userId")
    Flowable<User> getUserByUserId(int userId);

    @Query("SELECT * FROM users WHERE first_name LIKE :userName OR last_name LIKE :userName")
    Flowable<List<User>> getUserByName(String userName);

    @Query("SELECT * FROM users")
    Flowable<List<User>> getALlUser();

    @Insert
    void insertUser(User... users);

    @Delete
    void deleteUser(User user);

    @Query("DELETE FROM users")
    void deleteAllUser();

    @Update
    void updateUser(User... users);
}
```
- **Entity**

Component này đại diện cho một class chứa một row của database. Với mỗi một entity thì một database table sẽ được tạo để giữ các items tương ứng. Nên tham chiếu lớp enity thông qua mảng entities trong class Database. Mỗi một trường của enitty sẽ được pesist trong database trừ trường hợp bị chú thích là @Ignore.

Note: Các entity có thể hoặc là có hàm khởi tạo rỗng (trường hợp lớp DAO có thể truy cập từng field đã persist) hoặc là hàm khởi tạo với các đối số là các kiểu dữ liệu và tên khớp với một trong các field của entity. Room còn có thể sử dụng hàm khởi tạo đầy đủ hoặc một phần, ví dụ như hàm khởi tạo chỉ nhận một trong các field.

Với mỗi Object được định nghĩa với anotation @Entity Room sẽ tạo một table cho đối tượng này trong database với name là tableName được chú thích. Room sẽ tạo các cột tương ứng với số field được khai báo trong object. Nếu không muốn lưu trữ bạn có thể sử dụng anotation @Ignore Bạn có thể custom lại tên của cột thông qua anotaion @ColumnInfo(name = "column name").

**Primary key**

Mỗi Object phải xác định ít nhất 1 trường làm khóa chính. Ngay cả khi chỉ có 1 trường, bạn vẫn cần chú thích trường này bằng anotation @PrimaryKey. Ngoài ra, nếu bạn muốn Room gán ID tự động cho các thực thể, bạn có thể đặt thuộc tính autoGenerate của @ PrimaryKey.(Trường hợp thuộc tính là int, long)
```
@PrimaryKey(autoGenerate = true)
private int mId;
```
**Indices and uniqueness**
Trường hợp các bạn muốn đánh index cho một số trường trong database để tăng tốc độ truy vấn các bạn có thể sử dụng như sau:
```
@Entity(indices = {@Index(value = {"first_name", "last_name"}})
```
Một số trường hợp các bạn có thể muốn một số trường là duy nhất trong db ví dụ first_name và last_name không thể có bản ghi nào trùng nhau các bạn có thể thêm unique như sau:
```
@Entity(indices = {@Index(value = {"first_name", "last_name"},
       unique = true)})
```
**Nested objects**

Trong một số trường hợp các bạn tạo ra object với các nested object, các bạn không có nhu cầu lưu chúng thành 1 bảng riêng mà đơn giản chỉ giống như 1 column bình thường các bạn có thể sử dụng anotaion @Embedded cho chúng giống như mình đã làm cho Place trong object User
```
 @Embedded
    private Place mPlace;
```

**Relationships between objects**

Định nghĩa foreignKeys Ví dụ bạn có đối tượng khác là Pet.java và bạn có thể định nghĩa relationship tới đối tượng User.java thông qua @ForeignKey annotation như sau
```
@Entity(foreignKeys = @ForeignKey(entity = User.class,
                                  parentColumns = "id",
                                  childColumns = "user_id"))
class Pet {
    @PrimaryKey
    public int petId;

    public String name;

    @ColumnInfo(name = "user_id")
    public int userId;
}
```
