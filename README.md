# Behind the Scenes - DevByte Viewer App

n this lesson, I learned how to implement offline caching by building an app that lets users watch DevByte vides. This is in lesson 9 of [Android App Development in Kotlin course on Udacity](https://classroom.udacity.com/courses/ud9012)


Highlights:
- Offline cacheing 
- Creating Room
- Creating Repository 
- Using WorkManager

----
### What is Cache?
Caching means storing something for future use. In computer science when we say cache we often mean to cry data closer to the place it will be used, so it can be accessed faster. 

 In a mobile app we will cache data in the file system as well. Every android app has access to a folder it can use to store files and data. 

#### Network Caching:
- Cache results per query
- HTTP caching 
- Store network results on disk 

We can implement a cache the way our web browser does in our android app using Retrofit. We can configure Retrofit to do this caching for us, and it’s a really good idea to turn that on for production apps. 

However, most apps need to implement a more sophisticated type of caching.

#### Cache Invalidation 
Cache invalidation means knowing when the data in a cache is not correct anymore. It may have changed on the server or perphaps it’s just been in the cache so long it started to grow new features. Either way, the data is too old to use from the cache, so we have to throw it away. 


The best way to store structured data in an offline cache and give you control over how data is stored and invalidated is to use a SQL database. Instead of hosting the database on a server somewhere, you keep it locally on the device file system. 

Review on Room:
- Room is a library for working with database on Android. 
- When using Room, you save and load Kotlin data classes to a local database, which is perfect for an offline cache.

Using SharedPreferences in this scenario is not good because it’s limited to storing keys and values, and it isn’t a great way to store a lot of data or structured data. Compared to a Room database, it won’t let you do complex queries against your offline cache. 

#### Files
Another option to implement an offline cache is to use files. You can access your apps folder and write any file or files you want to it. It’s private to your app and it will be cleared when your app is uninstalled. This is great if you have specific need that can be solved by a file system, like saving in an image or a data file. But it means you’re going to be managing files yourself, which solves one problem by introducing another. 

So, Room is a great solution to local data caching.

----

### How to store data

![Application Code](https://user-images.githubusercontent.com/43662326/154152567-cb40d8b1-f9b7-40cd-ad35-796df4cd82e6.png)

When we add an offline cache, we want to show results from the database instead. This is a really key concept to adding an offline cache to your app. Always show content from the database instead of the network. When a network result comes in, instead of showing it right away, write it to the database.

![Application Code](https://user-images.githubusercontent.com/43662326/154152610-af9f100f-b4c0-4d85-9ec7-1130c2aa7e59.png)

And the UI will then get notified about the database changes and refresh the content on screen. We will treat the database as the single source of truth.  That means everywhere in our app we’ll trust what’s in the database to be correct. Even though there will be other sources like the network, we won’t ever show those directly on screen. By always updating the database, then showing the new values from the database, we ensure that the offline cache is always up to date. That’s because the database is our offline cache. 

If there is no network, we can still load the last result from the database. Even better, by following this pattern, it’s really easy to spot the bugs in your offline cache during  development not when your get a report of strange data from a user. 

#### What happens when you need to change a value that’s stored in the database.
Say for example, you have a user profile and the user changes their name. You need to update the server, and you also need to update the offline cache somehow since it’s the single source of truth. 
You could do this by writing the new name to your local database, then trying to write it to the server. But this has a major problem. Say you update the cache, then try to tell the server about the change. The server may end up of sync. It may never get the updated data because the network is not available or the server may experience an error while saving. If that happens, your app will show incorrect data for the offline cache. 

To avoid the data getting our of sync, the easiest way to update values is to delay updating the offline cache until after the server lets you know the data is saved. Once you get the result from the server, you can write it to the offline cace. 

This way , you can be sure your offline cache always holds the correct data. This is similar to the idea of a rollback. By delaying the right to the offline cache, you can roll back or go back to the previous version if the server fails to save. 

---

### Decorating a Room

Reference link: https://classroom.udacity.com/courses/ud9012/lessons/c5e4185e-3e76-47fb-962e-ba27c21d36d7/concepts/bfe89b7e-0eab-4795-9776-17132d1a5bdc

Talks about conflict strategy, which tells the database what to do when it experiences a conflict, for example when getting data with the same id. In this case we can add on conflict to replace the existing data with the new one. 

Upsert is combination of update and insert. It is also known as onConflict update by some developers. 

----

### Building a Room

Reference link: https://classroom.udacity.com/courses/ud9012/lessons/c5e4185e-3e76-47fb-962e-ba27c21d36d7/concepts/bcacb6d3-d2d9-478b-ab68-2468e0ac22f7 

It’s a good practice to separate your network, domain, and database objects. This is an example of separation of concerns. By keeping them separate, you’re able to easily change any of them without impacting your entire app’s code.

---

### Add a DatabaseVideo Entity

Reference link: https://classroom.udacity.com/courses/ud9012/lessons/c5e4185e-3e76-47fb-962e-ba27c21d36d7/concepts/fc0ea15c-716d-46c2-9de2-312b75e99173

---

### Add the Video Dao

Reference link: https://classroom.udacity.com/courses/ud9012/lessons/c5e4185e-3e76-47fb-962e-ba27c21d36d7/concepts/4872f984-759b-48bb-bf95-d281b05b7f45

---

### Refactor the VideoDao 
Reference link: https://classroom.udacity.com/courses/ud9012/lessons/c5e4185e-3e76-47fb-962e-ba27c21d36d7/concepts/08f5fcf6-3e1b-4c82-aab9-3c7d3bd894c6
  
When we return a livedata, room will do the database query in the background for us. Even more, it’ll update the live data anytime new data is written to the table. 

Returning LiveData instead of a List. 

---

### Add the VideosDatabase

Reference link: https://classroom.udacity.com/courses/ud9012/lessons/c5e4185e-3e76-47fb-962e-ba27c21d36d7/concepts/b19843b5-a8fd-464e-86d0-dbdcced9c746

---
### Using a Room

Now we will be using the database for offline cache. To do that, we’ll use a pattern called the repository pattern, and it’s called that because it’s a repository of data. 

The repository will provide a unified view of out data from several sources. In our app, data will come from the network and the database to be combined in the repository. Users of the repository don’t have to care where the data comes from, it can come from the network, or the database, or some other source we’ll add later. It’s all hidden away behind the interface provided by the repository. 

The repository has one other responsibility. It manages the data in the cache. You may have noticed our room database doesn’t have any logic for handling the offline cache. It just has a method to add new values and another one to get all values. 

Well, the repository will have all the logic to take a network result and update the database to make sure it’s up to date, and this is how we get separation of concerns. Classes that use the repository don’t have to care how the cache is managed or even how it’s implemented. It can be stored anywhere from their perspective. They’ll never even know where the database or the network calls exist. 

They’ll just get a livedata that gets updated at the appropriate times. 

---

### Building our repository

Repositories are just regular classes, you don’t extend anything or use an annotation. They’re responsible for providing a simple API to our data sources. For our repository, we’ll need a video’s database, so we’ll ask users to pass it as a constructor parameter.  

This is a really simple form of concept called dependency injection. By taking a database object as a constructor parameter, we don’t need to keep a reference to Android context in our repository, potentially causing leaks. 

Reference link: https://classroom.udacity.com/courses/ud9012/lessons/c5e4185e-3e76-47fb-962e-ba27c21d36d7/concepts/73c8fc9e-fe51-4386-968d-2f84bec5c1fe

---

### Using the Repository 

Reference link: https://classroom.udacity.com/courses/ud9012/lessons/c5e4185e-3e76-47fb-962e-ba27c21d36d7/concepts/ebc2fa27-2f2a-434d-8638-52911f3b7bd9

---

### WorkManger for the background

In this section, we will learn how to use WorkManager to perform tasks when your app is in the background. WorkManager is a Jetpack library to manage work while your app may be in the background or even not running. 

Compared to coroutines or other ways to do work on Android, you don’t tell WorkManager how to do work, but rather you tell it under what constraints to do the work. 

Constraints includes things like:
- Phone is charging
- Device idle - user not using the device
- On wifi - has network
- And so on 

Reference link: https://classroom.udacity.com/courses/ud9012/lessons/c5e4185e-3e76-47fb-962e-ba27c21d36d7/concepts/ccd9b1c5-0a96-444e-8263-11ec8ab44c1b

---

### Create a Worker

Reference link: https://classroom.udacity.com/courses/ud9012/lessons/c5e4185e-3e76-47fb-962e-ba27c21d36d7/concepts/21422bfd-b52d-4162-badd-9c6d26d446c2
