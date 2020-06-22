# Questions

*repository URL: https://github.com/ishanknijhawan/Questions.git*
*gist ID: 4771e2b719d987ecc7f512a904fa8445*
## Ques1. How video streaming from URL works. What is HLS ?
Answer: When someone streams a video online, he/she connects to the server where that video is stored. The client connects to a URL where that video is stored, and the server starts sending packets, i.e. server sends picture frames to the client's video player. And as soon as the server has sent enough frames to the client to form a scene, the video starts playing on the client side. In fact, the server always sends extra frames to keep the remaining frames in buffer so that when the client has an unstable internet connection, there are always few extra frames to be played in that time limit when the client's internet is slow. When the extra frames are also utilized and the client is not being able to receive more packets due to low bandwidth, client sees a buffering screen. 

Now, people might upload the video on the server with different formats and there are so many video formats available. And its difficult for the video player to keep track of videos of different formats. Therefore there are some fixed formats for web streaming like smooth streaming (by Microsoft) and HDS (by Adobe). But the most commonly used are **HLS (HTTP Live Steaming)** and **DASH (Dynamic Adaptive Streaming over HTTP)**. After the video is uploaded, there is served side processing of the video into these formats. 

HLS is a format which is developed by Apple. Rather than delivering the video in one go, HLS splits it into smaller segments which are contained within the MPEG-2 transport streams. Each segment has about 10 seconds duration with the extension *.ts*.H264 has to be used as the video codec. 

## Ques2. How is video played in Android app ?
Answer: Android has it's own native Mediaplayer class which helps in playing videos. It supports almost all audio and video formats, but the main drawback is that it's not very customizable. For that, android has what it's called *Exoplayer*. Youtube and google's many other apps which supports video streaming use Exoplayer. 
Exoplayer is extremely customizable unlike media player, which is just a black box. It also supports dynamic streaming over HTTP, and smooth streaming of the video played. 
## Ques3. How Youtube changes video quality based upon internet connection
Answer: Youtube compresses the data while storing so that it is able to differentiate between different data sizes of the same data, and store it in different sizes in different places across their servers. With the help of data compression, we can save a single piece of data in various sizes. Now when some user streams that video, youtube is able to determine the internet speed of that user by calculating the rate at which their data packets are being transferred. It then uses some basic conditional base code as shown below to determine which quality packets the server has to send the user.

 - 0 to 10kbps : Auto 144p
 - 10 to 50kbps : Auto 240p
 - 50 to 100kbps : Auto 360p
 - 100 to 500kbps : Auto 720p
 - 500kbps+ : Auto 1080p 

So, unless the video quality explicitly specified by the user, youtube follows these algorithms to dynamically adjust the video quality. 
## Ques4. How does Firebase database works 
Answer: Firebase database (or Firebase cloud Firestore) are based upon NoSQL databases. NoSQL is different from SQL (or MySQL or SQLite) database. In SQL we have to define database which store one or more records (which are in the form of tables). Each record has some specific fields and every record must adhere to the same fields. 

For example, we have a products table which has the following attributes
| product_id |description  |	price| tax|
|--|--|--|--|
|1|milk|20|2.5
|2|chips|10|1.75
Now, suppose we want to have a product which also includes a field named quantity, then we can't simply insert that record without adding that field to the entire table. We can assign that field as *null* to rest of the elements, but then that would very acquire a lot of useless space. 
**NoSQL**, on the other hand has a completely different schema, and Firebase firestore database has a schema very similar to the NoSQL database. Here, the records are stored as *documents* which are maintained inside a collection. Unlike SQL, a collection can contain documents with different fields. For example we have a chatApp, then there would be one collection named **Users** which stores all the user data, and the other collection can be **Chats** which stores every message as document with the fields are senderID, receiverID, message, time, etc. Suppose a user wants to add a an image to his message, then we don't need to add message field in all the documents which do not have image attached. We could simply update that particular document with the an image URL. A NoSQL database stores everything as a **JSON** tree.  

