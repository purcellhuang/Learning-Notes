[TOC]
# Glide简单使用
今天我们来学习一下其中一个Android主流的图片加载库的使用 - Glide
## 1. 简介
* 介绍：Glide，是Android中一个图片加载开源库
* 主要作用：实现图片加载
## 2. 开始
在app.gradle中添加依赖
```
dependencies {
    implementation 'com.github.bumptech.glide:glide:3.7.0'
}
```
在AndroidManifest.xml中添加网络权限
```
<uses-permission android:name="android.permission.INTERNET" />
```
在布局文件中添加ImageView控件布局
```
    <ImageView
        android:id="@+id/image_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
```
### 2.1 加载图片

```
    ImageView imageView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        imageView = (ImageView) findViewById(R.id.image_view);
        String url = "http://cn.bing.com/az/hprichbg/rb/Dongdaemun_ZH-CN10736487148_1920x1080.jpg";
        Glide.with(this).load(url).into(imageView);
    }
```
**核心代码**
```
Glide.with(this).load(url).into(imageView);
```
首先，调用Glide.with()方法用于创建一个加载图片的实例。with()方法可以接收Context、Activity或者Fragment类型的参数。也就是说我们选择的范围非常广，不管是在Activity还是Fragment中调用with()方法，都可以直接传this。那如果调用的地方既不在Activity中也不在Fragment中呢？也没关系，我们可以获取当前应用程序的ApplicationContext，传入到with()方法当中。注意with()方法中传入的实例会决定Glide加载图片的生命周期，如果传入的是Activity或者Fragment的实例，那么当这个Activity或Fragment被销毁的时候，图片加载也会停止。如果传入的是ApplicationContext，那么只有当应用程序被杀掉的时候，图片加载才会停止。

接下来看一下load()方法，这个方法用于指定待加载的图片资源。Glide支持加载各种各样的图片资源，包括网络图片、本地图片、应用资源、二进制流、Uri对象等等。因此load()方法也有很多个方法重载，除了我们刚才使用的加载一个字符串网址之外，你还可以这样使用load()方法：
```
// 加载本地图片
File file = new File(getExternalCacheDir() + "/image.jpg");
Glide.with(this).load(file).into(imageView);

// 加载应用资源
int resource = R.drawable.image;
Glide.with(this).load(resource).into(imageView);

// 加载二进制流
byte[] image = getImageBytes();
Glide.with(this).load(image).into(imageView);

// 加载Uri对象
Uri imageUri = getImageUri();
Glide.with(this).load(imageUri).into(imageView);
```

### 2.2 设置缓存
* 设置磁盘缓存策略

```
Glide.with(this).load(imageUrl).diskCacheStrategy(DiskCacheStrategy.ALL).into(imageView);

// 缓存参数说明
// DiskCacheStrategy.NONE：不缓存任何图片，即禁用磁盘缓存
// DiskCacheStrategy.ALL ：缓存原始图片 & 转换后的图片（默认）
// DiskCacheStrategy.SOURCE：只缓存原始图片（原来的全分辨率的图像，即不缓存转换后的图片）
// DiskCacheStrategy.RESULT：只缓存转换后的图片（即最终的图像：降低分辨率后 / 或者转换后 ，不缓存原始图片
```
* 设置跳过内存缓存
```
Glide
  .with(this)
.load(imageUrl)
.skipMemoryCache(true)
.into(imageView);
//设置跳过内存缓存
//这意味着 Glide 将不会把这张图片放到内存缓存中去
//这里需要明白的是，这只是会影响内存缓存！Glide 将会仍然利用磁盘缓存来避免重复的网络请求。
```
* 清理缓存
```
Glide.get(this).clearDiskCache();//清理磁盘缓存 需要在子线程中执行 
Glide.get(this).clearMemory();//清理内存缓存 可以在UI主线程中进行
```

### 2.3 占位图
顾名思义，占位图就是指在图片的加载过程中，我们先显示一张临时的图片，等图片加载出来了再替换成要加载的图片。
```
Glide.with(this)
     .load(url)
     .placeholder(R.drawable.loading)
     .diskCacheStrategy(DiskCacheStrategy.NONE) //禁用掉Glide的缓存功能
     .into(imageView);
```
当然，这只是占位图的一种，除了这种加载占位图之外，还有一种异常占位图。异常占位图就是指，如果因为某些异常情况导致图片加载失败，比如说手机网络信号不好，这个时候就显示这张异常占位图。
```
Glide.with(this)
     .load(url)
     .placeholder(R.drawable.loading)
     .error(R.drawable.error)
     .diskCacheStrategy(DiskCacheStrategy.NONE)
     .into(imageView);
```
### 2.4 指定图片格式
Glide另外一个强大的功能，那就是Glide是支持加载GIF图片的。这一点确实非常牛逼，因为相比之下Jake Warton曾经明确表示过，Picasso是不会支持加载GIF图片的。
```
Glide.with(this)
     .load(url)
     .asBitmap() //加载静态图片
     //.asGif()  加载动态图片
     .placeholder(R.drawable.loading)
     .error(R.drawable.error)
     .diskCacheStrategy(DiskCacheStrategy.NONE)
     .into(imageView);
```
### 2.5 指定图片大小
在绝大多数情况下我们都是不需要指定图片大小的，因为Glide会自动根据ImageView的大小来决定图片的大小。

不过，如果你真的有这样的需求，必须给图片指定一个固定的大小，Glide仍然是支持这个功能的。修改Glide加载部分的代码，如下所示：
```
Glide.with(this)
     .load(url)
     .placeholder(R.drawable.loading)
     .error(R.drawable.error)
     .diskCacheStrategy(DiskCacheStrategy.NONE)
     .override(100, 100)
     .into(imageView);
```
### 2.6 图片变换功能
顾名思义，图片变换的意思就是说，Glide从加载了原始图片到最终展示给用户之前，又进行了一些变换处理，从而能够实现一些更加丰富的图片效果，如图片圆角化、圆形化、模糊化等等。