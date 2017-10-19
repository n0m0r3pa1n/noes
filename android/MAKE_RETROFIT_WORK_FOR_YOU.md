# Making Retrofit work for you - [PRESENTATION](http://jakewharton.com/making-retrofit-work-for-you-ohio/)

## Http Client

Retrofit is not an HTTP client. It:
- Prepares requests
- Makes requests
- Lets someone else to execute them

## Building Two API classes

### Problem

```java
new Retrofit.Builder()
	.baseUrl("http://test.test")
	.client(new OkHttpClient()) // is always executed if you do not pass instance
	.build();
```

So having separate Retrofit builders will create separate OkHttpClients. It is better to share the OkHttpClients because:
- Disk caching
- Connection Pooling
- OkHttpRouting - the best route to talk to the server
- The same connection no matter if the host talks HTTP2

### Fix

Extract the client to a variable and pass it to the builder; 

```java
OkHttpClient okHttpClient = new OkHttpClient();
new Retrofit.Builder()
	.baseUrl("http://test.test")
	.client(okHttpClient) // is always executed if you do not pass instance
	.build();
```

## Common OkHttpClient

The OkHttpClass has a method called `newBuilder()` that you can use if you rewant to use the basic stuff you have in your base OkHttpClient.

```java
OkHttpClient client = new OkHttpClient();

// Shallow copy of client
OkHttpClient clientFoo = client.newBuilder()
	.addInterceptor(new FooInterceptor())
	.build();

// Timeout specific to this client only
OkHttpClient clientBar = client.newBuilder()
	.readTimeout(30, SECONDS)
	.build();
```

## Using Headers for Specific Endpoints

The idea is to mark specific endpoints with a custom Header. Like this:

```java
interface ApiService {
	@POST("/login")
	@Headers("No-Authentication: true")
	Call<User> login(@Body LoginRequest loginRequest);
}
```

Then use an interceptor that looks for this header and decides to whether add the `Authorization` header to the current request.

## Converters

### Single Converter Instance

Share the converter instance as it has internal machinery helping it transform bytes to objects faster. Instead of having two `GsonConverterFactory` instances, have only one.

```java
// This is the most important part
Gson gson = new Gson();

// Reusable for all Retrofit.Builders
GsonConverterFactory factory = new GsonConverterFactory(gson);

new Retrofit.Builder()
	.baseUrl("http://test.test")
	.addConverterFactory(factory)
	.client(okHttpClient)
	.build();
```

Either single instance factory or gson. No need for single instance factory if Gson is a single instance.

### Multiple Converters

You can have multiple converters added one after another instead of splitting the endpoints to JSONService and ProtoBufferService for example.

```java
GsonConverterFactory gsonFactory = new GsonConverterFactory();
ProtoConverterFactory protoFactory = new ProtoConverterFactory();

new Retrofit.Builder()
	.baseUrl("http://test.test")
	.addConverterFactory(protoFactory) // asked first
	.addConverterFactory(gsonFactory) // asked second
	.client(okHttpClient)
	.build();
```
Every converter knows what type it can handle. The orders matters. So the protoFactory is asked:

- Can you handle User objects?
- The factory sees if the User.class extends from Proto.class
- Reponds to the request or proceeds to gsonFactory

#### GsonConverter vs XMLConverter

Both converters say yes to all the available types of data. What happens if we want to use both of them simultaneously. 

A solution would be to use our own ConverterFactory with custom annotations

```java
interface @XML {}
interface @Json {}
```

```java
Metsclass XmlorJsonConverterFactory extends ConverterFactory {
	final ConverterFactory xml = //..
	final ConverterFactory json = //..
	
	@Override
	public Converter<ResponseBody, ?> responseBodyConverter(Type type, 
		Annotations[] annotations, Retrofit retrofit) {
			// Loop over annotations & check for XML or Json.class
			// Call appropriate converter
	} 
}
```

`AnnotatedConverterFactory` can be used for the case, it is created by Jake.

#### Dealing with Metadata

```java
class Envelope<T> {
	Meta meta;
	List<Notification> notifications;
	T response;
}

interface Service {
	@GET("/user")
	Call<Envelope<User>> getUser();
}
```

If you want to escape the Envelope class and work directly with user you can simply user a custom converter again. Please check the video [here](https://youtu.be/t34AQlblSeE?t=1908) for further details.

What is does it simple steps:
- Get the Envelope class
- Calls the next converter to convert to the Envelope<T> class
- Returns the Envelope.response object

#### Empty to Null Converter

Checks if the response.contentLength() is 0 and returns null. This is good for APIs that return zero bytes content instead of passing null.

#### Create Custom Endpoints

You can use converters to simulate network calls but actually make a call and parse HTML data for example.

## Call Adapters

Call adapters deal with the execution of queries - async, sync, threading etc.

### The Call Object

```java
interface Service {
	@GET("/user")
	Call<User> getUser();
}
```

What actually happens is:

```java
Call<User> <- Call<User> <- OkHttp.Call (similar to Retrofit but returns bytes) <- Call.Factory (aka OkHttpClient)
```

Actually, the `Call<User>` is wrapped two times when the response is received from the server.  When the original call happens, OkHttp does it on a background thread. This is the place where the conversion happens from JSON -> Java objects. Retrofit needs a way to deliver the result to the Android main thread and that's why it wrapps it twice. The mechanism that does this is called a CallAdapter.

```java
Call<User> <- CallAdapter <- Call<User> <- OkHttp.Call <- Call.Factory (aka OkHttpClient)
```
 ### Built-in Call Adapters

- The RxJavaCallAdapterFactory allows us to return a different type in the interface than the Call<> class.
 
```java
new Retrofit.Builder()
	.addCallAdapterFactory(
		RxJavaCallAdapterFactory.create())
	.addCallAdapterFactory(new BuiltInFactory()) // Implicitly added!
	.build();

interface Api {
	@GET("/user")
	Observable<User> getUser();

	@GET("/friends")
	Call<User> getFriends();
}
```

The way it works is it asks sequentially the adapter factories.

### Custom Call Adapters

The idea is again to get the next call adapter and wrap it inside an inner one. You can add the `observeOn(AndroidSchedulers.mainThread())` inside the call adapter instead of adding it everywhere.

### Handling Error Responses

Because Retrofit returns the ErrorBody as bytes you may need to implement your own class that handles the different types of error codes and bodies. For exampe you can have:

```java
interface Call<R, E> {
	Response<R, E> execute();
	void enqueue(Callback<R, E> cb);
}

interface Callback<R, E> {
	void onResponse(Response<R, E> response);
	void onFailure(IOException e);
}

interface Response<R, E> {
	R body();
	E error();
}
```

This allows us to return a custom body for the error. Or you can even have something like:

```java
interface Callback<R, E> {
	void onSuccess(R body);
	void onError(E errorBody);
	void onFailure(IOException e);
}
```

## Mock Mode

One way to mock data is implement the Service interface and return mock data.

```java
class MockService extends Service {
	BehaviorDelegate<Service> delegate;
	NetworkBehavior networkBehavior = NetworkBehavior.create();
}
```

What BehaviorDelegate allows you to apply Http-like semantics in terms of timing and failure. It can return response after several seconds or occasionaly throws an exception like the network fails.
















