# Activity

## 初始

### activity

1.想要在 AndroidManifest 中为 Activity 注册启动方式,分为两种:登录启动和默认启动

- 登录启动:即在程序开始时就启动该 Activity

```xml
<activity
    android:name=".MainActivity
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```

- 默认启动:需要借助 Intent 启动 Activity

```xml
<activity
    android:name=".MainActivity
    android:exported="true">
    <intent-filter>
        <action android:name="com.example.activitytest.ACION_STSRT"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="com.example.activitytest.MY_CATEGORY"/>
    </intent-filter>
</activity>
```

- Intent 有两种启动方式:显式启动和隐式启动  
  显式启动需要提供启动 Activity 的上下文 Context 和需要启动的目标 Activity,然后通过 startActivity(intent)函数启动

```kotlin
val intent = Intent(this,SecondActivity::class.java)
startActivity(intent)
```

隐式启动需要额外加入 category,提前在 xml 的 category 标签加入,android.intent.category.DEFAULT,为默认 category,可以不加

```kotlin
val intent = Intent("com.example.activitytest.DEFAULT")
intent.addCategory("com.example.activitytest.MY_CATEGORY")
startActivity(intent)
```

2.视图绑定
如需设置绑定类的实例以供 Activity 使用，请在 Activity 的 onCreate() 方法中执行以下步骤：

- 调用生成的绑定类中包含的静态 inflate() 方法。此操作会创建该绑定类的实例以供 Activity 使用。
- 通过调用 getRoot() 方法或使用 Kotlin 属性语法获取对根视图的引用。
- 将根视图传递到 setContentView()，使其成为屏幕上的活动视图。

* 由于 inflate(layoutInfalter)属于弱绑定,所以要 setContentView(binding.root)进行布局显示

```kotlin
private lateinit var binding:ActivityTestBinding
    override fun onCreate(saveInstanceState: Bundle?){
        super.onCreate(saveInstanceState)
        if(!::binding.isInitialized){
            binding = ActivityTestBinding.inflate(layoutInflater)
        }
        val view = binding.root
        setContentView(view)
    }
```

### Fragment

1.视图绑定

- 如需设置绑定类的实例以供 Fragment 使用，请在 Fragment 的 onCreateView() 方法中执行以下步骤：

- 调用生成的绑定类中包含的静态 inflate() 方法。此操作会创建该绑定类的实例以供 Fragment 使用。

- 通过调用 getRoot() 方法或使用 Kotlin 属性语法获取对根视图的引用。
  从 onCreateView() 方法返回根视图，使其成为屏幕上的活动视图。

```kotlin
 private lateinit var binding: ResultProfileBinding
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        if(!::binding.isInitialized){
            binding = ResultProfileBinding.inflate(inflater, container, false)
        }
        val view = binding.root
        return view
    }
```

2.动态加载

- 现在 xml 文件里面准备好 FrameLayout 布局作为容器

```xml
<FrameLayout>
    android:id=".."
    android:layout_width="0dp"
    androuid:layout_height="match_parent"
    android:layout_weight="1">
</FrameLayout>
```

- 创建待添加 Fragment 实例
- 获取 FragmentManager,在 Activity 中调用 getSupportFragmentManager()获取
- 开启一个事务,通过 beginTransaction()方法开启
- 想容器类添加或替换 Fragment,使用 repalce()方法实现,需要传入容器 id 和待添加的 Fragment 实例
- 提交事务,调用 commit()方法完成

```kotlin
fun replaceFragment(layoutId:Int,fragment:Fragment){
    val fragmentManger = supportFragmentManger
    val transaction = fragmentManger.beginTransaction()
    transaction.replace(layoutId,fragment)
    transaction.commit()
}
```

## 交互

1.Activity 和 Activity 之间的交互

- 向下传递,使用 putExtra()方法传递,第一个参数为健,第二个参数为值

```kotlin
val intent = Intent(this,SecondActivity::class.java)
intent.putExtra("extra_data",data)
startActivity(intent)
```

- 接收 intent 的类会调用父类的 getIntent()方法,调用 getStringExtra()方法获得字符串

```kotlin
val extraData = intent.getStringExtra("extra_data")
```

- 向下传递
- 首先要注册请求登陆器

```kotlin
private val requestDataLauncher = registerForActivityResult(ActivityResultContracts.StartActivityForResult()){result-> when(result.resultCode){RESULT_OK -> result.data?.getStringExtra( "data")}
```

- 然后借助登陆器启动 intent

```kotlin
val intent = Intent(this,SecondActivity::class.java)
requestDataLauncher.launch(intent)
```

- SecondActivity 则需要定义一个 Intent 当作容器来传递信息,然后用 setResult()方法传递处理结果和 intent

```kotlin
val intent = Intent()
intent.putExtra("data",data)
setResult(RESULT_OK,intent)
finish()
```

2. activity 和 fragment,fragment 和 fragment 之间的交互

- 都是通过类型的转换进行交互,fragment 的 getActivity()方法会自动获取与之关联的 activity

```kotlin
val mianAvctivity = activity as? MainActivity
```

- 也可以使用 requireContext()函数来避免空检测

```
val mainActivity = requireContext() as MainActivity
```

- 之后便可以利用 mainActivity 调用 Activity 的方法了
- 而 fragment 和 fragment 的交互也可以通过 Activity 作为媒介进行交互

## 控件

1. RecyclerView 的自定义适配器和 layoutManager

