# Retrofit框架

## 1.简介

一个重新封装的okHttps，用于网络请求

## 2.环境准备

引入依赖：

```groovy
implementation 'com.squareup.retrofit2:retrofit:2.6.3‘//retorfit的依赖
implementation 'com.squareup.retrofit2:converter-gson:2.6.3'//GsonConverterFactory的依赖
implementation 'com.google.code.gson:gson:2.8.0'//Gson的依赖
```

## 3.基本用法

基本过程就是先创建一个**接口**，用**注释**写好网络需求，然后再在主程序使用

### 1.注释

#### 1.@GET、@Query、@QueryMap（获取）

```java
public interface Api {
    //get请求
    @GET("user")
    Call<ResponseBody> getData();
    //Call<ResponseBody> getData(@Query("id") long idLon, @Query("name") String nameStr);
    //Call<ResponseBody> getData(@QueryMap Map<String, Object> map);
    //Call<Data<Info>> getJsonData(@Query("sort") String sort, @Query("format") String format);
}
```

 1.@GET(参数)：参数内容是接在baseUrl【baseUrl在等下建好的retrofit中】后的。若baseUrl是https://baidu.com/的话，那么加上get，到时候请求的网址是https://baidu.com/user

2.第二种是给get增加条件的。long idLon中long是数据类型，idLon是变量名，括号里的id是键名，到时候会将idLon赋给它。后面那个类似。假设idLon是123，namStr是kkk，那么此时的网址变为https://baidu.com/users?id=123&name=kkk

3.第三种类似与第二种只不过此时传入的是表。用法如下所示：

```java
//主程序中
  Map<String, Object> map = new HashMap<>();
  map.put("id", 10006);
  map.put("name", "刘亦菲");
  Call<ResponseBody> call = retrofit.create(Api.class).getData3(map);
```

此时网址变为`https://baidu.com/users?id=10006&name=刘亦菲`

4.第四种用法是从所给网址扒下json数据

#### 2.@POST、@FormUrlEncoded、@File、@FileMap、@Body（发送）

1.

```java
public interface Api {
    @POST("user/emails")
	Call<ResponseBody> getPostData();    
}
```

参数作用与GET的第一种用法作用一致

2.

```java
public interface Api {
    @FormUrlEncoded
    @POST("user/emails")
    Call<ResponseBody> getPostData2(@Field("name") String nameStr, @Field("sex") String sexStr);
    //Call<ResponseBody> getPostData3(@FieldMap Map<String, Object> map);
}

```

这串代码的第一种用法参数作用与GET第二种用法一致，第二种用法与GET第三种用法一致。用法不同在这里的注释不同，query（querymap）变成了了field（或者fieldmap）和增加了FormUrlEncoded

3.

```java
public interface Api {
    @POST("user/emails")
	Call<ResponseBody> getPostDataBody(@Body RequestBody body);
}
```

- **@Body**             上传json格式数据，直接传入实体它会自动转为json，这个转化方式是GsonConverterFactory定义的。参数作用与GET的第四种方法一致

- **特别注意：@Body注解不能用于表单或者支持文件上传的表单的编码，即不能与@FormUrlEncoded和@Multipart注解同时使用，否则会报错，**我们找到@Body的源码发现，isFormEncoded为true或者isMultipart为ture时，满足其中任何一个条件都会抛出该异常。

  

#### 3.@HTTP

@HTTP注解的作用是替换@GET、@POST、@PUT、@DELETE、@HEAD以及更多拓展功能,起融合作用。

```java
public interface Api {
   @HTTP(method = "GET", path = "user/keys", hasBody = false)
    Call<ResponseBody> getHttpData();
}
```

1**.method**用来指明需求（GET|POST)

2.**path**与GET和POST的参数作用一致

3.**hasBody**表示是否有请求体，是个布尔值。

#### 4.@Path

```java
public interface Api {
  @GET("orgs/{id}")
  Call<ResponseBody> getPathData(@Query("name") String nameStr, @Path("id") long idLon);
}
```

- **@Path**           请求参数注解，用于Url中的占位符{}，所有在网址中的参数，上述代码中将用idLon代替{id}

#### 5.@Url

代替掉GET后的参数用

```java
public interface Api {
 @GET
Call<ResponseBody> getUrlData(@Url String nameStr, @Query("id") long idLon);
}
```



#### 6.@Header、@Headers

**@Header**：用于动态赋值请求头如：

```java
public interface Api {
  @GET("user/emails")
 Call<ResponseBody> getHeaderData(@Header("token") String token);
}
```

其中token可在运行时再赋值，Header可以同时添加多个，通过该注解添加的请求头不会相互覆盖，而是共同存在。。

**@Headers**：用于静态赋值请求头，如：

```java
public interface Api {
 @Headers({"phone-type:android", "version:1.1.1"})
 @GET("user/emails")
 Call<ResponseBody> getHeadersData();
}
```

