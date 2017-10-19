# Android Touch Framework

## How Android deals with touch

Every event is wrapped up as a MotionEvent. It has the event action (ACTION_UP/DOWN etc.) and additional metadata.

ACTION_DOWN - initialization of every new gesture that happens in the app. Every gesture is terminated in ACTION_UP.

## How events are dispatched through the app

1. Every event is first dispatched to the app activity. The framework calls dispatchTouchEvent() method of the activity.
2. Traversed from the activity to the views from the top down. From the main container of the activity to each child. It calls dispatchTouchEvent method.
3. It will float to the views until one of the views says it is interested in receiving details for the event.
4. If no one handles the event in the boolean onTouch method of the view the event is dispatched again from the bottom of the hierarchy to the top to the onTouchEvent method. onTouchEvent is the end of this chain.
5. OnTouchListener can be used instead of subclassing

## Touch process

* Activity.dispatchTouchEvent()
  * Always first to be called
  * Send event to root view attached to Window
  * onTouchEVent()
    * Called if no views consume the event
    * Always last to be called
* View.dispatchTouchEvent()
  * Sends event to listener first, if exists
    * View.OnTouchListener.onTouch()
  * If not consumed, processes the touch itself
    * View.onTouchEvent()

## ViewGroups

* Each viewgroup evaluates based on the touch location which child views represent this location. If there are more than one it will iterate through the children in the reverse order when they were added to the group. 
* ViewGroups can steal touch events from the children - onInterceptTouchEvent
* The child component can block the parent from intercepting the gesture by calling requestDisallowTouchIntercept on it. By default, if you set the flag it is only set for the current gesture. It should be used on gesture by gesture basis but use with care.

## Clicking a passive View inside an activity

*ACTION_DOWN* example:
Activity.dispatchTouchEvent -> ViewGroup.dispatchTouchEvent -> View.dispatchTouchEvent -> View.onTouchEvent -> ViewGroup.onTouchEvent -> Activity.onTouchEvent

*ACTION_MOVE*/*ACTION_UP*:
Activity.dispatchTouchEvent -> Activity.onTouchEvent  - the system skips the viewgroup and the view because no view was interested in the ACTION_DOWN event

## Clicking a button inside on activity

*ACTION_DOWN* / *ACTION_MOVE* / *ACTION_UP*:
Activity.dispatchTouchEvent -> ViewGroup.dispatchTouchEvent -> View.dispatchTouchEvent -> View.onTouchEvent - not calling ViewGroup.onTouchEvent and Activity.onTouchEvent because the button consumes the event

## Clicking a button inside a ScrollView

*ACTION_DOWN*:
Activity.dispatchTouchEvent -> ViewGroup.dispatchTouchEvent -> View.dispatchTouchEvent -> View.onTouchEvent

*ACTION_MOVE/ACTION_UP*:
Activity.dispatchTouchEvent -> ViewGroup.disptachTouchEvent -> ViewGroup.onTouchEvent -> Activity.onTouchEvent - it skips the View.dispatchTouchEvent because the gesture is detected by the ScrollView that it is connected with it and is not a simple tap. The button received ACTION_CANCEL that he will not receive the gesture. 

## ViewConfiguration class

Use it to determine different gestures. 
* getScaledTouchSlop() to determine when a touch becomes a drag. 
* getScaledMinimumFlingVelocity() to determine the min velocity of a fling. 
* getScaledPagingTouchSlop - touch slop used for horizontal paging gestures

## Forward events from one view to another

Take the motion event and just call dispatch on the target. Calling onTouchEvent directly is not a good idea, it will skip the chain of receivers.

## Custom Touch Handling Warnings

* Call through to super whenever possible
  * View.onTouchEvent - state management which you can lose
* ACTION_MOVE - protect it with slop checks
* Always handle ACTION_CANCEL - after CANCEL you will not receive anything else

## Multi-touch handling

Pointer count will be greater than one if it is multitouch. You can send a filter to the device to check how many fingers it supports on touch. Each finger is given a unique id and has an index. Index is for the finger count, the id stays unique for each finger.

## Batches

All events are sent in a batch. Every time you receive an event you are receiving the latest batch of events. You can use the getHistoricalX/Y() and other historical events to review every single event.

## System Touch Handlers

* Use GestureDetector and ScaleGestureDetector - they will do the math and tell you what events happen like onScroll or onSingleTapUp etc.
* Use OnTouchListener

## Touch Delegate

Allows a parent view to define a specific touch zone. Inside of that zone forward all touch events to that view.

## onInterceptTouchEvent vs onTouchEvent

* onTouchEvent - reports to the framework that this event and subsequent events in the gesture, you as a view want to handle

* onInterceptTouchEvent - it is only available on ViewGroups. As a ViewGroup you say that this current gesture, which is being handled by someone else and start handling it yourself.
