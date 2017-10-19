# [Android Developer Toolbox](https://realm.io/news/mobilization-gautier-mechling-the-2016-android-developer-toolbox) - Notes

## Gradle

* Create different build variants which contain different code of your app
	* Internal - for the company. Embed a lot of tools to locate issues easily.
	* Production - only for customers

### U+2020 application

A demonstration of integration of different libraries. Read the [source code](https://github.com/jakewharton/u2020).

### IOSched app

The [iosched app](https://github.com/google/iosched) contains a separate debug screen which is visible only in the internal build.

### Example app

Check the [mobilization-2016 app](https://github.com/Nilhcem/mobilization-2016) for more details.

## Measuring tools

### Android Monitor tab

Use the monitor tab built in the Android Studio to track different measurements of your app - Cache, network, CPU, GPU etc.

### Android Studio Memory Leak Detection

Use the memory profiler to track memory leaks by making HRPOF dumps.

### [LeakCanary](https://github.com/square/leakcanary)

Detect memory leaks which happen in your app.

### [Takt](https://github.com/wasabeef/Takt)

See what is the frame rate of your app.

### [Hugo](https://github.com/JakeWharton/hugo)

Log how much time your methods execution take by using annotations.

### [Pidcat](https://github.com/JakeWharton/pidcat)

Use Pidcat to print the logs of your app. Easy to install and good design.

### [AndroidDevMetrics](https://github.com/frogermcs/AndroidDevMetrics)

Track the time needed for dependencies to be provided using Dagger.

## Code static analysis tools

### Lint

#### [SonarLint](http://www.sonarlint.org/)

More focused on the Java code than Android Lint.

#### [SonarQube](https://www.sonarqube.org/)

Continuous integration server. Tells you things like technical debt in days, count of code smells, tests coverage etc.

### [Error Prone](https://github.com/google/error-prone) (Google)

### [Infer](http://fbinfer.com/) (Facebook)

## Testing tools

### Mock the server

#### NodeJS + Express

#### [MockServer](http://www.mock-server.com/)

Use to return fake HTTP responses so testing can easily be done.

### [Hosts Editor](https://play.google.com/store/apps/details?id=com.nilhcem.hostseditor)

Associate a domain name to a different IP.

## HTTP Debugging

### [mitmproxy](https://mitmproxy.org/)

### [Fiddler](http://www.telerik.com/fiddler)

### [Charles Proxy](https://www.charlesproxy.com/)

Has debugging options with the option to replace values directly into the received response. Works great with Android. You can also simulate slow connection and repeat queries.

## Android State Restoring

### Don't keep activities

Option in the Developer options.

### [Fill Ram](https://play.google.com/store/apps/details?id=com.tspoon.androidtoolbelt)

Fills the RAM of your phone and the system begins clearing the available apps and later restores them. It is a good tool for QAs.

### Android Device Monitor - Stop Process

You can minimize your app by clicking the home button and the stop the process from the device monitor. Then the system will recreate the activities.

## Analyzing tools

### Developer Options

#### Show layout bounds

See layout positions and marign

#### Debug GPU overdraw

Check the view hierarchy you have and optimize it.

#### Animation Scale

Slow the animations in your app

#### Screencast + VLC

Make a screencast, open VLC and use the letter "E" to view the transition frame per frame.

### UIAutomatorViewer

Browse directly the view hieararchy and pick views by ids.

### [APKTool](https://ibotpeaches.github.io/Apktool/) + [Dex2Jar](https://github.com/pxb1988/dex2jar) + [JD-GUI](http://jd.benow.ca/)

Decompile APK and check its source code.

### [JADX](https://github.com/skylot/jadx)

### Android Studio - Build > Analyze APK

## Favorite tools

### [Stetho](http://facebook.github.io/stetho/)

Connect the phone to the PC and check the view hierarchy on the Chrome browser.

#### Stetho - Network 

Check the requests that happen and the responses received.

#### Stetho Dumpapp

Communicate with your app through terminal. You can display the content of the received response, the alarm manager scheduled intents etc.