- 适配器,处理传入的列表,可作为内部类嵌入 Activity 中,onCreateViewHolder(),onBindViewHolder(),getItemCount()三个方法必须重写
- onCreateViewHolder() 负责视图绑定
- onBindViewHolder() 负责数据处理
- getItemCount() 得到 List 长度
- 其中 FruitItem 作为每一个子项,需要自己定义

* 注意不能使用全局 binding,要对每一个 item 都进行视图绑定

```
class FruitAdapter(val fruitList:List<Fruit>):RecyclerView.Adapter<FruitAdapter.ViewHolder>(){

    inner class ViewHolder(val binding:FruitItemBinding):RecyclerView.ViewHolder(binding.root)

    override fun onCreateViewHolder(parent:ViewGroup,viewType:Int){

        val  binding = FruitItemBinding.inflate(LayoutInflater.from(parent.context),parent,false)
            return ViewHolder(binding)

    }

    override fun onBindViewHolder(holder:ViewHolder,position:Int){
        val fruit = fruitList[position]
        // 进行视图的赋值,监听等等操作
        // holder.binding
    }

    override fun getItemCount() = fruitList.size

}
```

- layoutManager 布局管理器,指定 RecyclerView 的布局方式

```kotlin
val layoutManager = LinearLayoutManager(this)
recyclerView.layoutManager = layoutManager
```

- onAttach
- onAttach 方法是 Fragment 生命周期中的一个回调方法，在 Fragment 被附加到宿主 Activity 时调用。你可以在 onAttach 方法中获取宿主 Activity 的引用并进行一些初始化操作。

```
override fun onAttach(context: Context) {
        super.onAttach(context)

        // context is the reference to the hosting Activity
        // You can typecast it to your MainActivity or any other specific Activity if needed
        if (context is MainActivity) {
            val mainActivity = requireContext() as MainActivity
            // Now you can interact with MainActivity and call its methods
        }
    }
```

## 示例

- 将以<<第一行代码>>p225 新闻界面与上述知识进行结合

  1.首先是数据结构部分

- 构建新闻数据类 data News

```kotlin
data class News(val title:String,val content:String)
```

- 然后构建 RecyclerView 的内容,实现新闻标题的 fragment
- 第一步要构建 item.xml,充当 newsList 的子项视图,逻辑都在 Recyclerview 里面写,不需要自己构建一个类

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/newsTile"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:maxLines="1"
    android:ellipsize="end"
    android:textSize="18sp"
    android:paddingLeft="10dp"
    android:paddingRight="10dp"
    android:paddingTop="15dp"
    android:paddingBottom="15dp">

</TextView>
```

- 第二部则是构建 RecyclerView 的相关内容,包括 xml 布局文件和类
- xml 文件只用引入 RecyclerView 即可

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/newsTitleRecyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        />

</LinearLayout>
```

- 由于要作为 Fragment 在 activity 里面进行切换,所以要继承 Fragment 类
- Recycler 的 layoutManager 是在 activity 的基础上创建的,这里的 adapter 使用了内部类的构建方式,所有的逻辑项都写在了适配器的 onBindViewHolder 内,里面是 fragment 通过 activity 与 fragment 的交互方式,调用了 activity 的 replace 方法切换 fragment

```kotlin

class NewsTitleFragment:Fragment() {
    private lateinit var binding:NewsTitleFragmentBinding
    private var isTwoPane = false

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        if(!::binding.isInitialized){
            binding = NewsTitleFragmentBinding.inflate(inflater,container,false)
        }
        val layoutManager = LinearLayoutManager(activity)
        isTwoPane = activity?.findViewById<View>(R.id.newsContentLayout) != null
        binding.newsTitleRecyclerView.layoutManager = layoutManager
        val adapter = NewsAdapter(getNews())
        binding.newsTitleRecyclerView.adapter = adapter

        return binding.root
    }

    // 适配器
    inner class NewsAdapter(val newsList:List<News>): RecyclerView.Adapter<NewsAdapter.ViewHolder>() {
        inner class ViewHolder(val binding: NewsItemBinding) : RecyclerView.ViewHolder(binding.root)

        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
            val binding: NewsItemBinding =
                NewsItemBinding.inflate(LayoutInflater.from(parent.context), parent, false)
            val holder = ViewHolder(binding)
            return holder
        }

        override fun onBindViewHolder(holder: ViewHolder, position: Int) {
            val news = newsList[position]
            holder.binding.newsTile.text = news.title
            holder.itemView.setOnClickListener{
                val mainActivity = requireContext() as MainActivity
                val newsContentFragment = NewsContentFragment(news.title,news.content)

                if (isTwoPane){
                    // 双页
                    mainActivity.replaceFragment(R.id.newsContentLayout,newsContentFragment)

                }
                else{
                    // 单页
                    mainActivity.replaceFragment(R.id.mainFrag,newsContentFragment)
                }
            }
        }

        override fun getItemCount() = newsList.size

        // --------------------------------------------------------------------
    }
//
    private fun getNews():List<News>{
        val newsList = ArrayList<News>()
        for(i in 1..50){
            val news = News("This is news title $i",getRandomLengthString("This is news content $i"))
            newsList.add(news)
        }
        return newsList
    }

    private fun getRandomLengthString(str:String) = str * (1..20).random()
}

```