Headers可以添加多个

#### 7.@Streaming

以**流的形式获取**文件。利于**大文件传输**

```java
public interface Api {
  @Streaming
  @POST("gists/public")
  Call<ResponseBody> getStreamingBig();
}
```

#### 8.@Multipart、@part、@PartMap（上传数据）

也用于**上传数据**

```java
public interface Api {
    @Multipart
	@POST("user/followers")
	Call<ResponseBody> getPartData(@Part("name") RequestBody name, @Part MultipartBody.Part file);
}
```

- **@Multipart**       表示请求实体是一个支持文件上传的表单，需要配合@Part和@PartMap使用，适用于文件上传

**@Part**            用于表单字段，适用于文件上传的情况，@Part支持三种类型：RequestBody、MultipartBody.Part、任意类型

**@PartMap**        用于多文件上传， 与@FieldMap和@QueryMap的使用类似

主代码写法

```java
//声明类型,这里是文字类型
 MediaType textType = MediaType.parse("text/plain");
//根据声明的类型创建RequestBody,就是转化为RequestBody对象
RequestBody name = RequestBody.create(textType, "这里是你需要写入的文本：刘亦菲");

//创建文件，这里演示图片上传
File file = new File("文件路径");
if (!file.exists()) {
   file.mkdir();
 }

//将文件转化为RequestBody对象
//需要在表单中进行文件上传时，就需要使用该格式：multipart/form-data
RequestBody imgBody = RequestBody.create(MediaType.parse("image/png"), file);
//将文件转化为MultipartBody.Part
//第一个参数：上传文件的key；第二个参数：文件名；第三个参数：RequestBody对象
MultipartBody.Part filePart = MultipartBody.Part.createFormData("file", file.getName(), imgBody);//其他（如partmap）不同在于
/*
 MultipartBody.Part filePart1 = MultipartBody.Part.createFormData("file1", file1.getName(), requestBody1);
    MultipartBody.Part filePart2 = MultipartBody.Part.createFormData("file2", file2.getName(), requestBody2);

    Map<String,MultipartBody.Part> mapPart = new HashMap<>();
        mapPart.put("file1",filePart1);
        mapPart.put("file2",filePart2);*/


Call<ResponseBody> partDataCall = retrofit.create(Api.class).getPartData(name, filePart);

```

## 4.主代码（举例获取数据）

### 1.添加网络权限

### 2.添加依赖

### 3.创建接收数据类

```java
public class Data<T> {
    private int code;
    private String message;
    private T data;

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
}

```

### 4.网络接口

```java
public interface Api {
    //get请求
    @GET("api/rand.music")
    Call<Data<Info>> getJsonData(@Query("sort") String sort, @Query("format") String format);
}

```

### 5.创建Retrofit

```java
    //构建Retrofit实例
    Retrofit retrofit = new Retrofit.Builder()
            //设置网络请求BaseUrl地址
            .baseUrl("https://api.uomg.com/")
            //设置数据解析器
            .addConverterFactory(GsonConverterFactory.create())
            .build();

```

### 6.创建网络请求接口实例

```java
//创建网络请求接口对象实例
Api api = mRetrofit.create(Api.class);
//对发送请求进行封装
Call<Data<Info>> dataCall = api.getJsonData("新歌榜", "json");

```

### 7.发送网络请求（同步/异步）

```java
//异步请求
dataCall.enqueue(new Callback<Data<Info>>() {
    //请求成功回调
     @Override
     public void onResponse(Call<Data<Info>> call, Response<Data<Info>> response) {
       }
    //请求失败回调
     @Override
     public void onFailure(Call<Data<Info>> call, Throwable t) {         
       }
  });

```

```java
//同步请求
Response<Data<Info>> data= dataCall.execute();

```

### 8.处理返回的数据

```java
 dataCall.enqueue(new Callback<Data<Info>>() {
     @Override
     public void onResponse(Call<Data<Info>> call, Response<Data<Info>> response) {
           Toast.makeText(MainActivity.this, "get回调成功:异步执行", Toast.LENGTH_SHORT).show();
           Data<Info> body = response.body();
           if (body == null) return;
           Info info = body.getData();
           if (info == null) return;
           mTextView.setText("返回的数据：" + "\n\n" + info.getName() + "\n" + info.getPicurl());
     }

      @Override
      public void onFailure(Call<Data<Info>> call, Throwable t) {
         Log.e(TAG, "回调失败：" + t.getMessage() + "," + t.toString());
          Toast.makeText(MainActivity.this, "回调失败", Toast.LENGTH_SHORT).show();
      }
 });

```

### 9.更新其他要求时只需要该接口就好。

## 注意点

<img src="Retrofit框架.assets/屏幕截图 2024-07-13 162509.png" alt="屏幕截图 2024-07-13 162509" style="zoom: 200%;" />

