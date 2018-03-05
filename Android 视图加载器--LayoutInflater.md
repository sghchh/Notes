#Android 视图加载器--LayoutInflater源码学习  
>参考文章：[https://www.jianshu.com/p/b4af0e96e57e](https://www.jianshu.com/p/b4af0e96e57e)  

## 1. 获取LayoutInflater对象  
通常我们获取LayoutInflater对象是这么获取的：  

	LayoutInflater inflater=LayoutInflater.from(context);  

进入这个**from(Context)**源码看看LayoutInflater是如何实现的：  
**LayoutInflater.java:**  

	/**
 	* Obtains the LayoutInflater from the given context.
 	*/
	public static LayoutInflater from(Context context) {
    LayoutInflater LayoutInflater =
            (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    if (LayoutInflater == null) {
        throw new AssertionError("LayoutInflater not found.");
    }
    return LayoutInflater;
	}
我们看到，最后调用的是**Context.getSystemService(Context.Layout_INFLATER_SERVICE)**这个方法；再次进入**Context**的源码看看：  
**Context.java:**  

	public abstract Object getSystemService(@ServiceName @NonNull String name);  
我们可以看到，只有一个抽象方法，而且这个参数是一个**String**对象，我们再来看看**Context的实现类：ContextImpl：**  
**ContextImpl.java:**  

	@Override
	public Object getSystemService(String name) {
    //继续调用SystemServiceRegistry.getSystemService
    return SystemServiceRegistry.getSystemService(this, name);
	}   
看到ContextImpl内部是调用的**SystemServiceRegistry**这个类的静态方法**getSystemService(ContextImpl ctx,String name)**：  
**SystemServiceRegistry.java:**  

	/**
 	* Gets a system service from a given context.
 	*/
	public static Object getSystemService(ContextImpl ctx, String name) {
    //先获取ServiceFetcher，在通过fetcher获取LayoutInflate
    ServiceFetcher<?> fetcher = SYSTEM_SERVICE_FETCHERS.get(name);
    return fetcher != null ? fetcher.getService(ctx) : null;
	}  
看到这个源码后，我们要知道**SYSTEM_SERVICE_FETCHERS是一个怎样的成员，ServiceFetcher是一个怎样的类**  
**SystemServiceRegistry.java:**  

	//使用键值对来保存
	private static final HashMap<String, ServiceFetcher<?>> SYSTEM_SERVICE_FETCHERS =
        new HashMap<String, ServiceFetcher<?>>();

	/**
 	* Base interface for classes that fetch services.
 	* These objects must only be created during static initialization.
 	*/
	static abstract interface ServiceFetcher<T> {
    //只有一条接口，通过context获取服务，先看一下其实现类
    T getService(ContextImpl ctx);
	}
首先，**SYSTEM_SERVICE_FETCHERS**是一个装有**ServiceFetcher**的一个**Map集合**，所以上面的**SYSTEM_SERVICE_FETCHERS.get(name)**得到**ServiceFetcher**对象只是简单的从集合中取出而已。  
而**ServiceFetcher**只是一个接口，而且有一个方法**getService(ContextImpl ctx)**。  
在**SystemServiceRegistry**的源码中，我们还发现了一段静态代码块儿：  

	static {
    .....
    registerService(Context.LAYOUT_INFLATER_SERVICE, LayoutInflater.class,
        new CachedServiceFetcher<LayoutInflater>() {
    @Override
    public LayoutInflater createService(ContextImpl ctx) {
        return new PhoneLayoutInflater(ctx.getOuterContext());
    }});

	......
	}  
在这个静态代码块中，通过registerService进行初始化注册服务。我们先看看这个静态方法。  

	/**
 	* Statically registers a system service with the context.
 	* This method must be called during static initialization only.
 	*/
	private static <T> void   registerService(String serviceName, Class<T> serviceClass,
        ServiceFetcher<T> serviceFetcher) {
    SYSTEM_SERVICE_NAMES.put(serviceClass, serviceName);
    SYSTEM_SERVICE_FETCHERS.put(serviceName, serviceFetcher);
	}  
我们通过**put()方法**也可以猜到这个**SYSTEM_SERVICE_NAMES**也是一个Java容器类实例，在这里只是保存了一个服务对象；也就是说**SystemServiceRegistry会初始化的时候注册各种服务**，而我们在获取LayoutInflater的时候传入的key参数是**Context.LAYOUT_INFLATER_SERVICE**,而这个key代表的字符串的值是**“layout_inflater”**  
看完了注册了服务，我们来看看**ServiceFetcher**的实现类，因为在**SystemServiceRegistry**源码中，我们最后返回的是**ServiceFetcher.getService(ContextImpl ctx).**  

**ServiceFetcher实现类CacheServiceFetcher.java:**  

	/**
 	* Override this class when the system service constructor needs a
 	* ContextImpl and should be cached and retained by that context.
 	*/
	static abstract class CachedServiceFetcher<T> implements ServiceFetcher<T> {
    private final int mCacheIndex;

    public CachedServiceFetcher() {
        mCacheIndex = sServiceCacheSize++;
    }

    @Override
    @SuppressWarnings("unchecked")
    public final T getService(ContextImpl ctx) {
        final Object[] cache = ctx.mServiceCache;
        synchronized (cache) {
            // Fetch or create the service.
            Object service = cache[mCacheIndex];
            if (service == null) {
                service = createService(ctx);
                cache[mCacheIndex] = service;
            }
            return (T)service;
        }
    }

    public abstract T createService(ContextImpl ctx);
	}  
CachedServiceFetcher的作用用于保存我们的泛型T,同时这个CachedServiceFetcher有一个抽象方法createService，createService这个方法用来创建这个服务，因此使用这个类就必须重写这个方法，我们继续看回： 

**SystemServiceRegistry.java:**  

	 /**
 	* Gets a system service from a given context.
 	*/
	public static Object getSystemService(ContextImpl ctx, String name) {
    //先获取ServiceFetcher，在通过fetcher获取LayoutInflate
    ServiceFetcher<?> fetcher = SYSTEM_SERVICE_FETCHERS.get(name);
    return fetcher != null ? fetcher.getService(ctx) : null;
	}


**ServiceFetcher.java:**  

	staic{
    registerService(Context.LAYOUT_INFLATER_SERVICE, LayoutInflater.class,
            new CachedServiceFetcher<LayoutInflater>() {
        @Override
        public LayoutInflater createService(ContextImpl ctx) {
            return new PhoneLayoutInflater(ctx.getOuterContext());
    }});
	}  
**ServiceFetcher实现类CacheServiceFetcher.java:**  

	/**
 	* Override this class when the system service constructor needs a
 	* ContextImpl and should be cached and retained by that context.
 	*/
	static abstract class CachedServiceFetcher<T> implements ServiceFetcher<T> {
    private final int mCacheIndex;

    public CachedServiceFetcher() {
        mCacheIndex = sServiceCacheSize++;
    }

    @Override
    @SuppressWarnings("unchecked")
    public final T getService(ContextImpl ctx) {
        final Object[] cache = ctx.mServiceCache;
        synchronized (cache) {
            // Fetch or create the service.
            Object service = cache[mCacheIndex];
            if (service == null) {
                service = createService(ctx);
                cache[mCacheIndex] = service;
            }
            return (T)service;
        }
    }

    public abstract T createService(ContextImpl ctx);
	}  

首先，代码到**SystemServiceRegistry**我们知道，一路**return**，最终返回的是**fetcher.getService(ctx)**的结果；而fetcher的具体的对象是**CachedServiceFetcher**实例，后者对**getService()**的实现中当service为空时调用了**createService(ctx)**方法；在**ServiceFetcher**中，我们是这么重写该方法的：  

	new CachedServiceFetcher<LayoutInflater>() {
        @Override
        public LayoutInflater createService(ContextImpl ctx) {
            return new PhoneLayoutInflater(ctx.getOuterContext());
    }});  
最后得到的是**PhoneLayoutInflater--LayoutInflater的实现类。**  
我们获取LayoutInflater对象可以通过两种方法：  

	LayoutInflater inflater1=LayoutInflater.from(context);
	LayoutInflater inflater2= (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);

## 2. LayoutInflater.inflater(int id,ViewGroup root,boolean attachToRoot)  

**方法一：**  

	public View inflate(@LayoutRes int resource, @Nullable ViewGroup root) {
    return inflate(resource, root, root != null);
    }

**方法二：**

	public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) {
    final Resources res = getContext().getResources();
    if (DEBUG) {
        Log.d(TAG, "INFLATING from resource: \"" + res.getResourceName(resource) + "\" ("
                + Integer.toHexString(resource) + ")");
    }
    //通过资源加载器和资源Id,获取xml解析器
    final XmlResourceParser parser = res.getLayout(resource);
    try {
        return inflate(parser, root, attachToRoot);
    } finally {
        parser.close();
    }
	}  