2. 构建 content 内容界面,让 RecyclerViewFragment 的 itemView 跳转到相应的 content 界面

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:id="@+id/contentLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:visibility="invisible">

        <TextView
            android:id="@+id/newsTitle"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:padding="10dp"
            android:textSize="20sp"
            />

        <View
            android:layout_width="match_parent"
            android:layout_height="1dp"
            android:background="#000"
            />

        <TextView
            android:id="@+id/newsContent"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1"
            android:padding="15dp"
            android:textSize="18sp"
            />

    </LinearLayout>
    <View
        android:layout_width="1dp"
        android:layout_height="match_parent"
        android:layout_alignParentLeft="true"
        android:background="#000"
        />

</RelativeLayout>
```

- 构造参数添加 title 和 content 方便传入参数

```kotlin
class NewsContentFragment(private val title: String, private val content: String):Fragment() {
    private lateinit var binding:NewsContentFragmentBinding
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        if(!::binding.isInitialized){
            binding = NewsContentFragmentBinding.inflate(inflater,container,false)
        }
        binding.contentLayout.visibility = View.VISIBLE
        binding.newsTitle.text = title
        binding.newsContent.text = content
        return binding.root
    }

}
```

3. 主 activity 的实现

- xml 里面使用 FrameLayout 作为容器,动态的加载 Fragment

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <FrameLayout
        android:id="@+id/mainFrag"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    </FrameLayout>


</FrameLayout>
```

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding:ActivityMainBinding
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        if(!::binding.isInitialized){
            binding = ActivityMainBinding.inflate(layoutInflater)
        }
        setContentView(binding.root)
        replaceFragment(R.id.mainFrag,NewsTitleFragment())

    }

    fun replaceFragment(id:Int,fragment: Fragment) {
        val fragmentManager = supportFragmentManager
        val transaction = fragmentManager.beginTransaction()
        transaction.replace(id,fragment)
        transaction.addToBackStack(null)
        transaction.commit()
    }
}
```

# Broadcast

- BroadcastReceiver 标准写法

```kotlin
class MyBoardcastReceiver(){
    override fun onCreate(context:Context,intent:Intent){
        // your function
    }
}
```

1. 动态注册

```kotlin
val intentFilter = IntentFilter()
        intentFilter.addAction("com.example.broadcastbestpractice.FORCE_OFFLINE")
        receiver = ForceOfflineReceiver()
        registerReceiver(receiver,intentFilter)
```

- 动态注册一定要在 onDestory()处取消注册

```kotlin
override fun onDestroy() {
    super.onDestroy()
    // 注销
    unregisterReceiver(timeChangeReceiver)
}
```

2. 静态注册

- 静态注册可以在程序未启动的情况下也能接收广播,需要在 AndroidMainfest.xml 注册

```xml
<receiver
    android:name=".MyBroadcastReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter android:priority="100">
        <action android:name="com.example.broadcasttest.MY_BROADCAST" />
    </intent-filter>
</receiver>
```

## 标准广播

- 异步广播,广播发出后 BroadcastReceicer 几乎在同一时刻收到广播信息,没有先后顺序,无法被截断

* 发送广播

- 使用 intent 和 sendBroadcast(intent)进行发送

```kotlin
val intent = Intent("com.example.broadcasttest.MY_BROADCAST")
intent.setPackage(packageName)
sendBroadcast(intent)
```

## 有序广播

- 设置优先级,可以截断

```xml
 <receiver
            android:name=".MyBroadcastReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter android:priority="100">
                <action android:name="com.example.broadcasttest.MY_BROADCAST" />
            </intent-filter>
        </receiver>
```

- 将 sendBroadcast 换成了 sendOrderedBroadcast

```kotlin
val intent = Intent("com.example.broadcasttest.MY_BROADCAST")
intent.setPackage(packageName)
sendOrderedBroadcast(intent,null)
```

```kotlin
// 截断广播
abortBroadcast()
```

## 示例

1.实现关闭所有 Activity 的功能

- Activity 管理器

```kotlin
object ActivityCollector {
    private val activities = ArrayList<Activity>()

    fun addActivity(activity: Activity){
        activities.add(activity)
    }

    fun removeActivity(activity: Activity){
        activities.remove(activity)
    }

    fun finishAll(){
        for (activity in activities){
            if(!activity.isFinishing){
                activity.finish()
            }
        }
        activities.clear()
    }
}
```

- BaseActivity 作为所有 Activity 的父类

```kotlin
open class BaseActivity:AppCompatActivity() {
    private lateinit var receiver:ForceOfflineReceiver
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ActivityCollector.addActivity(this)
    }

    override fun onDestroy() {
        super.onDestroy()
        ActivityCollector.removeActivity(this)
    }

}
```

2. 发送广播

```kotlin
 binding.forceOffline.setOnClickListener {
    val intent = Intent("com.example.broadcastbestpractice.FORCE_OFFLINE")
    sendBroadcast(intent)
}
```

3. 注册广播

- 当 Activity 在栈顶的时候进行动态注册,不在栈顶时进行取消注册,所以采用在 BaseActivity 动态注册的方式

```kotlin
override fun onResume() {
        super.onResume()
        val intentFilter = IntentFilter()
        intentFilter.addAction("com.example.broadcastbestpractice.FORCE_OFFLINE")
        receiver = ForceOfflineReceiver()
        registerReceiver(receiver,intentFilter)
    }

    override fun onPause() {
        super.onPause()
        unregisterReceiver(receiver)
    }
