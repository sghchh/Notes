# Android图片加载库--Glide  
## 一 使用篇  
### 1. 准备工作  
要想使用Glide这个第三方库，需要添加依赖  

	compile 'com.github.bumptech.glide:glide:3.7.0'  
当然，别忘了在**清单文件中添加网络权限（加载远程网络图片）或者SD卡权限（加载本地SD卡上的图片）**  
### 2. 使用  
最简单的使用就是：  

	Glide.with(Context).load(Url).into(ImageView)  
下面就来介绍这一步一步的方法都是什么作用。  
### 3.绑定生命周期  
 Glide通过**with()静态方法**来绑定生命周期，自行管理加载图片的请求的生命周期，该方法的参数有四种选择：  

	Glide.with(Context context);// 绑定Context
    Glide.with(Activity activity);// 绑定Activity
    Glide.with(FragmentActivity activity);// 绑定FragmentActivity
    Glide.with(Fragment fragment);// 绑定Fragment   
 
### 4.加载图片  
Glide加载的图片根据来源不同，其load()方法的参数也有多种选择：  

	DrawableTypeRequest<String> load(String string)
	DrawableTypeRequest<Uri> load(Uri uri)
	DrawableTypeRequest<File> load(File file)
	DrawableTypeRequest<Integer> load(Integer resourceId)
	DrawableTypeRequest<URL> load(URL url)  
其中最后一种通过URL参数的加载已经被标记@Deprecated了，不再推荐使用。
### 5.占位图  
在使用Glide加载图片的时候，或多或少会花费一些时间才能显示出来，时间短到人眼无法识别最好，但是如果是很明显的一段时间是空白，这时候图片突然加载出来，难免会有一种突兀的感觉，这个时候就可以设置一个占位图，专门在这一段空白时间来供ImageView显示,**placeholder(int resourceId)**：  

	  Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/55.jpg").placeholder(R.mipmap.place).into(imageView);  

上面说的是加载过程中的占位图，但是如果是出现了错误情况呢？比如给的Url不对，File路径不对等，导致错误情况下无法加载图片，这时候用户也不知道发生了什么情况，如果ImageView没有图片显示的话，会是一个空白区域，用户就不知道这里有一块图片显示的区域，所以为了在错误的情况下有所显示，也可以设置占位图，**error(int resourceId)**,使用方式同上。  
### 6.增加监听  
上面说到，可以通过**error(int)**方法在加载错误的时候显示一个指定的图片，如果不满足于此，想知道错误的原因的话可以通过增加一个监听来实现；**listener(RequestListener)**:  
这个参数创建的时候，要重写两个方法：  

	  /设置错误监听
         RequestListener<String,GlideDrawable> errorListener=new RequestListener<String, GlideDrawable>() {
			//错误的时候回调
            @Override
            public boolean onException(Exception e, String model, Target<GlideDrawable> target, boolean isFirstResource) {

                Log.e("onException",e.toString()+"  model:"+model+" isFirstResource: "+isFirstResource);
                imageView.setImageResource(R.mipmap.ic_launcher);
                return false;
            }
			//成功的时候回调
            @Override
            public boolean onResourceReady(GlideDrawable resource, String model, Target<GlideDrawable> target, boolean isFromMemoryCache, boolean isFirstResource) {
                Log.e("onResourceReady","isFromMemoryCache:"+isFromMemoryCache+"  model:"+model+" isFirstResource: "+isFirstResource);
                return false;
            }
        } ;  
当错误的时候回调方法中的Exception参数，可以通过该参数了解到是什么原因导致了错误。  
### 7.动画的控制和取消  
Glide在加载图片的时候默认是有一个**淡入淡出的动画的，默认时长是300ms**，我们可以通过**crossFade(int time)**这个方法来**控制这个时长**；如果不想要这个动画可以调用**dontAnimate()**方法取消。  
### 8.图片预处理  
有时候有了特殊要求，需要将图片进行一些处理再显示，比如显示圆角，进行高斯模糊等等；这时候可以通过**transform(BitmapTransformation)**方法来进行预处理，用法同**listener(RequestListener)**类似，参数是需要自定义一个类继承自BitmapTransformation，重写其两个方法：  

	public  class CornersTransform extends BitmapTransformation {

        @Override
        protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
            //TODO
        }



        @Override
        public String getId() {
           //TODO
        }
	}  
第一个方法中含有Bitmap实例，所以这里就是对图片进行处理的地方。  
### 9.图片裁剪  
图片裁剪不同于预处理，目的是经过裁剪使得图片可以适应ImageView的大小，显示的更加“正常”而已；而预处理的目的是使得图片产生另外一种效果。  
#### ImageView默认的ScaleType  