可以看到最终都是调用的方法二，核心方法是**inflate(parser, root, attachToRoot)**。    

	public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
    synchronized (mConstructorArgs) {
        // ...

        // 因为parser实现了AttributeSet接口，所以这里是强转
        final AttributeSet attrs = Xml.asAttributeSet(parser);

        // result是需要return的值
        View result = root;

        try {
            // 通过一个循环，寻找根节点
            int type;
            while ((type = parser.next()) != XmlPullParser.START_TAG &&
                    type != XmlPullParser.END_DOCUMENT) {
                // Empty
            }

            if (type != XmlPullParser.START_TAG) {
                // 如果没找到根节点，报错
                throw new InflateException(parser.getPositionDescription()
                        + ": No start tag found!");
            }

            // 找到了根节点，获取根节点的名称
            final String name = parser.getName();

            if (TAG_MERGE.equals(name)) {
                // 如果根节点是merge标签
                if (root == null || !attachToRoot) {
                    // merge标签要求传入的ViewGroup不能是空，并且attachToRoot必须为true， 否则报错
                    throw new InflateException("<merge /> can be used only with a valid "
                            + "ViewGroup root and attachToRoot=true");
                }

                // 递归生成根节点下的所有子节点
                rInflate(parser, root, inflaterContext, attrs, false);
            } else {
                // 根据节点的信息（名称、属性）生成根节点View对象
                final View temp = createViewFromTag(root, name, inflaterContext, attrs);

                // 根节点的LayoutParams属性
                ViewGroup.LayoutParams params = null;

                if (root != null) {
                    // 如果传入的ViewGroup不为空

                    // 调用root的generateLayoutParams方法来生成根节点的LayoutParams属性对象
                    params = root.generateLayoutParams(attrs);
                    if (!attachToRoot) {
                        // 不需要讲根节点添加到传入的ViewGroup节点下，则将LayoutParams对象设置到根节点内
                        // 否则的话在后面将会通过addView方式设置params
                        temp.setLayoutParams(params);
                    }
                }

                if (DEBUG) {
                    System.out.println("-----> start inflating children");
                    // 开始解析所有子节点
                }

                // 解析根节点下的子节点
                rInflateChildren(parser, temp, attrs, true);

                if (DEBUG) {
                    System.out.println("-----> done inflating children");
                    // 结束了所有子节点的解析
                }

                if (root != null && attachToRoot) {
                    // 如果传入的ViewGroup不是空，并且需要添加根节点到其下面
                    root.addView(temp, params);
                }

                if (root == null || !attachToRoot) {
                    // 如果根节点为空，或者是attachToRoot为false，返回根节点
                    result = temp;
                }
            }

        } catch (XmlPullParserException e) {
            // ....
        } catch (Exception e) {
            // ....
        } finally {
            // ....
        }

        // return 结果（根节点或者是传入的ViewGroup）
        return result;
    }
	}

