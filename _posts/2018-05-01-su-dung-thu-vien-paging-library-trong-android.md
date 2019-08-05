---
layout: article
title: Sử dụng thư viện Paging Library trong Android
tags: [android, java]
---

### 1. Giới thiệu
Trong thực tế khi lập trình ứng dụng ta sẽ gặp trường hợp phải tải rất nhiều dữ liệu từ database về và khi đó ta thường sử dụng kỹ thuật phân trang. Nhưng thử tưởng tượng cơ sở dữ liệu cả bạn có khối lượng dữ liệu khổng lồ như youtube thì bạn sẽ phải làm như thế nào nếu truy vấn lên database thì sẽ mất rất nhiều thời gian và tốn bộ nhớ chính vì vấn đề này mà Google đã cho ra mắt thư viên Paging library để giải quyết vấn đề này.

Sau đây là một số khái niệm cơ bản của thư viên này.

#### 1.1 Main Component of Paging Library
Main component of Paging Library là PagedListAdapter được kết thừa từ RecyclerViewAdapter, PagedList và DataSource.

![Sơ đồ thư viện Paging Library](/assets/images/37bb0383-093f-4363-a38f-f9701356f116.png)
#### 1.2. Datasource
Datasource là một interface cho page source để cung cấp dữ liệu. Khi sử dụng chúng ta phải implement 1 trong 2 loại datasource được cung cấp là DataSource và TiledDataSource nó sẽ được sử dụng khi ta load dữ liệu từ datasource.
- Sử dụng KeyedDataSource nếu bạn cần dùng N data để lấy N-1 data. Ví dụ khi bạn muốn lấy được danh sách comment của facebook thì bạn cần biết được id của bình luận N để lấy được id của bình luận N+1 và cứ như thế.
- Sử dụng TiledDataSource khi bạn muốn load từ bất kỷ vị trí nào bạn muốn trong datasource của bạn ví dụ bạn muốn lấy danh sách sản phẩm từ sản phẩm vị trí 50 trong danh sách 1000 sản phẩm.

- LoadCount(): Đây là phương thức cho bạn biết số lượng hữu hạn hay vô hạn item được hiển thị trên danh sách của bạn.

#### 1.3. PagedList
PagedList là một thành phần giúp tự động load data và phát tín hiệu để cập nhật lại data trên RecyclerViewAdapter. Dữ liệu sẽ được tự động load trên background theard và được sử dụng trên main theard. Nó hỗ trợ cả danh sách hữu hạn và danh sách vô hạn. PageList cho phép bạn thiết lập một số cấu hình như kích thước của danh sách đó và số item được load mỗi lần cuộn của list đó.

#### 1.4. PagedListAdapter
PagedListAdapter từ lớp RecyclerView.Adapter đại diện cho PageList. Cơ chế hoạt động là khi dữ liệu được load thì PagedListAdapter sẽ báo hiệu cho RecyclerView là data đã được tải xuống lúc này PagedListAdapter sẽ được chạy trong background để tính toán những thay đổi cần thiết từ PageList và RecyclerView sẽ cập nhật lại những thay đổi đã được tính toán.

### 2. Sử dụng Paging Library
![Cách hoạt động thư viện Paging library](/assets/images/d2690e37-6901-4c2b-9abe-2e659144e3fb.gif)
#### 2.1. Thêm thư viện cần thiết

Add Component Architecture to your project Open build.gradle trong project và add lines bên dưới.

```
allprojects {
    repositories {
        jcenter()
        maven { url 'https://maven.google.com' }
    }
}
```

Thêm đoạn sau vào build.gradle để thêm các thư viện cần thiết.

