# Android Architecture Components - [VIDEO](https://www.youtube.com/watch?v=FrteWKKVyzI)

## Main Focus

* Focus on fundamentals
* Play well with others
* Be opinionated
* Scale
* Reach

## Solutions

* [Android Architecture Guide](https://developer.android.com/topic/libraries/architecture/guide.html)
* [Architecture Components](https://developer.android.com/topic/libraries/architecture/index.html) library

### Architecture Components

* Lifecycles as 1st-class citizen
* Lifecycle-aware observables
* Lightweight ViewModel
* Object mapping for sqlite

#### Lifecycles

The problem with the activity lifecycle is that many times your callbacks get executed while the activity is in onStop or onResume so you have to have methods like listener.start() and listener.stop() so you can better handle the callback call.

* LifecycleOwner - A thing with a lifecycle
* LifecycleObserver - A thing that cares about lifecycles

The main idea is that now the Activities/Fragments have lifecycles and you can pass the lifecycle object to the component that is interested in. You can also use LifecycleAnnotations within your component to listen for certain the Android events so you can call different methods at different times.

The idea is to have Lifecycle Aware Components which know how to initialize / cleanup itself based on the provided lifecycle.

#### LiveData<T>

* An observable data holder
* Lifecycle aware
* Manages subscriptios automatically
* Simple start / stop semantics

##### Active / Inactive observers

* Active - in the started or resumed state
* Inactive - paused or stopped

#### Configuration changes

* Retain data across configuration changes

##### ViewModel class

* Hold data for Activity/Fragments
* Survives configuration changes
* DO NOT reference views because it will keep them 

Use the ViewModelProvides class to get an instance of the ViewModel for each activity. It will be retained accross configuration changes.

#### Persistence

* Boilerplate-free code
* SQLite
* Compile time verification

[Room](https://developer.android.com/topic/libraries/architecture/room.html) - object mapping library for SQLite.

### Architecture

The difference is that now there is an official guide on Android app architecture.

UI Controller <-> ViewModel <-> Repository <-> Data Sources

#### UI Controllers

* Activities & Fragments
* Observes the ViewModel
* Keeps the UI up-to-date
* Forwards user actions back to the ViewModel

#### ViewModel

* Prepares & keeps the data for the UI
* Includes LiveData, Observables etc.
* Survives confirguration changes
* The gateway for the UI controller

#### Repository

* The complete data model for the app
* Provides simple data modification & retrieval APIs
* Coordinates fetching, syncing, persisting etc. from different data sources

#### Data Sources

* API to data sources
* Retrofit for REST calls
* Room for local persistence
* External content provider from OS

#### Dagger

All of the layers should be provided from Dagger.