![Capture](https://user-images.githubusercontent.com/45118110/80212647-80d4e700-8655-11ea-979c-4543cb1bfa24.PNG)

An example of Firebase NoSQL database with a Chat and users collection and a document with some fields. 
## Ques5. How sensex API works 
APIs like displaying stocks, sensex, real time news coverage all use Server Side Events or SSE to keep track of changes in real time, see answer to Ques9.
## Ques6.  Explain whatsapp system design
*This answer is summary of [this](https://www.youtube.com/watch?v=vvhC64hQZMk) video.* 
A chat application should have the following features that i'll be going to discuss as well as to how these features are implemented. 

 1. One to one messaging
 2. Group messaging
 3. sent/delivered/read receipts status
 4. online/last seen feature
 5. Image/video sharing
 6. Permanent and temporary chats

The user connects through the app by something we call as a **Gateway**. Whenever any activity happens, the gateway is notified about that. We shall discuss all the above mentioned features in detail.

 ### One to one messaging
 Suppose there are two users, user A and user B. when user A sends a message, it is then updated in gateway A. Now, there can be two ways in which the message from gateway can be sent from gateway A to gateway B (and then to user B). First is, we could store the information of *which user is connected to which box* in the gateway itself. So, when A sends message to B through gateway A, this gateway has stored that information as to which box this sender and receiver belongs to. But, this is a very expensive task as occupies a lot of memory in the gateway box which could've been used in storing more TCP connections coming from the sender. Not just that, this information is duplicated in all the gateways and hence consuming even more memory. 

So, the second way is that we want to have that TCP connection as a dumb connection, i.e. when gateway A receives a message, it doesn't know to whom it belongs, and it always redirects that message to a so called **microservice**. This microservice stores the information which every gateway was storing in the last case, i.e. who is connected to which box. Now when A sends a message using *sendMessage(B)* (where B is the userID of user B), gateway A forwards it to the microservice. And when this step is done, user A receives a **sent** status or a single tick inside his/her app.  The microservice then sends this message to gatewayB using the information stored inside it.

Now, the gatewayB can't send this message directly to user B because the server are supposed to be *'client to server'* only, and not vice versa. So, there are many ways in which user B can receive the message. One way is **long polling** over HTTP, in which user B asks the gateway every minute or so whether there is any new message or not. And if there is, it receives from the gateway server. But as this is a chat application, we want the message to be sent in realtime. For that, we can **web sockets** over TCP itself. Web sockets help users with peer to peer communication without the need of any Client server in between. For more information regarding sockets, refer Ques 9. So, using web sockets user A can send message directly to user B.  

### Group messaging
 
 ### sent/delivered/read receipts status
 As we discussed earlier how user A sends message to B. We read that as soon as the message is sent to the microservice by the gateway A, user A is updated with a **sent** status. Now, when the micro service sends the message to gatewayB, it waits for B to ask whether there is a message or not (using long polling method), and soon as user B has received a message, user A is updated with a **delivered** status. And as you might have guessed, as soon as B open the chat window of the sender in his/her app, user A is updated with a **seen** status. 
 ### online/last seen feature
 This feature is quite straight forward. Let's say we want to display the last seen of user A in user B's application. Then we make a microservice which tracks user's activity. So, whenever user A closes the app, this microservice will be updated with the user A's timestamp, which will be displayed in user B's app. Now, if user A has opened the app but hasn't closed it, we should show user A as **online** instead of last seen. Also, we should set a threshold after user A closes the app. Let's say it's been only 3 seconds that user has closed the app, now user B should not be shown that user A has last seen 3 seconds ago, instead it should show online only. So, ideally we can set a last seen threshold of about 20 seconds. Another point, there are times when the app is working in the background and is continuously in touch with the gateway. So we should differentiate this interaction with the user's interaction with a flag variable. When user is interacting, set flag to say *user request*. Else set flag to *app request*, and update last seen only when it is a user request flag.
 ### Image/video sharing
 ### Permanent and temporary chats
 Apps like instagram have a permanent chat feature, i.e. if a user has deleted the app and reinstalls it, all the chats will be restored because all the messages were stored in the cloud, Whereas, apps like snapchat have a temporary chat feature where the messages are destroyed as soon as user closes the app. Apps like whatsapp also have some sort of temporary messages, because if user A and his user B both have deleted the app then those messages are permanently deleted. All the messages are locally stored inside the app and there is no cloud storage unless the user manually prepares for a chat backup. 
## Ques7. How whatsapp compresses images
Answer: There are basically two types of compression techniques:
- lossless
- lossy

Whatsapp uses lossy image compression technique to reduce the size of images to a great extent. Consider an 12MP file with a resolution of around 3000x4000 pixels and a file size of around 5.9MB. Whatsapp reduces it to a file size of mere 220KB, with a resolution of 960x1280 with a 50% compressioin rate. If the image already has this resolution, whatsapp doesen't scale it because it is already in its minimum resolution which is set by whatsapp. I tried sending a file of size 960x1280 and it kept the same. Let's see a higher level algorithm of how whatsapp compresses images: 

    function compressImage(image){
	    var originalHeight <- *image height*
	    var orignalWidth <- *image width*
	    var maxHeight <- 960
	    var maxWidth <- 1280
	    var imgRatio <- actualWidth/actualHeight
	    var maxRatio <- maxWidth/maxHeight
	    
	    if(actualHeight > maxHeight or actualWidth > maxWidth){
		    if(imgRatio < maxRatio){
				 imgRatio <- maxHeight/originalHeight
	            originalWidth <- imgRatio * originalWidth
	            originalHeight <- maxHeight   
		    }
	    }
	    else if(imgRatio > maxRatio){
		    imgRatio <- maxWidth/originalWidth
            originalHeight <- imgRatio * originalHeight
            originalWidth <- maxWidth
	    }
	    else{
		    originalHeight = maxHeight
		    originalWidth = maxWidth
	    }
	    //image with updated resolution
	    var updatedImage = originalWidth x originalHeight
    }

 
## Ques8. What is deeplink ?
## Ques9. Difference between web sockets, long polling and Server Side Events (SSE). 
- Long polling means sending repeated requests to the server, as each new incoming connection is established, the HTTP headers are parsed, a query for new data is performed, and a response is generated and delivered. But rather than having to repeat this process multiple times for every client until new data for a given client becomes available, long polling is a technique where the server chooses to hold a client’s connection to be open for as long as possible, thus delivering a response only after data becomes available or a timeout threshold is reached. 

- On the other hand, web sockets provide a persistent connection between the client and the server, they are built around **TCP/IP** connection. In this, unlike long polling, the client doesn't have to repeatedly send request to the server. The WebSocket protocol enables interaction between a web browser and a web server with lower overhead than half-duplex alternatives such as HTTP polling, providing real-time data transfer from and to the server. Web sockets have advantages in scenarios in chat application like whatsapp where realtime updation of data is greatly preferred. 

> *Web sockets require user to handle lots of exceptions which were taken of in HTTP long polling on it's own*
 1. Server side events or SSE provides the mechanism that allows the server to push data to the client asynchronously as soon as the client-server connection is established.  Since SSE is based on HTTP, it has a natural fit with HTTP/2 and can be combined to get the best of both: HTTP/2 handling an efficient transport layer based on multiplexed streams and SSE providing the API up to the applications to enable push. Therefore it is greatly preferred over long polling and web sockets for building real time apps like sensex display, stock update, Covid19 updates, etc. 
## Ques 10. Difference between Main/UI Thread and background Thread. 
In android, the main thread corresponds to all the task the are running on the UI of the application. The main thread is also known as UI thread because all the UI components like button clicks, textviews, populating a recycler view, Activity lifecycle and other UI components happens on the Main Thread/UI Thread. 
Now, any other task which can take more than 5 seconds to complete should happen on the background thread. Otherwise it is blocking all the task on the main thread. For example, let's say we have an app in which we fetch data from an API and display that in a Textview inside our app. Now, if we run both the tasks on the main thread, our app will crash. This happens because the data from the web is extracted separately from the main thread and once it is fetched, then it should load inside the text view or a list. Till then, a progress bar can be shown inside that UI component.

## Ques 11. How does PIP (picture in picture) works in android applications ?


## Ques 12. Briefly explain the following terms in android
### 1. Activity
An activity is a single, focused thing that a user can do. It's like frames in java. It generally occupies the whole screen of an Android Device. Unlike the `main()` method in other programming language, an Android app always starts with an Activity. An activity provides the window in which the app draws its UI. This window typically fills the screen, but may be smaller than the screen and float on top of other windows. Generally, one activity implements one screen in an app
### 2. Fragment
A fragment is similar to an activity but it doesn't necessarily occupy the whole screen of an app, unlike an activity. For example, in *Whatsapp*, there are three table below the action bar viz *chats, status and calls.* These tab corresponds to *fragments* of an app because there is no new screen opening when we are moving from chats screen to calls screen. The tabs and search bar remain at their places and and bottom portion of the screen just slides smoothly to a new section. 
This is not the case when you open someone's chat, however. It then opens a fresh new screen and all the tabs and that search bar is now gone. This is the main key difference between an activity and a fragment. 
### 3. Activity lifecycle
When the user navigates through the app, in and out of different activities, the `Activity` instances in your app transition through different states in their lifecycle. Through the different lifecycle callback methods, the user can configure different tasks in different stages of an app. For example, we have a chat app and we want to show when the user is online. We can tell our app that as soon as the user open the main activity, make online variable true, and when the activity pauses, make online false. Or suppose we have a music app, in which we want to convert that song into a sticky notification as soon as user minimised the app, we can do this from the following lifecycle callback methods.

`onCreate()` 
*In the onCreate() method, you perform basic application startup logic that should happen only once for the entire life of the activity.*

`onStart()` 
*The onStart() call makes the activity visible to the user, as the app prepares for the activity to enter the foreground and become interactive*

`onResume()` 
*When the activity enters the Resumed state, it comes to the foreground, and then the system invokes the onResume() callback. This is the state in which the app interacts with the user. The app stays in this state until something happens to take focus away from the app.*

`onPause()` 
*The system calls this method as the first indication that the user is leaving your activity (though it does not always mean the activity is being destroyed); it indicates that the activity is no longer in the foreground (though it may still be visible if the user is in multi-window mode).*

`onStop()` 
*When your activity is no longer visible to the user, it has entered the _Stopped_ state, and the system invokes the onStop() callback. This may occur, for example, when a newly launched activity covers the entire screen. The system may also call onStop() when the activity has finished running, and is about to be terminated.*

`onDestroy()` 
*The system invokes this method when either of the following 2 things happen*

 - *the activity is finishing (due to the user completely dismissing the activity or due to finish().*
 - *the system is temporarily destroying the activity due to a configuration change (such as device rotation or multi-window mode)*

<img src = " https://static.javatpoint.com/images/androidimages/Android-Activity-Lifecycle.png">

### 4. Retrofit
Retrofit is a type safe HTTP client developed by Square, the company who also developed libraries like Dagger, OkHTTP and Picasso. Retrofit models REST endpoints and converts them into Java interfaces. Using retrofit, in addition wirh OkHttp, we can asynchronously fetch the data from an API in JSON format and convert it using GSON, and then we can assign that data into a textview or a listview. 

### 5. Dagger2
Suppose we have a class, and it takes parameters as objects which are also objects of some other class. For example, there is a class `Car()`, which takes 2 parameters, namely `Engine()` and `Wheels()`. Then the `Engine()` and `Wheels()` are so called dependencies of `Car()`. Now suppose `Engine()` class further depends upon say. `Piston()` and `Cylinder()` class. Then, if we have to initialize the `Car()` class, we have to initialize every other class in order to fill in the parameters of the `Car()` class. This can be very tedious and memory consuming. Therefore Square developed this library called Dagger, a dependency injection library, which injects dependencies at runtime and the user doesn't have to take care of all the values and parameters in order to satisfy the dependencies of `Car()` class.  
### 6. RxJava (doubt)
RxJava is a library which simplifies development focused around threading. A developer need not worry too much about the details of how to perform operations that should occur on different threads. For example, if we want to make a network call using `AsyncTask()` , we have to

 1. Create an inner `AsyncTask()` subclass in our Activity/Fragment
 2. perform the network operation in the background
 3. take the result of that operation and update the UI in the background. 
 
 This code, might look simple, but can cause certain bugs namely memory leak. Also, if we want to chain another long operation after network call, it wouldn't be possible. On the contrary, RxJava uses a different approach. In the RxJava world, everything can be modeled as streams. A stream emits item(s) over time, and each emission can be consumed/observed.

> *The three O's of RxJava: Observable, Observer and Operator*

- An Observable is the stream abstraction in RxJava. It is similar to an **Iterator** in that, given a sequence, it iterates through and produces those items in an orderly fashion. 
- The next component to the Observable stream is the Observer (or Observers) subscribed to it.
- Items emitted by an Observable can be transformed, modified, and filtered through Operators before notifying the subscribed Observer objects.

### 7. Kotlin Coroutines
In short, Kotlin Coroutines helps to simplify the code which runs asynchronously. In Android, Coroutines help to solve 2 primary issues: 

- Manage long running tasks that might freeze the app if run on main/UI thread.
- provide safe calling network or disk operations from main thread.

If your app is assigning too much work to the main thread, the app can seemingly freeze or slow down. Network requests, JSON parsing, reading or writing from a database, or even just iterating over large lists can cause your app to run slowly enough to cause slow or frozen UI that responds slowly to touch events. These long-running operations should run outside of the main thread.

Kotlin Coroutines use **Dispatchers** to determine which threads are used for Coroutine execution. To run code outside of the main thread, you can tell Kotlin coroutines to perform work on either the _Default_ or _IO_ dispatcher. In Kotlin, all coroutines must run in a dispatcher, even when they're running on the main thread.

Kotlin provides 3 Dispatchers in use: 

 1. `Dispatchers.Main`- use this Dispatcher to run the code on the main Android thread
 2. `Dispatchers.IO`- This dispatcher is optimized to perform disk or network I/O outside of the main thread
 3. `Dispatchers.Default`- This dispatcher is optimized to perform CPU-intensive work outside of the main thread.

### 8. SQLite Database/Room Persistence
SQLite Database is a mobile version of the MySQL database. SQLite helps to store data locally inside the application, so that when app is destroyed, the data is still persisted. It has all the functions which a normal SQL has like *SELECT, INSERT, UPDATE, DELETE, ORDER_BY, FILTER, etc.* 
Room is also a database library, which is part of Android architecture components. According to developers.android.com, *The Room persistence library provides an abstraction layer over SQLite to allow for more robust database access while harnessing the full power of SQLite.*
Room helps to create a Cache of your app, and store the key frame of the app's data (just like Firebase snapshot). Room has all the functionality of SQLite and some of the common operation like INSERT, DELETE and UPDATE are per defined inside the library. It reduces a lot of code which has to written in the SQLite database. 
Room, when combined with tools like Viewmodel, Repository, Livedata and Kotlin Coroutines can be a very efficient method to implement inside and app and to improve app's performance. 
### 9. Design Patterns
Design patterns are reusable solutions to common software problems. They can be broadly classified into three types:

 1. **Creational Patterns**
 - *Builder*
 Builder pattern separates the construction of a complex object. in this way, the same construction process can create different representations. In Android, the Builder pattern appears when using objects like  `AlertDialog.Builder`
 
 - *Dependency Injection*
 Please refer to part 5 of the same Question for the answer. 
 
 - *Singleton*
 The Singleton Pattern specifies that only a single instance of a class should exist with a global point of access. This works well when modeling real-world objects only having one instance. The Kotlin `object` keyword is used to declare a singleton, without the need to specify a static instance as in other languages.
 
 2. **Structural Patterns**
 - *Adapter*
An adapter lets two incompatible classes work together by converting the interface of a class into another interface the client expects. For example, RecyclerView extends the Adapter class to bind the view inside it. 
 
 - *Facade*
 Please refer to part 4 of the same Question for the answer.
 
 3. **Behavioral Patterns**
 -  *Command*
 The Command pattern lets you issue requests without knowing the receiver. You encapsulate a request as an object and send it off; deciding how to complete the request is an entirely separate mechanism.
 
-   *Observer*
The Observer pattern defines a one-to-many dependency between objects. When one object changes state, all of its dependents are notified and updated automatically. This is a versatile pattern; you can use it for operations of indeterminate time, such as API calls. You can also use it to respond to user input. The *RxAndroid* framework (aka Reactive Android) will let you implement this pattern throughout your app. 

-   *Model View Controller*
Also knows as MVC, it refers to the current reigning presentation architectural pattern across several platforms; it’s particularly easy to set your project up in this way on Android. It refers to the three divisions of classes used in this pattern:

1.  **Model**:  your data classes. If you have  `User`  or  `Product`  objects, these “model” the real world.
2.   **View**:  your visual classes. Everything the user sees falls under this category.
3.   **Controller:**  the glue between the two. It updates the view, takes user input, and makes changes to the model.

Dividing your code between these three categories will go a long way toward making your code decoupled and reusable. Additionally, moving as much layout and resource logic as possible into Android XML keeps your View layer clean and tidy.

-   *Model View ViewModel*
This architectural pattern is similar to the MVC pattern; the Model and View components are the same. The ViewModel object is the *glue* between the model and view layers, but operates differently than the Controller component. Instead, it exposes commands for the view and binds the view to the model. When the model updates, the corresponding views update as well via the data binding. Similarly, as the user interacts with the view, the bindings work in the opposite direction to automatically update the model.

-   *Clean Architecture*
Clean Architecture exists at a higher abstraction level than the MVC and MVVM presentation architecture patterns. It describes the overall application architecture: how the various layers of an app (business objects, use cases, presenters, data storage, and UI) communicate with one another.

### 10. Memory leak
A memory leak happens when a code allocated memory for an object, but never deallocates it. When a memory leak occurs the Garbage Collector thinks an object is still needed because it’s still referenced by other objects. But those references should have cleared. It can be caused by many reasons like memory leak from Thead, Handler, Asynctask, Singleton, etc.

### 11. Gradle build systems
In Android Studio, Gradle is used to build our application projects. Before Android Studio, in Eclipse we used to compile and build the applications using command line tool which was soon taken over by GUI based steps to build and run Android Applications in eclipse using ANT. Every android application development tool has to compile resources, java source code, external libraries and combine them into a final Android Package Kit (APK). 

> ***Gradle** is a build system, which is responsible for code compilation, testing, deployment and conversion of the code into `.dex` files and hence running the app on the device.*

There are 2 gradle files inside every android project, one is project level and other is application level. In the build process, the compiler takes the source code, resource, external libraries, JAR files and the `AndroidManifest.xml`(this contains all the meta data of the) file, and converts them into a `.dex` file, which includes bytecode. Then *APK Manager* combines the `.dex` files and all other resources into single **apk** file. 

### 12. Threads
### 13. Threadpool
### 14. Queries
 Query is a part of `androidx.room.Query`, is is given by `@Query` annotation.
As an extension over SQLite bind arguments, Room supports binding a list of parameters to the query. At runtime, Room will build the correct query to have matching number of bind arguments depending on the number of items in the method parameter. For example: 

```Java
@Query("SELECT * FROM song WHERE id IN(:songIds)")
 public abstract List<Song> findByIds(long[] songIds);
```
For more information about room, refer Ques. 8

### 15. Performance optimization
Making an android app is not a very challenging task, the thing that makes it challenging is how fast and responsive the app behaves at user's end. Below are some of the Performance optimization tips that should be kept in mind while designing an Android application: 

1. **Avoid Creating Unnecessary Objects**
As you allocate more objects in your app, you will force a periodic garbage collection, creating little *janks* in the user experience. The concurrent garbage collector introduced in Android 2.3 helps, but unnecessary work should always be avoided. 

2. **Prefer Static over Virtual**
If you don't need to access an object's fields, make your method static. Invocations will be about 15%-20% faster. It's also good practice, because you can tell from the method signature that calling the method can't alter the object's state.

3. **Use static `const val` for constants**
Using the `const val` directly, we can send that variable directly into forming a `.dex` file rather than forming an instance of it while compiling. 

4. **Use enhanced for loop syntax**
The enhanced `for` loop (also sometimes known as "for-each" loop) can be used for collections that implement the `Iterable` interface and for arrays. With collections, an iterator is allocated to make interface calls to `hasNext()` and `next()`. With an `ArrayList`, a hand written counted loop is about considerably faster, but for other collections the enhanced for loop syntax will be exactly equivalent to explicit iterator usage.

5. **Avoid using Floating point**
Floating point is about 2 times slower than integer on Android devices. In speed terms, there's no difference between `float` and `double` on the more modern hardware. Space-wise, `double` is 2 times larger.

### 16. Profiling
Android Profiler is a tool which measures performance of an application in real time. The Android Profiler tools provide data to help us understand how our app uses CPU, memory, network, and battery resources. To open the **Profiler** window, choose **View > Tool Windows > Profiler** or click **Profile**  in the toolbar. 	It continues to fetch the data in the form of a shared timeline until the user disconnects manually. The whole screen looks something like this. 

<img src="https://i.pinimg.com/originals/06/c6/97/06c697758639b88d6376cebc9938d061.png"/>

### 17. Debugging
Android Studio provides a debugger that allows you to do the following and more:

-   Select a device to debug your app on.
-   Set breakpoints in your Java or Kotlin code.
-   Examine variables and evaluate expressions at runtime.
#### Start Debugging
1. Select some breakpoints in your code
2. Select a device to debug from the dropdown list of devices available (connect via USB or enable an AVD)
3. In the toolbar, click **Debug**. The output should look something like this, 
<img src="https://developer.android.com/studio/images/debug/debugger_2x.png">

### 18. API
 
**A**pplication **P**rogramming **I**nterface, or API in short, is a software intermediary that allows two applications to talk to each other. When you use an application on your mobile phone, the application connects to the Internet and sends data to a server. The server then retrieves that data, interprets it, performs the necessary actions and sends it back to your phone. The application then interprets that data and presents you with the information you wanted in a readable way. This is what an API is - all of this happens via API. 

The API also provides a level of security, your smartphone's data is never fully exposed to the server, and likewise the server is never fully exposed to your phone. Instead, each communicates with small packets of data, sharing only that which is necessary. An API usually contains data in the form of a JSON and it is converted by tools such as GSON to make is usable for the software. An example API can be [Covid 19 India API](https://api.covid19india.org/data.json)


## Ques 13. What Dynamic programming. Explain how software like auto spell checker and google maps make us of this.  
Answer: Dynamic programming (DP in short) is an algorithmic technique for solving an optimization problem by breaking down into simpler sub problems and taking advantage of the fact that solution to overall problem depends upon the optimal solution of its sub problems. 
In DP, we  store the sub solutions in a matrix which come in handy when computing the solutions of problems which come further after in the recursive procedure. A very good example of this Fibonacci series in which nth Fibonacci number can be computed using concept of Dynamic programming. When using plain old recursive procedure in calculating nth Fibonacci number, it recomputes multiple values in the tree which are already computed. So, instead of going down the trees for computing the values which are already computed, we can store them in a matrix and check on runtime before computing a value that whether this value is already being computed or not. Doing this, we are reducing the time complexity to a significant amount. 
Dynamic programming has many other applications, two of the very important applications have been discussed below:

- ### Spell Checker: 
Spell checkers make suggestions for misspelled words. Given a misspelled string, a spell checker return the words in the dictionary which are closest to the misspelled string. The main concept behind them is what we call as a **Levenshtein distance** or Edit distance. It follows similar approach to a dynamic programming, stores all the cost required for insertion, deletion and substitution for each letter with every other letter between two words. And based on that distance, it recommends the words which are nearest to input word. For example, while typing in a smartphone keyboard, we often see suggestion before we even type the whole word. We also see corrected spelling for any given word in out text input. 

- ### Google Maps
Google maps approach to dynamic programming is pretty smart. My guess would that it stores be all the path values (distances between two paths) and the real time traffic values as a solution of the sub problem and whenever the user  wants to know the shortest distance, it has all the values pre loaded. And based upon the real time traffic it tells which path the user should follow. 

## Ques 14. Explain Trie data structure.
Answer: Trie (also known as digital tree or prefix tree), is an ordered tree data structure which is used to store dynamic set of arrays where the keys are usually strings. Simply saying, a Trie is a tree like data structure wherein each node stores a character and we can search, insert or delete the strings by traversing in that Trie. This improves the time complexity of a string search operation drastically. Because we store the strings in a tree like dictionary and ignore the repeated ones. For example, there are two strings *pqrs* and *pqrt*. Then instead of storing these 2 separate strings in an arraylist or or 2D array, we can store these in a Trie as follows. 

- Store p as the root element. Check if q exists, if yes then don't add. If no then insert this character as the child of p. Similarly check r and s. So we now have a tree with nodes p, q, r and s with q being child of p, r being child of q and s being child of r. 
- Now, for the second string, we check if p already exists. In this case, yes it is the root of this element, so we move to next element q, which is also present in the string pqrt. We check for r, it's present as well, and at last we check for t, which is not present. So we make t as a child of r and sibling of s. This way, we stored a total of 5 characters in the tree by ignoring the repeated characters. 
- But, what if the first character is different in a string, well in that case we redesign out Trie DS. We set an empty character as our root node and all the nodes which were previously the root nodes are now the child of this empty character root node. 

![enter image description here](https://miro.medium.com/max/1000/1*-KWorUiWCwn-a5iGw2chZg.jpeg)

For more details, refer to [this](https://medium.com/basecs/trying-to-understand-tries-3ec6bede0014) article. 

## Ques 15. What is Graph QL ?
Answer: GraphQL is a syntax which is generally used to load data from a server to a client. With **GraphQL**, the user is able to make a single call to fetch the required information rather than to construct several REST requests to fetch the same. GraphQL is usually a string query that is sent to server to be implemented. That query when called, returns a JSON format to the client. When compared with the **REST**, it is much more flexible and solves many of the shortcomings and inefficiencies that developers experience when interacting with REST APIs.
Instead of making multiple requests for multiple endpoints like in REST API, user can simply write one single query describing all the needs and requirements. Let’s consider a simple example scenario: In a blogging application, an app needs to display the titles of the posts of a specific user. The same screen also displays the names of the last 3 followers of that user. How would that situation be solved with REST and GraphQL? 
### Using REST
With a REST API, you would typically gather the data by accessing multiple endpoints. In the example, these could be `/users/<id>` endpoint to fetch the initial user data. Secondly, there’s likely to be a `/users/<id>/posts` endpoint that returns all the posts for a user. The third endpoint will then be the `/users/<id>/followers` that returns a list of followers per user.

![enter image description here](https://imgur.com/VRyV7Jh.png)

### Using Graph QL
In GraphQL on the other hand, you’d simply send a single query to the GraphQL server that includes the concrete data requirements. The server then responds with a JSON object where these requirements are fulfilled.

![enter image description here](https://imgur.com/z9VKnHs.png)



## Ques 16. What is conversational AI and how it works. Explain Amazon Alexa.
Answer: 

## Ques 17. What is block chain ? why so fuss around it ?

## Ques 18. What is 5G, IOT and why it is important to know about it ?

## Ques 19. What is System Design, why it is required ?

## Ques 20. What is Caching ?