```
dependencies {
    ....

    //For Lifecycles, LiveData, and ViewModel
    implementation 'android.arch.lifecycle:runtime:1.1.1'
    implementation 'android.arch.lifecycle:extensions:1.1.1'
    annotationProcessor 'android.arch.lifecycle:compiler:1.1.1'

    //For Room
    implementation 'android.arch.persistence.room:runtime:1.1.1'
    annotationProcessor "android.arch.persistence.room:compiler:1.1.1"

    //For Paging
    implementation 'android.arch.paging:runtime:1.0.1'
}
```
#### 2.2. Tạo datasource
Tạo một lớp đối tượng
```
@Entity
public class User {
    @PrimaryKey(autoGenerate = true)
    @ColumnInfo(name = "user_id")
    public long userId;
    @ColumnInfo(name = "first_name")
    public String firstName;
    public String address;
}
```
#### 2.3. Data Access Objects (DAO)
Để đơn giản trong connection giữa Database và RecyclerView, chúng ta sử dụng LivePagedListProvider.
```
@Dao
public interface UserDao {
 
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    public void insertAll(List<User> users);
 
    @Query("SELECT * FROM User")
    public abstract LivePagedListProvider<Integer,User> usersByFirstName();
}
```
Tạo Database trong ví dụ này mình sử dung sqlite.
```
@Database(entities = {User.class}, version = 1)
abstract public class AppDatabase extends RoomDatabase {
    public static final String DATABASE_NAME = "UserDb";
 
    public abstract UserDao userDao();
}
```
#### 2.4. Tạo các thành phần của Adapter
Create ViewModel ViewModel này sẽ extends từ ViewModel của Architecture Component và sử dụng chúng để tham chiếu tới LiveData của PagedList. Chúng ta sẽ lấy tham chiếu đó từ DAO bằng cách gọi method getUsers(). Config cấu hình mà bạn muốn. Ví dụ: Size của page là 50, và size của mối lần load data là 50 item.
```
public class UserViewModel extends ViewModel {
 
    public LiveData<PagedList<User>> userList;
 
    public UserViewModel() {
 
    }
 
    public void init(UserDao userDao) {
        userList = userDao.usersByFirstName().create(0,
                new PagedList.Config.Builder()
                        .setEnablePlaceholders(true)
                        .setPageSize(50)
                        .setPrefetchDistance(50)
                        .build());
    }
}
```
Tạo một Adapter để thông báo cho PagedListAdapter biết sự khác nhau giữa hai phần tử implement lớp DiffCallback.
```
@Entity
public class User {
    public static DiffCallback<User> DIFF_CALLBACK = new DiffCallback<User>() {
        @Override
        public boolean areItemsTheSame(@NonNull User oldItem, @NonNull User newItem) {
            return oldItem.userId == newItem.userId;
        }
 
        @Override
        public boolean areContentsTheSame(@NonNull User oldItem, @NonNull User newItem) {
            return oldItem.equals(newItem);
        }
    };
 
    @Override
    public boolean equals(Object obj) {
        if (obj == this)
            return true;
 
        User user = (User) obj;
 
        return user.userId == this.userId && user.firstName == this.firstName;
    }
}
```
Tạo một PageListAdapter
```
public class UserAdapter extends PagedListAdapter<User, UserAdapter.UserItemViewHolder> {
 
    protected UserAdapter() {
        super(User.DIFF_CALLBACK);
    }
 
    @Override
    public UserItemViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        LayoutInflater layoutInflater = LayoutInflater.from(parent.getContext());
        View view = layoutInflater.inflate(R.layout.item_user_list, parent, false);
        return new UserItemViewHolder(view);
    }
 
    @Override
    public void onBindViewHolder(UserItemViewHolder holder, int position) {
        User user= getItem(position);
        if(user!=null) {
            holder.bindTo(user);
        }
    }
 
}
```
Và trong main Activity chúng ta sẽ viết như thế này.
```
RecyclerView recyclerView = findViewById(R.id.userList);
LinearLayoutManager llm = new LinearLayoutManager(this);
llm.setOrientation(LinearLayoutManager.VERTICAL);
recyclerView.setLayoutManager(llm);
 
 UserViewModel viewModel = ViewModelProviders.of(this).get(UserViewModel.class);
 viewModel.init(userDao);
 final UserAdapter userUserAdapter = new UserAdapter();
 
 viewModel.userList.observe(this, pagedList -> {
        userUserAdapter.setList(pagedList);
 });
 
 recyclerView.setAdapter(userUserAdapter);
 ```