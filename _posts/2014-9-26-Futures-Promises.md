---
layout: post
title: Futures, Promises, and Async Programming on the JVM
---

Futures are becoming a very hot topic in the JVM world.  I have asked many friends and collegues if they know what a future is and how to use it.  
The answers tend to be "sort of" with a few people using them but not knowing how or why, which of course leads to improper use.  I was not very
pleased with the current state of literature to help engineers understand futures better so I figured I would give it a shot.

#Why Futures?#
If you know about asynchronous programming, feel free to skip this section.
In typical programming, all the code you write gets executed serially and in one thread / execution context.  Each command is only executed AFTER the
prior command finishes executing. If you want to communicate with a separate system, the thread blocks/waits on receiving data.  You can use multiple 
threads to handle multiple requests simultaneously, but there are only so many threads an application can effectively use.  
Recently, event driven asynchronous programming became popular with the release of libevent.  With libevent (and similar libraries), code is 
executed as a response to IO events such as receiving data on a socket, reading from a file, etc...  What this means is that you can wait on responses
without blocking the thread that you are using to execute code.  If you are working on some sort of web application, it is important to not
have blocking code in your controllers.  Most web frameworks only allocate so many threads to handle controller code, so when you run out of threads 
your application stops handling requests properly.  In theory, you can handle waiting on thousands of socket events simultaneously without 
eating up many threads (see [C10k problem](http://www.kegel.com/c10k.html)).

Event driven programming is based executing code (callbacks) on specific IO events.  The question is, how do you properly develop in a mature 
object oriented language with all of these callbacks?

#What are futures?#
Futures are an object oriented methodology for encapsulating callbacks from IO driven events. Are you asking yourself "what in hell does that mean?" 
you are not alone.  Let's imagine that there is some library you are using that uses some event driven process to do web requests.  When the response
comes in, you want to manipulate the data somehow.  How would you imagine that you would interact with such library?  In javascript, you pass an 
anonymous function to the library.  It looks something like this:
{% highlight javascript %}
httpLibrary.get("www.meow.com", function(data, error) {
    //CODEBLOCK1
    //do some data manipulation here
});
//Other code here
{% endhighlight %}
But that won't fly in Java.  It would look something more like this:
{% highlight java %}
Future<Response> respFut = httpLibrary.get("www.meow.com");
respFut.onComplete((data) -> {
  //CODEBLOCK1
  //do some data manipulation here
});
//Other code here
{% endhighlight %}
What this is saying is do some get request on www.meow.com.  Keep executing code (other code here) without waiting for any response.  
When the response comes back, execute "CODEBLOCK1" in a different context.  This is what the kids call asynchronous programming.

#How to use futures?#
There are a a few good future libraries out now for Java.  I personally like the [scala.concurrent package](http://www.scala-lang.org/files/archive/nightly/docs/library/index.html#scala.concurrent.Future).  It comes with a lot of good functionality built in that is very easy to use.  I personally use the 
[play framework](https://www.playframework.com/) a lot.  Their controllers allow you to return a promise (I will get into that later) and the framework
itself will handle waiting for a future to complete and return the result to the requester.  

As I mentioned earlier, you can use an onComplete function to take some action when a future completes.  However, imagine that you are writing a function
that makes some web call using a future and you want to manipulate the response and return it to whoever is calling the function.  onComplete will not work because it
returns a void and whatever happens in the onComplete cannot leave the scope of the anonymous function.  You can code your function to wait for the
response, do the manipulations and return it, but then you are blocking whatever thread calls your function (which is bad, mmmkay?).  What you want to do is be able to
chain some function to be called when the response is ready and that function will return a future of some data type.  That's where map and flatMap
comes into play.  With map/flatMap functions, you pass in code to be executed when the response is ready.  Whatever your code returns gets wrapped into
a future.  Example:
{% highlight java %}
public Future<String> getDataStripSpaces() {
  Future<String> dataFut = httpLibrary.get("www.meow.com");
  Future<String> replaceFut = dataFut.map(response -> {
    return response.replaceAll(" ", "");
  });
  return replaceFut;
}
{% endhighlight %}


