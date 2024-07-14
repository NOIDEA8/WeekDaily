# Glide（返回**RequestBuilder** ）

  Glide 是一个快速高效的 Android 开源媒体管理和图像加载框架，它将媒体解码、内存和磁盘缓存以及资源池封装到一个简单易用的界面中。

​    Glide 支持**拉取，解码和展示视频快照，图片，和GIF动画**。

## 1.权限申请

在注册表中申请权限：

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

在build.gradle中增添依赖

```groovy
implementation 'com.github.bumptech.glide:glide:4.12.0'
```

## 2.基本用法

Glide 建造者要求最少有三个参数才能完成一个完整的工作。

```java
//图片加载
Glide.with(context)
    .load(url)
    .into(imageView);

//取消加载imageview中图片
Glide.with(context).clear(imageView);
```

1. **context** ：可以传入fragment 、activity 和applicationContext 等所在的位置。
   **load** ：图片资源。可以是resourceId 、url 、uri 、drawable 、file 等资源标识。
   **into** ：图片会显示到对应的 ImageView 中。

2. 取消加载并非是必要的操作，在Glide.with(）中传入的context如果是activity或fragment，当它们的实例销毁时，Glide 会自动取消加载并回收资源。

小拓展：
resourceID转换为Uri的方法

```java
private static Uri resourceIdToUri(Context context, int resourceId) {
    return Uri.parse("android.resource://" + context.getPackageName()
     + "/" + resourceId);
}
```

# —————————————————————

## 3.占位符

Glide允许用户指定三种不同类型的占位符，分别在三种不同场景使用。他们分别是placeholder，error和falllback。

### 1.placeholder（占位符）

，这个是后两个的父级，简单来说**起背景图的作用**。占位符是当请求正在执行时被展示的 Drawable 。当请求成功完成时，占位符会被请求到的资源替换。

```java
Glide.with(context)
  .load(url)
  .placeholder(R.drawable.placeholder)//虽然这里传的是id但是也可以传RequestBulider对象（就是再次进行Glide）
  .into(view);
```

### 2.error（错误符）

error Drawable **在请求永久性失败时展示，比如加载失败等调用**。
error()接受的参数只能是已经初始化的 drawable 对象或者指明它的资源(R.drawable.xxx)

```java
Glide.with(context)
  .load(url)
  .error(R.drawable.error)//虽然这里传的是id但是也可以传RequestBulider对象（就是再次进行Glide）
  .into(view);
```

在 error 方法里 来指定一个 RequestBuilder，可以在主请求失败时开始一次新的加载。如果主请求成功完成，这个error里的RequestBuilder 将不会被启动。如果你同时指定了一个 thumbnail() 和一个 error() RequestBuilder，则这个后备的 RequestBuilder 将在主请求失败时启动，即使缩略图请求成功也是如此。

### 3.fallback（后备回调符）

fallback Drawable 在请求的**url为 null** 时展示。

```java
Glide.with(context)
  .load(url)
  .fallback(R.drawable.fallback)//虽然这里传的是id但是也可以传RequestBulider对象（就是再次进行Glide）
  .into(view);
```

当Gilde.load（）中传入的资源为null，或者是无法成功获取时，展示优先级为fallback > error > placeholder。
占位符都是独立的，可以根据开发需要来调用。除基本用法中的三个方法，Glide的其他方法也是如此。

## 3.选项

### 1.请求选项

Glide中的大部分设置项都可以直接应用（通过新建 RequestOptions对象在引用选项最后apply）在 **Glide.with() 返回的** **RequestBuilder** 对象上（也可以直接接在Glide后面，但是这样不利于复用），表达对相关图像的请求。

可用的选项包括但不限于：占位符(Placeholders)、转换(Transformations)、缓存策略(Caching Strategies)、组件特有的设置项，例如编码质量，或Bitmap的解码配置等。
**如果想让应用的不同部分之间共享相同的加载选项，可以初始化一个新的 RequestOptions 对象**并在每次加载时通过 apply() 方法传入这个对象：

```java
RequestOptions cropOptions = new RequestOptions().centerCrop(context);

Glide.with(context)
    .load(url)
    .apply(cropOptions)
    .into(imageView);
```

**apply() 方法可以被调用多次，因此 RequestOption 可以被组合使用。如果 RequestOptions 对象之间存在相互冲突的设置，那么只有最后一个被应用的 RequestOptions 会生效。**

### 2.过渡选项（多图片时缓冲切换）(Transition)

TransitionOptions 用于决定加载完成时会发生什么。

使用 TransitionOption 可以应用以下变换：View淡入、与占位符交叉淡入或者什么都不发生
用处：如果不使用变换，你图像将会直接出现在其显示位置，替换掉之前的图像。为了避免这种突然的改变，可以淡入view，或者让多个Drawable交叉淡入，而这些都需要使用TransitionOptions完成。
下面是一个简单的运用。

```java
import static com.bumptech.glide.load.resource.drawable.DrawableTransitionOptions.withCrossFade;//哪种变换导入相应包
Glide.with(fragment)
    .load(url)
    .transition(withCrossFade())//交叉变换
// .transition(withNoTransition())//不发生变换
    .into(view);

```

## 4.RequestBuilder

使用 RequestBuilder 可以指定：

加载的资源类型(Bitmap, Drawable, 或其他)、加载的资源地址(url/model)、最终加载到的View、想应用的（一个或多个）RequestOption 对象、想应用的（一个或多个）TransitionOption 对象、想加载的缩略图 thumbnail()。

即Glide也可以设置这些东西，下面进行介绍。

**创建对象：**

要构造一个 RequestBuilder 对象，可以通过先调用 Glide.with()然后再调用某一个 as 方法来完成：

```java
RequestBuilder<XXX> requestBuilder = Glide.with(fragment).asXXX();//XXX为资源类型如Drawable、Bitmap等来指定资源类型
```

或先调用 Glide.with() 然后 load()：

```java
RequestBuilder<Drawable> requestBuilder = Glide.with(fragment).load(url);
```

RequestBuilders 是特定于它们将要加载的资源类型的。默认情况下会得到一个 Drawable RequestBuilder ，但可以使用 as… 方法来改变请求类型。

**增加条件：**

创建完对象后增加条件

```java
requestBuilder.apply(requestOptions);
requestBuilder.transition(transitionOptions);//上面两个参数就是先前介绍的选项
```

**缩略图 (Thumbnail) 请求**
Glide 的 thumbnail() API 允许指定一个 RequestBuilder 来与你的主请求并行启动。
thumbnail() 会**在主请求加载过程中展示**。如果**主请求在缩略图请求之前完成，则缩略图请求中的图像将不会被展示。**thumbnail() API 允许简单快速地加载图像的低分辨率版本，并且同时加载图像的无损版本。

```java
Glide.with(context)
  .load(url)
  .thumbnail()//thumbnail里可以传一个再次进行Glide，即创建RequestBuilder对象。也可以传一个浮点数，表示将原图缩小n（n属于0到1）倍（不推荐，因为这样等待时是一片空白）
  .into(view);
```

## 5.请求优先度

包括：

```java
Priority.LOW
Priority.NORMAL
Priority.HIGH
Priority.IMMEDIATE
```

方式就是.priority()传参

## 6.组件选项

大概知道它是可以自定义就行

## 7.转换（Transform)

### 1.内置变换

内置变换有：

CenterCrop、FitCenter、CircleCrop

### 2.多重变换

可以.transform(new CircleCrop(),new CenterCrop(),new FitCenter() )这样子多重运用



## 补充listener

```java
 listener(new RequestListener<Drawable>() {
                     @Override
                     public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                         if (e != null) {
                             for (Throwable throwable : e.getRootCauses()) {
                                 Log.e("GlideError", "Glide load failed", throwable);
                             }
                         }
                         return false; // 重要的是返回 false，这样 Glide 才会继续尝试加载图片
                     }

                     @Override
                     public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                         return false; // 通常不需要在这里做任何处理，除非你有特殊的资源准备后的逻辑
                     }
                 }).
```



# ————————————————————————

# 注意横线内的内容除了4都可以是RequestOption的内容

## 8.目标

目标是into里加载的资源，平时传进去imageview时会被封装成target。在单纯加载不满足是可以自己进行重写target。

**1.target**

```java
Target<Drawable> target = 
  Glide.with(context)
    .load(url)
    .into(new Target<Drawable>() {
      ...
    });
Glide.with(context).clear(target);
lide.with(context).clear(target);

//与加载到imageView类似
Glide.with(context)
  .load(newUrl)
  .into(imageView);
Glide.with(context).clear(imageView);
```



**2.SimpleTarget**

```java
SimpleTarget target = new SimpleTarget<Bitmap>( 250, 250 ) {  
    @Override
    public void onResourceReady(Bitmap bitmap, GlideAnimation glideAnimation) {
        imageView.setImageBitmap( bitmap );
      
    }
};
/*动画资源
Glide.with(context)
  .load(url)
  .into(new SimpleTarget<>() {
    @Override
    public void onResourceReady(Drawable resource, Transition<GifDrawable> transition) {
      if (resource instanceof Animatable) {
        resource.start();
      }
      // Set the resource wherever you need to use it.
    }
  });



*/
 Glide.with( context.getApplicationContext() ) // safer!
      .load( url )
      .asBitmap()
      .into( target );


```

**3.ViewTarget**

Glide 并不支持加载图片到自定义 view 中，所以可以使用ViewTarget 来实现。

```Java
CustomView customView = (CustomView) findViewById( R.id.custom_view );

    viewTarget = new ViewTarget<CustomView, GlideDrawable>( customView ) {
        @Override
        public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> glideAnimation) {
            //view.setImag是自定义view里自己去实现的方法
            this.view.setImage( resource.getCurrent() );
        }
    };

    Glide
        .with( context) 
        .load( url )
        .into( viewTarget );

```

**以上target在使用完后要clear，避免一些内存问题**

## 9.缓存

默认情况下，Glide 会在开始一个新的图片请求之前检查以下多级的缓存：

1.活动资源 (Active Resources) - 现在是否有另一个 View 正在展示这张图片
2.内存缓存 (Memory cache) - 该图片是否最近被加载过并仍存在于内存中
3.资源类型（Resource） - 该图片是否之前曾被解码、转换并写入过磁盘缓存
4.数据来源 (Data) - 构建这个图片的资源是否之前曾被写入过文件缓存

前两步检查图片是否在内存中，如果是则直接返回图片。
后两步则检查图片是否在磁盘上，以便快速但异步地返回图片。

如果四个步骤都未能找到图片，则Glide会返回到原始资源以取回数据（原始文件，Uri, Url等）。

磁盘缓存策略
DiskCacheStrategy 可被 diskCacheStrategy 方法应用到每一个单独的请求。

DiskCacheStrategy.NONE -------------什么都不缓存
DiskCacheStrategy.DATA ------------- 仅仅只缓存原来的全分辨率的图像。
DiskCacheStrategy.RESOURCE -----仅仅缓存最终的图像，即，降低分辨率后的（或者是转换后的）
DiskCacheStrategy.ALL ----------------缓存所有版本的图像
DiskCacheStrategy.AUTOMATIC ----让Glide根据图片资源智能地选择使用哪一种缓存策略（默认选项）

**跳过磁盘缓存**

```java
Glide.with(context)
  .load(url)
  .diskCacheStrategy(DiskCacheStrategy.NONE)
  .into(view);
```

**跳过内存缓存**

```java
Glide.with(context)
  .load(url)
  .skipMemoryCache(true)
  .into(view);
```

**仅从缓存加载图片**

```Java
Glide.with(context)
  .load(url)
  .onlyRetrieveFromCache(true)
  .into(imageView);
```

**清理缓存**

```Java
Glide.get(context).clearDiskCache();//只能在子线程执行
Glide.get(context).clearMemory();//只能在主线程执行
```