```

4. 接收广播

```kotlin
inner class ForceOfflineReceiver: BroadcastReceiver(){
        override fun onReceive(context: Context?, parent: Intent?) {
            Toast.makeText(context, "receiver the broadcast", Toast.LENGTH_SHORT).show()
            AlertDialog.Builder(context).apply{
                setTitle("Warning")
                setMessage("You are forced to be offline.Please try to login again.")
                setCancelable(false)
                setPositiveButton("OK"){
                        _,_,->
                    ActivityCollector.finishAll()
                    val i = Intent(context,LoginActivity::class.java)
                    context?.startActivity(i)
                }
                show()
            }
        }
```

# 持久性存储

## 内置的 SQLite 数据库

- 使用 SQLiteOpenHelper 类,需要指定类继承,并重写 onCreate()和 onUpgrade()方法
  MyDatabaseHelper(this,"BookStore.db",1) 三个参数为根,数据库名,版本号

```kotlin
class MyDatabaseHelper(val context:Context,name:String,version:Int):SQLiteOpenHelper(context,name,null,version) {

    private val createBook = "create table Book(" +
            "id integer primary key autoincrement," +
            "author text," +
            "price real," +
            "pages integer," +
            "name text)"

    override fun onCreate(db: SQLiteDatabase?) {

    }

    override fun onUpgrade(db: SQLiteDatabase?, p1: Int, p2: Int) {

    }

}

val dbHelper = MyDatabaseHelper(this,"BookStore.db",1)

```

### 创建表

execSQL(database)写在 onCreate 里在创建数据库的同时创建表

```kotlin
class MyDatabaseHelper(val context:Context,name:String,version:Int):SQLiteOpenHelper(context,name,null,version) {

    private val createBook = "create table Book(" +
            "id integer primary key autoincrement," +
            "author text," +
            "price real," +
            "pages integer," +
            "name text)"

    override fun onCreate(db: SQLiteDatabase?,oldVersion:Int,newVersion:Int) {
        db?.execSQL(createBook)
        Toast.makeText(context,"Create succeeded",Toast.LENGTH_SHORT).show()
    }

    ...
}

```

### CRUD 操作

- 均需要 ContentValues()作为容器

```kotlin
val values1 = ContentValues().apply {
                put("name","The Da Vinci Code")
                put("author","Dan Brown")
                put("pages",454)
                put("price",16.96)
            }
```

- 进阶  
  使用内置的封装函数

```kotlin
val values = contentValuesOf("name" to "The Da Vinci Code","author" to "Dan Brown")
```

- 并且借助 writeableDatabase 或者 readableDarabase 对象进行操作

```kotlin
val db = dbHelper.writableDatabase
// val db = dbHelper.readableDarabase
```

1.添加数据  
**接收三个参数:第一个为表名,第二个为空的默认值,第三个为 ContentValues 对象**

```kotlin
db.insert("Book",null,values1)
```

2.删除数据  
**接收三个参数:第一个为表名,第二第三为约束条件**

```kotlin
db.delete("Book","pages > ?", arrayOf("500"))
```

3.更新数据  
**接收四个参数:第一个为表名,第二个为 contentvalues 对象,第三第四为约束条件**

```kotlin
db.update("Book",values,"name = ?", arrayOf("The Da Vinci Code"))
```

4. 查询数据

|  query()参数  |      对应 SQL 部分       |               描述               |
| :-----------: | :----------------------: | :------------------------------: |
|     table     |     from table_name      |          指定查询的表名          |
|    columns    |  select column1,column2  |          指定查询的列名          |
|   selection   |   where column = value   |       指定 where 约束条件        |
| selectionArgs |                          | 为 where 中的占位符提供具体的值  |
|    groupBy    |     group by column      |      指定需要 group by 的列      |
|    having     |  having column = value   | 对 group by 后的结果进一步的约束 |
|    orderBy    | order by column1,column2 |      指定查询结果的排序方式      |

**最终会返回一个 cursor 对象,利用其进行取值操作**

```kotlin
val cursor = db.query("Book",null,null,null,null,null,null)
if(cursor.moveToFirst()){
    do {
        val name = cursor.getString(cursor.getColumnIndexOrThrow("name"))
        val author = cursor.getString(cursor.getColumnIndexOrThrow("author"))

    }while (cursor.moveToNext())
}
cursor.close()
}
```

# 共享数据

## 权限

- 权限声明

```xml
<uses-permission android:name="android.permission.READ_CONTACTS"/>
```

- 申请权限标准写法

```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        if(!::binding.isInitialized){
            binding = ActivityMainBinding.inflate(layoutInflater)
        }
        setContentView(binding.root)

        binding.makeCall.setOnClickListener {
            if (ContextCompat.checkSelfPermission(this,android.Manifest.permission.CALL_PHONE) != PackageManager.PERMISSION_GRANTED){
                ActivityCompat.requestPermissions(this, arrayOf(android.Manifest.permission.CALL_PHONE),1)
            }
            else{
                call()
            }
        }
    }

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        when(requestCode){
            1 -> {
                if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED){
                    call()
                }
                else{
                    Toast.makeText(this,"You denied the permission",Toast.LENGTH_SHORT).show()
                }
            }
        }
    }

    private fun call(){
        val intent = Intent(Intent.ACTION_CALL)
        intent.data = Uri.parse("tel:10086")
        startActivity(intent)
    }