* matrix：不缩放 ,图片与控件 左上角 对齐,当图片大小超过控件时将被 裁剪  
* center：不缩放 ,图片与控件 中心点 对齐,当图片大小超过控件时将被 裁剪
* centerInside：以完整显示图片为目标, 不剪裁 ,当显示不下的时候将缩放,能够显示的情况下不缩放
* centerCrop：以填满整个控件为目标,等比缩放,超过控件时将被 裁剪 ( 宽高都要填满 ,所以只要图片宽高比与控件宽高比不同时,一定会被剪裁)  
* fitCenter(**默认**)：自适应控件, 不剪裁 ,在不超过控件的前提下,等比 缩放 到 最大 ,居中显示
* fitStart：自适应控件, 不剪裁 ,在不超过控件的前提下,等比 缩放 到 最大 ,靠左(上)显示
* fitEnd：自适应控件, 不剪裁 ,在不超过控件的前提下,等比 缩放 到 最大 ,靠右(下)显示
* fitXY：以填满整个控件为目标, 不按比例 拉伸或缩放(可能会变形), 不剪裁  

#### Glide配置    
Glide内置了两个方法设置图片的裁剪策略：  

1. fitCenter()
2. centerCrop()  

这两个方法其实都是通过**调用transform方法**来对图片进行处理  
### 10.缓存控制  
Glide的缓存资源分为两种：  

* **原图（SOURCE）：原始图片**  
* **处理图（RESULT）：经过压缩和变形处理过的图片**  

#### 10.1 内存的缓存控制  
Glide默认是在内存中**缓存处理图**的；可以通过**skipMemoryCache(boolean)**方法人为控制是否跳过内存缓存，由于默认是进行的，即**默认是false**  
#### 10.2 磁盘缓存控制  
Glide的磁盘缓存策略有四种：  

* ALL：缓存原图和处理图  
* NONE：什么都不缓存  
* SOURCE：只缓存原图  
* RESULT：只缓存处理图（**默认值**）  

可以看到Glide默认是在内存和磁盘中都是缓存的处理图的。想要控制磁盘缓存的策略，调用**diskCacheStrategy()方法，参数是四种策略之一**就能够控制了。  
#### 10.3 清除所有缓存  
**清除所有内存缓存(需要在Ui线程操作)**  

	Glide.get(this).clearMemory();  
**清除所有磁盘缓存(需要在子线程操作)**  

	Glide.get(MainActivity.this).clearDiskCache();  
**注:在使用中的资源不会被清除**  

### 11.使用GlideMoudle定制Glide
我们一般情况下使用Glide都很简单,只用简单的调用几个方法就能够很好的显示图片了,但其实Glide在初始化的时候进行了一系列的默认配置,比如缓存的配置,图片质量的配置等等.接下来我们就介绍一下一个比较高级的功能,通过Modules定制自己的个性Glide  
#### 11.1 创建一个类实现GlideMoudle  

	public class XiayuGlideModule implements GlideModule {

    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
       //TODO
    }

    @Override
    public void registerComponents(Context context, Glide glide) {
       //TODO
    }
	}  
#### 11.2 配置清单文件  
在AndroidManifest中配置刚刚创建的GlideModule,需要在application节点下添加  

	<meta-data
        android:name="com.xiayu.xiayuglidedemo.XiayuGlideModule"
        android:value="GlideModule" />  
#### 11.3 进行自定义配置  
**刚才创建的GlideModule的实现类时,会要实现两个方法,这里要用到的是其中的applyOptions方法,applyOptions方法里面提供了一个GlideBuilder,通过GlideBuilder我们就能实现自定义配置了**  

	public class XiayuGlideModule implements GlideModule {

    @Override
    public void applyOptions(Context context, GlideBuilder builder) {

        builder.setDiskCache();//自定义磁盘缓存

        builder.setMemoryCache();//自定义内存缓存

        builder.setBitmapPool(); //自定义图片池

        builder.setDiskCacheService();//自定义本地缓存的线程池

        builder.setResizeService();//自定义核心处理的线程池

        builder.setDecodeFormat();//自定义图片质量

    }

    @Override
    public void registerComponents(Context context, Glide glide) {
        //TO
    }
	}  