重点还是else下面的代码：  

	else {
                // 根据节点的信息（名称、属性）生成根节点View对象
                final View temp = createViewFromTag(root, name, inflaterContext, attrs);

                // 根节点的LayoutParams属性
                ViewGroup.LayoutParams params = null;

                if (root != null) {
                    // 如果传入的ViewGroup不为空

                    // 调用root的generateLayoutParams方法来生成根节点的LayoutParams属性对象
                    params = root.generateLayoutParams(attrs);
                    if (!attachToRoot) {
                        // 不需要讲根节点添加到传入的ViewGroup节点下，则将LayoutParams对象设置到根节点内
                        // 否则的话在后面将会通过addView方式设置params
                        temp.setLayoutParams(params);
                    }
                }

                if (DEBUG) {
                    System.out.println("-----> start inflating children");
                    // 开始解析所有子节点
                }

                // 解析根节点下的子节点
                rInflateChildren(parser, temp, attrs, true);

                if (DEBUG) {
                    System.out.println("-----> done inflating children");
                    // 结束了所有子节点的解析
                }

                if (root != null && attachToRoot) {
                    // 如果传入的ViewGroup不是空，并且需要添加根节点到其下面
                    root.addView(temp, params);
                }

                if (root == null || !attachToRoot) {
                    // 如果根节点为空，或者是attachToRoot为false，返回根节点
                    result = temp;
                }
            }
首先**temp**就是我们inflate方法参数中加载的布局**根布局**；可以看到：  

	if (!attachToRoot) {
                    // 不需要讲根节点添加到传入的ViewGroup节点下，则将LayoutParams对象设置到根节点内
                    // 否则的话在后面将会通过addView方式设置params
                    temp.setLayoutParams(params);
	 }  
当**root不为空，且attachToRoot为false的时候才会根据root生成布局参数LayoutParams，也就是说如果root为空的话，加载的布局所有的以“layout_”开头的属性都没有效果了**  
而代码：  

	if (root == null || !attachToRoot) {
                // 如果根节点为空，或者是attachToRoot为false，返回根节点
                result = temp;
            }  
