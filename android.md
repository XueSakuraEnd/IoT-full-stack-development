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
val intent = Intent(this,SecondActivity::class.java)
intent.addCategory("com.example.activitytest.MY_CATEGORY")
startActivity(intent)
```

2.视图绑定
如需设置绑定类的实例以供 Activity 使用，请在 Activity 的 onCreate() 方法中执行以下步骤：

- 调用生成的绑定类中包含的静态 inflate() 方法。此操作会创建该绑定类的实例以供 Activity 使用。
- 通过调用 getRoot() 方法或使用 Kotlin 属性语法获取对根视图的引用。
- 将根视图传递到 setContentView()，使其成为屏幕上的活动视图。

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
private val requestDatal = registerForActivityResult(ActivityResultContracts.StartActivityForResult()){result-> when(result.resultCode){RESULT_0K -> result.data?.getStringExtra( "data")}
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

```
class FruitAdapter(val fruitList:List<Fruit>):RecyclerView.Adpter<FruitAdapter.ViewHolder>(){
    private lateinit var binding:FruitItemBinding
    inner class ViewHolder(val binding:FruitItemBinding):RecyclerView.ViewHolder(binding.root)

    override fun onCreateViewHolder(parent:ViewGroup,viewType:Int){
        if(!::binding.isInitialized){
            binding = FruitItemBinding.inflate(LayoutInflater.from(parent.context),parent,false)
            return ViewHolder(binding)
        }
    }

    override fun onBindViewHolder(holder:ViewHolder,position:Int){
        val fruit = fruitList[position]
        // 进行视图的赋值,监听等等操作
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

```

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

```
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