```

## 共享数据

在获取了权限以后,便可以借助**_ContentResolver_**来访问**_ContentProvide_**的共享数据

- ContentResolver 不接收表名  
  标准命名格式

```kotlin
content://com.example.app.provider/table1
// 传换为Uri格式
val uri = Uri.parse("content://com.example.app.provider/table1")
```

### CRUD 操作

1. 查询

|  query()参数  |      对应 SQL 部分       |              描述               |
| :-----------: | :----------------------: | :-----------------------------: |
|      uri      |     from table_name      |  指定某个应用程序下的某一张表   |
|  projection   |  select column1,column2  |         指定查询的列名          |
|   selection   |   where column = value   |       指定 where 约束条件       |
| selectionArgs |                          | 为 where 中的占位符提供具体的值 |
|   sortOrder   | order by column1,column2 |     指定查询结果的排序条件      |

**由于返回的是 Cursor 对象,故可以使用 apply 高阶函数**

```kotlin
contentResolver.query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,null,null,null,null)?.apply {
    do{
            val name = getString(getColumnIndexOrThrow(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME))
            val num = getString(getColumnIndexOrThrow(ContactsContract.CommonDataKinds.Phone.NUMBER))

    } while(moveToNext())
    close()
}
```

2.插入

```kotlin
val values = contentValuesOf("column1" to "text")
contentResolver.insert(uri,values)
```

3. 更新

```kotlin
val values = contentValuesOf("column1" to "")
contentResolver.insert(uri,values,"column1 = ? and column2 = ?",arrayOf("text","1"))
```

4.删除

```kotlim
contentResolver.delete(uri,"column2 = ?",arrayOf("1"))
```

### 系统共享数据接口

- 使用系统 Uri:**_ContactsContract.CommonDataKinds.Phone.CONTENT_URI_**

```kotlin
contentResolver.query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,null,null,null,null)?.apply {
    while (moveToNext()) {
        val name = getString(getColumnIndexOrThrow(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME))
        val num = getString(getColumnIndexOrThrow(ContactsContract.CommonDataKinds.Phone.NUMBER))
        val contact = ContactData(name,num)
        contactList.add(contact)
    }
    close()
}
```

### 自定义的 ContentProvider 类

1. 标准写法,需要重写 6 个方法

```kotlin
class MyProvider:ContentProvider(){
    override fun onCreate():Boolean{
        return false
    }

    override fun query(uri:Uri,projection:Array<String>?,saelection:String?,selectionArgs:Array<String>?,sortOrder:String?):Cursor?{
        return null
    }

    override fun insert(uri:Uri,values:ContentValues?):Url??{
        return null
    }

    override fun update(uri:Uri,selection:String?,selectionArgs:Array<String>?):Int{
        return 0
    }

    override fun getType(uri:Uri):String?{
        return null
    }
}
```

2. 制定 URI

- 匹配任意表内容的 URI 格式

```kotlin
content://com.example.app.provider/*
```

- 匹配 table1 表中任意一行数据的内容的 URI 格式

```kotlin
content://com.example.app.provider/table1/#

```

3. 利用 uriMatch 进行 uri 解析

- addURI()方法将 authority,path 和自定义代码添加进去,当调用 match()方法时,返回某个能匹配这个 Uri 对象所对应的自定义代码,利用这个代码可以判断调用方期望访问的是哪张表中的数据

```kotlin
    private val bookDir = 0
    private val bookItem = 1
    private val categoryDIr = 2
    private val categoryItem = 3


    private val uriMathcher by lazy {
        val matcher = UriMatcher(UriMatcher.NO_MATCH)
        matcher.addURI(authority,"book",bookDir)
        matcher.addURI(authority,"book/#",bookItem)
        matcher.addURI(authority,"category",categoryDIr)
        matcher.addURI(authority,"category/#",categoryItem)
        matcher
    }
```

- 利用 match()方法实现各种功能

```kotlin


    override fun query(
        uri: Uri, projection: Array<String>?, selection: String?,
        selectionArgs: Array<String>?, sortOrder: String?
    ) :Cursotr? {
        val cursor = when (uriMathcher.match(uri)) {
            bookDir -> {

            }

            bookItem -> {

            }

            categoryDIr->{

            }

            categoryItem->{

            }


            else->null
        }
        cursor
    }

```

4.getType()方法

- 格式规定  
  1.必须以 vnd 开头  
  2.如果内容 URI 以路径结尾,后接 android.cursor.dir/,如果以 id 结尾,后接 android. cursor.item/  
  3.最后接上`vnd.<authority>.<path> `

```kotlin
override fun getType(uri:Uri) = when(uriMatcher.match(uri)){
                bookDir -> "vnd.android.cusor.dir/vnd.com.example.app.provider.tabler1"

            bookItem -> "vnd.android.cusor.item/vnd.com.example.app.provider.tabler1"

            categoryDIr->"vnd.android.cusor.dir/vnd.com.example.app.provider.tabler2"

            categoryItem->"vnd.android.cusor.item/vnd.com.example.app.provider.tabler2"


            else->null
}
```

5. 使用自定义 contentprivider 里面的 authority+表名作为 uri 传入 contentResolver 的方法中使用

## 示例

**要访问表中的某一个数据,需要用 uri.pathSegments[1]方法获取 id**

```kotlin
class DatabaseProvider : ContentProvider() {

    private val bookDir = 0
    private val bookItem = 1
    private val authority = "com.example.databasetest.provider"
    private var dbHelper:MyDatabaseHelper? = null

    private val uriMathcher by lazy {
        val matcher = UriMatcher(UriMatcher.NO_MATCH)
        matcher.addURI(authority,"book",bookDir)
        matcher.addURI(authority,"book/#",bookItem)
        matcher
    }

