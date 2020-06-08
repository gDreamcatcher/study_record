# Spring-boot-web-demo

[TOC]



## 返回restful接口

### 什么是REST
REST是**RE**presentational **S**tate **T**ransfer 的简称，是分布式超媒体系统体系的结构样式。和其他风格一样，REST也有6条基准原则：

**1. Client-server:** 通过将用户界面问题和数据存储分开， 可以提高用户界面在多个平台的可移植性，并通过简化服务器组件提高了系统的伸缩性；
**2. Stateless:** 无状态要求server从客户端接收到的每一条请求都要包含足够的信息，因此Session的状态全部保存在客户端；
**3. Cacheable:** 如果响应是可以缓存的，则客户端被允许将缓存的response返回给以后相同的request；
**4. Uniform interface:** 接口统一
**5. Layered system:** 分层系统
**6. Code on demand(optional):** REST允许通过下载和执行小程序或脚本形式的代码来扩展客户端功能。通过减少预先实现的功能数量，简化了客户端;

### Spring对REST的支持

```java
@RestController
public class RestDemoController {
	private static final String template = "Hello, %s";
	private final AtomicLong counter = new AtomicLong();

	@GetMapping("/name/{name}")
	public RestResponse getUserByName(@PathVariable("name")String name){
		if (!name.equals("test")) return RestResponse.success(new User(counter.incrementAndGet(), String.format(template, name)));
		else return RestResponse.fail(new TestException(String.format("test fail, name[%s] id not existed!", name)));
	}
}
```

其中

- `@RestController``
- ``@GetMapping("/name/{name}")`:等同于`@RequestMapping(name = "/name/{name}", method = RequestMethod.GET)` ,意思是将get方法的这个路径路由到这个方法；
- `@PathVariable`取uri路径中的参数，相似的还有`@RequestParam`, `@RequestBody` 

### 统一封装response的格式

在项目中，将response返回信息的格式进行统一封装，不仅有利于与前端统一处理接口的返回，而且具备良好的可读性。通常情况下项目中采用JSON(JavaScript Object Notation)序列化数据再进行传输，统一格式的response也就包含三部分信息：

1. 状态码：定义了成功、失败的标志，还可以包含一些内部定义的状态码，如对应用户名不存在时应该返回的状态码。注意此状态码与HTTP状态码无关。
2. 提示信息：以一段字符串补充说明状态码的具体信息。如果允许，前端还可以直接将该字段信息反馈给用户。
3. 携带信息：这部分是response中的关键，包含了用户正在希望请求的信息。特别的是，如果请求失败时，该部分通常为null或被隐藏。

#### 状态码和异常定义

生产中失败的状态码

```java
// 状态码的定义
public enum ResultCode {
	SUCCESS,
	FAIL,
	TOKEN_INVALID,
	ACCESS_DENIED
}
// 定义项目异常的基类
public class BaseException extends Exception{
	protected ResultCode code = ResultCode.SUCCESS;

	public BaseException() {
		super();
	}

	public BaseException(String message) {
		super(message);
	}

	public BaseException(String message, Throwable cause) {
		super(message, cause);
	}

	public BaseException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
		super(message, cause, enableSuppression, writableStackTrace);
	}

	public int getCode(){
		return this.code.ordinal();
	}
}

// 定义一个继承基类的异常，如参数异常
public class ParamException extends BaseException{
	public ParamException(String message) {
		super(message);
		this.code = ResultCode.FAIL;
	}
}

```

定义response类，里面有sucess方法和fail方法，返回的是一个response对象

```java
public class RestResponse {
	private int code;
	private String message;
	private Object data;

	public RestResponse() {	}

	public RestResponse(BaseException e) {
		this.code = e.getCode();
		this.message = e.getMessage();
	}

	public RestResponse(int code, String message, Object data) {
		this.code = code;
		this.message = message;
		this.data = data;
	}

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

	public Object getData() {
		return data;
	}

	public void setData(Object data) {
		this.data = data;
	}

	public static RestResponse success(){
		return new RestResponse(new BaseException(ResultCode.SUCCESS.name()));
	}

	public static RestResponse success(Object data){
		RestResponse res = new RestResponse(new BaseException(ResultCode.SUCCESS.name()));
		res.setData(data);
		return res;
	}

	public static RestResponse fail(BaseException e){
		return new RestResponse(e);
	}

	public static RestResponse fail(ResultCode code, String message){
		return new RestResponse(code.ordinal(), message, null);
	}

}
```

