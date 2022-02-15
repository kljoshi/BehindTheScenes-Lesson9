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

——

### How to store data


