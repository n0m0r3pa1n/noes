# It's 21st century - Stop using EVENTBUS!

Do you still use it!? Have you used it? I mean the EventBus. If the answer is **NO**, then you are either a new developer who started Android recently or you had the wonderful opportunity to not meet this three-headed monster. What I am talking about? Let's make a brieft visit of the Android development history.

## Android Development History

I will skip some of the periods in the Android app development as I don't remember exactly how things escalated from one thing to another.

At the beginning of my career there was [RoboSpice](https://github.com/stephanenicolas/robospice). If you don't know, RoboSpice uses a background service to execute the network requests and then delivers them back to the main thread using callbacks.

[Retrofit](https://github.com/square/retrofit/) emerged half an year later after RoboSpice. And believe it or not, you could make them work together. It was a really fancy thing. 

There were also other networking libraries like [ION](https://github.com/koush/ion) , [Android Async Http Client](https://github.com/loopj/android-async-http) and even the crazy thing named [Spring for Android](http://projects.spring.io/spring-android/). 

So there was enough choice for all devs to switch from one to another networking library and see what the effect is. But do you know what the common thing with all libraries was:

**You had to use the fuckin callbacks again and again!**

And this was shit. Every programmer with some common sense towards Android would feel like something was wrong. Why? Because of the wonderful Android lifecycle handling. You would make a call inside your fragment like:

```java
public void onResume() {
	Ion.with(getActivity())
		.execute("http://whatever.shitty.endpoint")
		.(new Callback() {
			if (!isAdded()) {
				return;
			}
			// Do whatever
		});
}
```

Do you see the problems in the above code?

- The `if` check was everywhere, in every request you made so you had  to check whether the fragment/activity is there when the request is completed 
- You have tight coupling between Activity/Fragment and your network layer
- This makes your code practically untestable for unit tests

Things that also didn't help include:
- Noone thought of MVP/MVVM then, it was quite unpopular and there were some attempts to use MVC in Android, but it wasn't looking good at all.
- Google was quiet as shit. There was [one lecture](https://www.youtube.com/watch?v=xHXn3Kg2IQE) about application architecture and it was so complicated that noone would bother using it
- Volley was introduced to solve the problem. But it required so much boilerplate for a single request that it made the problem bigger and not easier. I was a fan of it. Then I stopped being lame and moved to Retrofit.
- People were still making AsyncTasks that execute requests with custom Http clients. Why bother using something custom when it creates "a ton of shit" and we can create our own shit!?

And  then EventBus was made.

## EventBus to the rescue

The first EventBus library made specially for Android was [Otto](https://square.github.io/otto/). It made a huge impact on the poor Android developers that were struggling between Services, Loaders and a very complicated lifecycle management. It made a huge impact on me. And there was the idea that:

> Android is event-driven! Event-	driven code you shall write!

### EventBus Life Saver

You have this central shared object which is of type EventBus. You can use it to post events on it. Every class can listen for a specific type of event that is posted on this event bus. That's all. Simple and straightforward.

Example:

```java
// MyActivity

@Override
public void onCreate(SavedInstanceState state) {
	 toolbar.setNavigationOnClickListener(() -> { 
		ApiService.getInstance().loadAppDetails();
	});
}

@Override
public void onAttach(Activity activity) {
	super.onAttach(activity);
	BusProvider.getInstance().register(this);
}

@Override
public void onDetach() {
	super.onDetach();
	BusProvider.getInstance().unregister(this);
}

@Subscribe
public void onAppDetailsLoaded(LoadAppDetailsApiEvent event) {
	populateAppDetails(event.getBaseApp());
}
```

How it actually works is:

- It calls ApiService to make a request
- When the request is done it posts an event on the EventBus of type LoadAppDetailsApiEvent that contains the data
- If the Activity is not destroyed, it will receive the event from the bus
- Otherwise, it will be unregistered and it won't receive the event

What a wonderful approach, isn't it?

And it actually solved the problems we had:

- The problem of the `if` checks if the fragment is added - SOLVED. Now you just post an event and don't need to check anything. The component will be unregistered if the user presses back for example.
- You have "loose" coupling between you components. They can work separately. You move the networking logic to another class, UI listens for events.
- You can have multiple components listen for the same event that happens. So if you have a button as a custom view and press it, all the instances of that button will be updated too when the event is posted.
- You could even get bigger separation in your classes. You could post an event like "GetAppsApiEvent" to which the ApiService listens and then posts "AppsLoadedApiEvent" to which the fragment listens.
- And it even allowed you to post events between threads. You do code like:
```java
// This now magically works passing events between multiple threads
// You could post event from Service and receive it in the UI
Bus eventBus = new Bus(ThreadEnforcer.ANY);
```

Sounds good, right? So many problems fixed with one event bus instance. Then [GreenRobot EventBus](https://github.com/greenrobot/EventBus) was released. Hey, who would release another event bus if the first wasn't working, right? And there was a boom with these event bus stuff.

> Two years later, after the first version of the app was released, the team was now 10 Android developers. It was growing fast because product owners wanted things delivered fast. The only solution was to grow the team. Noone knew how the app worked. Events were posted everywhere for everything. That's when hell broke loose.

## Event Everything

I had the close opportunity to start an app from scratch, grow it a lot and see how this approach does not work. [Here](https://github.com/n0m0r3pa1n/apphunt_android/blob/master/app/src/main/java/com/apphunt/app/event_bus/events/) is the result. Do you see how many UI events are there? How many API ones? It is crazy, right? Well, when we started it, it sounded like a good idea. An app based on events with separate classes listening to them. But how separate are the classes? Do they allow you to test them? How about in terms of unit tests? Let's see why this approach is very very bad.

### Bye Unit Tests

It is very hard to mock the event bus. And if you have it as a part of your networking or whatever  components, it is quite impossible to test them following the unit test manner. You could do integration testing but what time would it take you to do it? And how would you detect if a problem occurs, in which code layer exactly it happened?

### Hello Hours Spent on Onboarding New Dev

Imagine a new developer joins. Imagine you join a project with 50 events, some UI, some API and you need to find where these events are posted and where they are received. You will need your Sherlock Holmes skills with you because it will take you some days.

### Welcome Reflection

Now the event buses use compile time annotations. Then, they were using reflection. Althought the impact wasn't huge, it was there and it was not cool. You cannot depend on reflection to work on different devices.

### Freedom to Write Shit

It just allows you to write shitty code. Why should you pass data through bundles when you can post an event with it? Why should you call the APIService when you can post an event to it to it to load data? Why do you need interfaces and callbacks and things like `setTargetFragment()` when events can do this with several lines of code?

Because you want your code to be maintainable, unit tested and to add features in several days, not several weeks. You need to easily follow what happens through the code and event bus prevents you from doing that. It does stimulate you to add "one more event".

## Let's Close the Page

The cool thing about event bus approach in Android was that at first sight it offered flexibility and "loose" coupling with the price of only adding a single library and nothing more. It looked so perfect, such a lust to developers, you could get an app with "real-time" updates in no time. Just post an event. Most developers would close their eyes to the growing count of useless classes.

But now we have [RxJava](https://github.com/ReactiveX/RxJava), [Dagger](https://google.github.io/dagger/). We have tons of examples with [MVVM / MVP / MVC](https://academy.realm.io/posts/eric-maxwell-mvc-mvp-and-mvvm-on-android/) / [VIPER](https://gist.github.com/marciogranzotto/1c96e87484a17bd914e159b2604e6469) / [Clean](https://academy.realm.io/posts/clean-app-design-with-architecture-components/). We have [Google Architecture Components](https://developer.android.com/topic/libraries/architecture/index.html), [LiveData](https://developer.android.com/topic/libraries/architecture/livedata.html), [DataBinding](https://developer.android.com/topic/libraries/data-binding/index.html) and even some architectural examples from [Google themselves](https://github.com/googlesamples/android-architecture). I think we have enough tools to achieve way better things than using event bus.

The reason I am writing this post is because I hear more and more often that people use event-driven architecture in their apps. And not ordinary people, most of them are working for big companies that grow so fast. Now, I am part of a team of 17+ Android devs working in a single project. And everyone of us is afraid to touch the legacy code in the app. It, too, is even-driven. And it is a real mess 3 years after the project was started.

Please, if you haven't stopped using it, may be it is time. You can spend your time using an architecture that would split the business logic from your UI classes without needing to post anything on an event bus. And unit test that logic. Just, lets turn that page and check the event bus pattern as tried, but doesn't work. May be this article will help the next person who is thinking of trying the "event-driven" approach. :)