    override fun delete(uri: Uri, selection: String?, selectionArgs: Array<String>?) = dbHelper?.let {
        val db = it.writableDatabase
        val deletedRows = when(uriMathcher.match(uri)){
            bookDir->db.delete("Book",selection,selectionArgs)
            bookItem->{
                val bookId = uri.pathSegments[1]
                db.delete("Book","id=?", arrayOf(bookId))

            }
            else->0
        }
        deletedRows
    } ?: 0

    override fun getType(uri: Uri) = when(uriMathcher.match(uri)){
        bookDir->"vnd.android.cursor.dir/vnd.com.example.databasetest.provider.book"
        bookItem->"vnd.android.cursor.item/vnd.com.example.databasetest.provider.book"
        else->null
    }

    override fun insert(uri: Uri, values: ContentValues?) = dbHelper?.let {
        val db = it.writableDatabase
        val uriReturn = when(uriMathcher.match(uri)){
            bookDir,bookItem->{
                val newBookId = db.insert("Book",null,values)
                Uri.parse("content://$authority/book/&newBookId")
            }
            else->null

        }
        uriReturn
    }

    override fun onCreate() = context?.let {
        dbHelper = MyDatabaseHelper(it,"BookStore.db",2)
        true
    } ?: false

    override fun query(
        uri: Uri, projection: Array<String>?, selection: String?,
        selectionArgs: Array<String>?, sortOrder: String?
    ) = dbHelper?.let {
        val db = it.readableDatabase
        val cursor = when (uriMathcher.match(uri)) {
            bookDir -> db.query("Book", projection, selection, selectionArgs, null, null, sortOrder)
            bookItem->{
                val bookID = uri.pathSegments[1]
                db.query("Book",projection,"id=?", arrayOf(bookID),null,null,sortOrder)
            }
            else->null
        }
        cursor
    }

    override fun update(
        uri: Uri, values: ContentValues?, selection: String?,
        selectionArgs: Array<String>?
    ) = dbHelper?.let {
        val db = it.writableDatabase
        val updatedRows = when(uriMathcher.match(uri)){
            bookDir->db.update("Book",values,selection,selectionArgs)
            bookItem->{
                val bookId = uri.pathSegments[1]
                db.update("Book",values,"id=?", arrayOf(bookId))
            }
            else->null
        }
        updatedRows
    } ?: 0
}
```

# 多媒体

## 通知

1. 创建通知渠道

- NotificationChannel 接收三个参数:渠道 ID,渠道名称和重要级
- 重要级有:**_IMPORTANCEE_HIGH_**,**_IMPORTANCEE_DEFAULT_**,**_IMPORTANCEE_LOW_**,**_IMPORTANCEE_MIN_**

```kotlin
// 获得noticemanager
val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
// 新建channel
if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
    val channel = NotificationChannel("normal","Normal",NotificationManager.IMPORTANCE_DEFAULT)
    manager.createNotificationChannel(channel)
}
```

2. 创建和发送通知

- 通过**_NotificationCompat.Builder_**创建通知
- 通过**_manager.notify_**发送通知,

```kotlin
val notification = NotificationCompat.Builder(this,"normal").setContentTitle("This is content title").setContentText("This is content text").setSmallIcon(R.drawable.small_icon).setLargeIcon(BitmapFactory.decodeResource(resources,R.drawable.large_icon)).setAutoCancel(true).build()
manager.notify(1,notification)
```

3. 设置点击效果

- **_PendingIntent_**可选择**_getActivity_**方法,**_getBroadcast_**方法,**_getService_**方法
- 接收四个参数:第一个为 context,第二个为 0,第三个为 intent 对象,第四个确定行为,**_FLAG_ONE_SHOT_**,**_FLAG_NO_CREATE_**,**_FLAG_CANCEL_CURRENT_**和**_FLAG_UPDATE_CURRENT_**

```kotlin
val intent = Intent(this,NotificationActivity::class.java)
val pi = PendingIntent.getActivity(this,0,intent,PendingIntent.FLAG_UPDATE_CURRENT)
```

- 然后在 builder 里面添加 setContentIntent(pi)

```kotlin
val notification = NotificationCompat.Builder(this,"normal").setContentTitle("This is content title").setContentText("This is content text").setSmallIcon(R.drawable.small_icon).setLargeIcon(BitmapFactory.decodeResource(resources,R.drawable.large_icon)).setContentIntent(pi).setAutoCancel(true).build()
```

## 打开摄像头

- 注册器里面使用 BitmapFactory.decodeStream()方法将 output_image.jpg 解析成 bitmap 对象
- intent 使用**_android.media.action.IMAGE_CAPTURE_**,并 putExtra()填入**_MediaStore.EXTRA_OUTPUT,imageUri_**来指定保存路径
- 在安卓 7.0 之前,Uri 路径可以直接使用本地真实路径,而在 7.0 之后则需要用**_FileProvider.getUriForFile_**的方法来获取 Uri 路径,该方法接收三个参数,第一个为 Context 对象,第二个为字符串,第三个为 File 对象

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private lateinit var outputImage:File
    private lateinit var imageUri:Uri
    private val requestDataLauncher =
        registerForActivityResult(ActivityResultContracts.StartActivityForResult()){
                result ->
            if (result.resultCode == RESULT_OK) {
                    val bitmap = BitmapFactory.decodeStream(contentResolver.openInputStream(imageUri))
                    binding.imageView.setImageBitmap(rotateIfRequired(bitmap))
                }
            }


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        if(!::binding.isInitialized){
            binding = ActivityMainBinding.inflate(layoutInflater)
        }
        setContentView(binding.root)
        binding.takePhotoBtn.setOnClickListener {
            outputImage = File(externalCacheDir,"output_image.jpg")
            if(outputImage.exists()){
                outputImage.delete()
            }
            outputImage.createNewFile()
            imageUri = if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.N){
                FileProvider.getUriForFile(this,"com.example.cameraalbumtest.fileprovider",outputImage)
            }
            else{
                Uri.fromFile(outputImage)
            }

            //启动相机程序
            val intent = Intent("android.media.action.IMAGE_CAPTURE")
            intent.putExtra(MediaStore.EXTRA_OUTPUT,imageUri)
            requestDataLauncher.launch(intent)
        }
    }

    private fun rotateIfRequired(bitmap: Bitmap):Bitmap{
        val exif = ExifInterface(outputImage.path)
        val orientation = exif.getAttributeInt(ExifInterface.TAG_ORIENTATION,ExifInterface.ORIENTATION_NORMAL)
        return when(orientation){
            ExifInterface.ORIENTATION_ROTATE_90 -> rotateBitmap(bitmap,90)
            ExifInterface.ORIENTATION_ROTATE_180 -> rotateBitmap(bitmap,180)
            ExifInterface.ORIENTATION_ROTATE_270 -> rotateBitmap(bitmap,270)
            else -> bitmap
        }
    }

    private fun rotateBitmap(bitmap:Bitmap,degree:Int):Bitmap{
        val matrix = Matrix()
        matrix.postRotate(degree.toFloat())
        val rotatedBitmap = Bitmap.createBitmap(bitmap,0,0,bitmap.width,bitmap.height,matrix,true)
       bitmap.recycle()
        return rotatedBitmap
    }

}
```