这段代码表明，当**attachToRoot为false的时候**返回的是**temp--加载的布局的根布局**；所以当**attachToRoot为true返回的就是root了。**

## 3.RequestFocus、Include、Merge、ViewStub标签  
定义Android Layout(XML)时，有四个比较特别的标签是非常重要的，其中有三个是与资源复用有关，分别是<viewStub/>, <requestFocus />, <merge /> and<include />。可是以往我们所接触的案例或者官方文档的例子都没有着重去介绍这些标签的重要性。  
### 3.1 viewStub  
**viewStub:**此标签可以使UI在特殊情况下，直观效果类似于设置View的不可见性，但是其更大的(R)意义在于被这个标签所包裹的Views在默认状态下不会占用任何内存空间。viewStub通过include从外部导入Views元素。  
**用法：通过android:layout来指定所包含的内容。默认情况下，ViewStub所包含的标签都属于visibility=GONE。viewStub通过方法inflate()来召唤系统加载其内部的Views。**  

	<ViewStub android:id="@+id/stub"
	android:inflatedId="@+id/subTree"
	android:layout="@layout/mySubTree"
	android:layout_width="120dip"
	android:layout_height="40dip" />  
### 3.2 include  
**include:**可以通过这个标签直接加载外部的xml到当前结构中，是复用UI资源的常用标签。  
**用法：将需要复用xml文件路径赋予include标签的Layout属性。**  

	<include android:id="@+id/cell1" layout="@layout/ar01" />
	<include android:layout_width="fill_parent" layout="@layout/ar02" />  
### 3.3 requestFocus  
**requestFocus:**标签用于指定屏幕内的焦点View  
**用法: 将标签置于Views标签内部**  

	<EditText id="@+id/text"
             android:layout_width="fill_parent"
             android:layout_height="wrap_content"
             android:layout_weight="0"
             android:paddingBottom="4">
       <requestFocus />
	</EditText>  
### 3.4 merge  
**merge:**目的是通过删减多余或者额外的层级，从而优化整个Android Layout的结构。  
**将通过一个例子来了解这个标签实际所产生的作用，这样可以更直观的了解merge的用法。**  
建立一个简单的Layout，其中包含两个Views元素：ImageView和TextView 默认状态下我们将这两个元素放在FrameLayout中。其效果是在主视图中全屏显示一张图片，之后将标题显示在图片上，并位于视图的下方。以下是xml代码：  

	<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">

    <ImageView
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"

        android:scaleType="center"
        android:src="@drawable/golden_gate" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="20dip"
        android:layout_gravity="center_horizontal|bottom"

        android:padding="12dip"

        android:background="#AA000000"
        android:textColor="#ffffffff"

        android:text="Golden Gate" />

	</FrameLayout>  
启动 tools> hierarchyviewer.bat工具查看当前UI结构视图：

layouttutorial_merge_01

layouttutorial_merge_02

我们可以很明显的看到由红色线框所包含的结构出现了两个framelayout节点，很明显这两个完全意义相同的节点造成了资源浪费（这里可以提醒大家在开发工程中可以习惯性的通过hierarchyViewer查看当前UI资源的分配情况），那么如何才能解决这种问题呢（就当前例子是如何去掉多余的frameLayout节点）？这时候就要用到<merge />标签来处理类似的问题了。我们将上边xml代码中的framLayout替换成merge：  

	<merge xmlns:android="http://schemas.android.com/apk/res/android">

    <ImageView
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"

        android:scaleType="center"
        android:src="@drawable/golden_gate" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="20dip"
        android:layout_gravity="center_horizontal|bottom"

        android:padding="12dip"

        android:background="#AA000000"
        android:textColor="#ffffffff"

        android:text="Golden Gate" />

	</merge>   

运行程序后在Emulator中显示的效果是一样的，可是通过hierarchyviewer查看的UI结构是有变化的，当初多余的 FrameLayout节点被合并在一起了，或者可以理解为将merge标签中的子集直接加到Activity的FrameLayout跟节点下（这里需要提醒大家注意：所有的Activity视图的根节点都是frameLayout）。如果你所创建的Layout并不是用framLayout作为根节点（而是应用LinerLayout等定义root标签），就不能应用上边的例子通过merge来优化UI结构。

layouttutorial_merge_03  

>merge:只可以作为xml layout的根节点。
当需要扩充的xml layout本身是由merge作为根节点的话，需要将被导入的xml layout置于 viewGroup中，同时需要设置attachToRoot为True。