>相关属性的自定义配置参考：[http://blog.csdn.net/shangmingchao/article/details/51026742](http://blog.csdn.net/shangmingchao/article/details/51026742)
### 12.图片压缩  
#### 12.1 安卓图片的相关知识   
安卓的图片显示的质量配置主要有四种：  

* ARGB_8888 :32位图,带透明度,每个像素占4个字节
* ARGB_4444 :16位图,带透明度,每个像素占2个字节
* RGB_565 :16位图,不带透明度,每个像素占2个字节
* ALPHA_8 :32位图,只有透明度,不带颜色,每个像素占4个字节  

而Glide默认的图片质量是RGB_565；所以想要加载一张4000*2000像素的图片，要的内存是4000*2000*2/1024/1024=15MB，所以同时加载几张图片就会OOM，不管你的ImageView的宽高是多少。  
所以我们要进行图片的压缩：  
#### 12.2 图片质量的压缩  
更改图片的质量可以达到降低内存消耗的作用，更改图片质量需要通过自定义GlideMoudle实现  
#### 12.3 图片尺寸压缩  
通过调用**override(int wpx,int hpx)**来讲图片压缩尺寸  
### 13.加载特殊图  
Glide也可以加载gif或者视频缩略图:  
#### 13.1 加载gif  
加载gif和加载普通图片没有任何区别  
#### 13.2 加载gif缩略图  
如果只希望加载gif图的第一帧，当做一个普通图一样，只需要加上**asBitmap()方法**即可：  

	  Glide.with(this).load(mGifUrl).asBitmap().placeholder(R.mipmap.place).error(R.mipmap.icon_photo_error).into(mIv);  
#### 13.3 检查是否是gif  
如果希望加载的是一个gif，这时候如果不是的话就会显示**错误图片error**，只要加上**asGif()方法**即可  

	Glide.with(this).load(mGifUrl).asGif().placeholder(R.mipmap.place).error(R.mipmap.icon_photo_error).into(mIv);  
#### 13.4 加载本地视频缩略图  
Glide只会加载本地视频的第一帧,也就是缩略图，这种情况就是将一个数据源是视频的参数传入了**load()**方法中，Glide只会加载第一帧。  
#### 14. Target--特殊需求特殊处理  
上面所说的也是我们**常见的使用Glide的情况--使用ImageView加载远程图片**，假如我只是想得到远程的图片，并不用ImageView去显示它该怎么办呢？这时候我们的第一个Target就登场了--SimpleTarget  
##### 14.1 SimpleTarget  
Target的使用是在Glide调用**into()**方法时，**代替ImageView作为参数传进去**。  
首先我们得先创建一个SimpleTarget实例，**实现其中的onResourceReady方法：**  

	private SimpleTarget target = new SimpleTarget<Bitmap>() {  
    @Override
    public void onResourceReady(Bitmap bitmap, GlideAnimation glideAnimation) {
        // do something with the bitmap
        // for demonstration purposes, let's just set it to an ImageView
        imageView1.setImageBitmap( bitmap );
    }
	};  
看到，该**方法的参数中有我们想要的Bitmap对象**，我们可以对这个Bitmap进行任何操作。  
>为了防止我们的数据源是一个**gif图**或者是**视频**，我们在使用Glide.into(target)之前最好调用一下**asBitmap()方法，确保得到的是Bitmap**
 
----  
>**注意：**1. 首先是 SimpleTarget 对象的字段声明。从技术上来说，Java/Android 会允许你在 **.into()** 方法中去声明 target 的匿名内部类。然而，这大大增加了这样一个可能性：即在 Glide 做完图片请求之前， Android 垃圾回收移除了这个匿名内部类对象。最终这可能会导致一个情况，当图像加载完成了，但是回调再也不会被调用。所请确保你所声明的回调对象是作为一个字段对象的，这样你就可以保护它避免被邪恶的 Android 垃圾回收机制回收 ;  
>2. 第二个关键部分是 Glide 建造者中这行：**.with(context)**。 这里的问题实际是 Glide 的功能：当你传了一个 context（例如是当前应用的 activity），Glide 将会自动停止请求当 activity 已经停止的时候。这整合到了应用的生命周期中通常是非常有帮助的，但是有时工作起来是困难的，如果你的 target 是独立于应用的 activity 生命周期。这里的解决方案是用 application 的 context: .with(context.getApplicationContext))。当应用资深完全停止时，Glide 才会杀死这个图片请求。

另一个潜在的问题是，target 没有指明大小。如果你传一个 ImageView 作为参数给 **.into()**，Glide 将会用 ImageView 的大小去限制图像的大小。比如说，如果加载的图片是 1000x1000 像素的，但是 ImageView 只有 250x250 像素，Glide 将会减少图片的尺寸去节省时间和内存。很显然，在和 target 协作的时候并没有这么做，因为我们并没有已知的大小。然而，如果你有一个指定的大小，你可以加强回调。如果你知道这种图片应该要多大，你应该在你的回调声明中指定它以节省一些内存：**在创建SimpleTarget实例的时候传入两个int类型参数作为指明的宽和高（单位是px）**。  

##### 14.2 ViewTarget  
有时候我们不能使用ImageView，使用了一个自定义的View(**不是继承自ImageView**)，这时候**into()**方法就不能够将这个View的实例作为参数传入，我们应该使用**ViewTarget**来解决这个问题：  

	viewTarget = new ViewTarget<DefinedView, GlideDrawable>( customView ) {
        @Override
        public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> glideAnimation) {
            this.view.setImage( resource.getCurrent() );
        }
    };
可以看到**ViewTarget泛型中要有自定义View的类名**