- 因为要用到 fileprovider,所以要进行注册  
  **meta-data**指定了 Uri 共享路径需要创建\*\*

```xml
<provider
            android:authorities="com.example.cameraalbumtest.fileprovider"
            android:name="androidx.core.content.FileProvider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths"
                />
        </provider>
```

- path 为共享路径

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path
        name="my_images"
        path="/"
        />
</paths>
```

## 从相册选择图片

- 注册器中从 result.data?.data 里面获取 uri
- 注意文件选择器的隐式启动方式

```kotlin
private val fromAlbumLauncher =
        registerForActivityResult(ActivityResultContracts.StartActivityForResult()){
            if(it.resultCode == RESULT_OK){
                // 获取选择的图片的 URI
                val selectedImageUri = it.data?.data
                if(selectedImageUri != null){
                    val bitmap = getBitmapFromUri(selectedImageUri)
                    binding.imageView.setImageBitmap(bitmap)
                }
            }
        }

binding.fromAlbumBtn.setOnClickListener {
    // 打开文件选择器
    val intent = Intent(Intent.ACTION_OPEN_DOCUMENT)
    intent.addCategory(Intent.CATEGORY_OPENABLE)
    // 指定只显示图片
    intent.type = "image/*"
    fromAlbumLauncher.launch(intent)
}

private fun getBitmapFromUri(uri:Uri) = contentResolver.openFileDescriptor(uri,"r")?.use {
    BitmapFactory.decodeFileDescriptor(it.fileDescriptor)
}

```

## 播放多媒体文件

### 播放音频

1. 在 main 下创建 assets 目录,存放 mp3

2. 初始化 mediaPlayer

```kotlin
private fun initMediaPlayer(){
    val assetManager = assets
    val fd = assetManager.openFd("music.mp3")
    mediaPlayer.setDataSource(fd.fileDescriptor,fd.startOffset,fd.length)
    mediaPlayer.prepare()
}
```

3. 各种功能实现

```kotlin
binding.play.setOnClickListener {//播放音乐
    if(!mediaPlayer.isPlaying){
        mediaPlayer.start()
    }
}

binding.pause.setOnClickListener {//暂停音乐
    if(mediaPlayer.isPlaying){
        mediaPlayer.pause()
    }
}

binding.stop.setOnClickListener {//停止音乐
    if(mediaPlayer.isPlaying){
        mediaPlayer.reset()
        initMediaPlayer()
    }
}
```

4. 回收

```kotlin
override fun onDestroy() {
    super.onDestroy()
    mediaPlayer.start()
    mediaPlayer.release()
}
```

### 播放视频

1. 解析 uri

````kotlin
val uri = Uri.parse("android.resource://$packageName/${R.raw.video}")
binding.videoView.setVideoURI(uri)

2. 各种功能

```kotlin
binding.play.setOnClickListener {
    if (!binding.videoView.isPlaying){
        binding.videoView.start() // 开始播放
    }
}

binding.pause.setOnClickListener {
    if (!binding.videoView.isPlaying){
        binding.videoView.start() // 暂停播放
    }
}


binding.replay.setOnClickListener {
    if (!binding.videoView.isPlaying){
        binding.videoView.resume() // 重新播放
    }
}
````

3. 回收

```kotlin
override fun onDestroy() {
    super.onDestroy()
    binding.videoView.suspend()
}
```

# 多线程编程

## AsyncTask 异步消息处理机制

- 需要创建子类去继承 AsyncTask,需要指定 3 个泛型参数

  1. Params: 传入后台任务的参数
  2. Progress:进度单位
  3. Result:结果返回类型

- 还需要重写几个方法

  1. onPreExecute()  
     这个方法会在后台任务开始之前调用,用于进行一些界面上的初始化操作

  2. doInBackground(Params...)  
     执行耗时任务

  3. onProgressUpdate(Progress...)  
     当后台任务中调用了 publishProgress(Progress...)方法后,该方法会被调用,可以对 UI 进行操作

  4. onPostExecute(Result)  
     当后台任务执行完毕并通过 return 返回时,该方法被调用,返回的数据作为参数传递到此方法中,可以利用返回的数据进行一些 UI 操作

```kotlin
class DownloadTask:AsyncTask<Unit,Int,Boolean>(){
    override fun onPreExecute(){
        progressDialog.show() // 显示进度对话框
    }

    override fun doInBackground(vararg params:Unit?) = try{
        while(true){
            val downloadPercent = doDownload()
            publishProgress(downloadPercent)
            if(downloadPercent >= 100){
                break
            }
        }
    }catch(e:Exceeption){
        false
    }

    override fun onProgressUpdate(vararg values:Int?){
        // 更新下载进度
        progressDialog.setMessage("Downloaded ${values[0]}%")
    }

    override fun onPosExecurte(result:Boolean){
        progressDialog.dismiss() // 关闭进度对话框
        //提示下载效果
        if(result){
            Toast.makeText(context,"Download succeeded",Toast.LENGTH_SHORT).show()
        }
        else{
            Toast.makeText(context,"Download failed",Toast.LENGTH_SHORT).show()
        }
    }

}


// 启动任务
DownloadTask().execute()
```

## Service

### Service 基本用法

- 在项目中点击 New->Service 创建 Service,可自动完成注册
- onBind 方法是抽象方法,必须重写
- onCreate 在 Service 创建时调用,onStartCommand 在 Service 每次启动时调用,onDestory 在 Service 销毁时调用

```kotlin
class MyService : Service() {

    override fun onBind(intent: Intent): IBinder {
        TODO("Return the communication channel to the service.")
    }

    override fun onCreate() {
        super.onCreate()
    }


    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        return super.onStartCommand(intent, flags, startId)
    }


    override fun onDestroy() {
        super.onDestroy()
    }
}
```

- 利用 intent 启动和停止 Service

```kotlin
binding.startServiceBtn.setOnClickListener {
    val intent = Intent(this,MyService::class.java)
    startService(intent)
}

binding.stopServiceBtn.setOnClickListener {
    val intent = Intent(this,MyService::class.java)
    stopService(intent)
}
```

### Service 与 Activity 进行通信

1. 在自定义的 Service 类中定义 Bindear 类,并通过 onBind 方法返回

```kotlin
private val mBinder = DownloadBinder()

    inner class DownloadBinder : Binder(){
        fun startDownload(){
            Log.d("MyService","startDownload executed")
        }

        fun getProgress():Int{
            Log.d("<yService","getProgress executed")
            return 0
        }
    }

    override fun onBind(intent: Intent): IBinder {
        return mBinder
    }
```

2. 在 activity 中定义 ServiceConnection 类

- **_onServiceConnected_**方法在 Activity 和 Service 成功绑定时调用

```kotlin
private lateinit var downloadBinder:MyService.DownloadBinder
    private val connection = object : ServiceConnection{
        override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
            downloadBinder = service as MyService.DownloadBinder
            downloadBinder.startDownload()
            downloadBinder.getProgress()
        }

        override fun onServiceDisconnected(p0: ComponentName?) {

        }

    }
```

3. 通过 intent 进行绑定和取消绑定

- bindService 接收三个参数:第一个参数为 intent 对象,第二个为 ServiceConnection 实例,第三个为标志位,**_BIND_AUTO_CREATE_**表示在 Activity 和 Service 绑定后会自动启动 Service

```kotlin
binding.bindServiceBtn.setOnClickListener {
    val intent = Intent(this,MyService::class.java)
    bindService(intent,connection,Context.BIND_AUTO_CREATE) // 绑定Service
}

binding.unBindServiceBtn.setOnClickListener {
    unbindService(connection)
}
```

### 前台 Service

- 只需修改**_OnCreate()_**方法即可
- 注意调用了**_startForeground_**方法,接收两个参数,第一个为 id,第二个为 Notification 对象

```kotlin
override fun onCreate() {
    super.onCreate()
    // 获得noticemanager
    val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
    // 新建channel
    if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
        val channel = NotificationChannel("my_service","前台Service通知",NotificationManager.IMPORTANCE_DEFAULT)
        manager.createNotificationChannel(channel)
    }
    val intent = Intent(this,MainActivity::class.java)
    val pi = PendingIntent.getActivity(this,0,intent,PendingIntent.FLAG_UPDATE_CURRENT)
    val notification = NotificationCompat.Builder(this,"normal").setContentTitle("This is content title").setContentText("This is content text").setSmallIcon(R.drawable.small_icon).setLargeIcon(
        BitmapFactory.decodeResource(resources,R.drawable.large_icon)).setContentIntent(pi).setAutoCancel(true).build()
    startForeground(1,notification)
}
```

- 需要进行权限声明

```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>

```

### IntentService
* 将Service中的耗时任务装入到子线程中运行

```kotlin
class MyIntentService:IntentService("MyIntentService"){
    override fun onHandleIntent(intent:Intent){
        // 耗时任务
    }

    override fun onDestory(){
        super().onDestory()
    }
}
```