# Thread Context Handler

[![Build Status](https://travis-ci.org/imamchishty/thread-context-handler.svg?branch=master "thread-context-aspect")](https://travis-ci.org/imamchishty/thread-context-handler) [![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.shedhack.thread/thread-context-handler/badge.svg?style=plastic)](https://maven-badges.herokuapp.com/maven-central/com.shedhack.thread/thread-context-handler)

## Introduction

This library provides the ability to set context state which would enable better **logging** and **debugging**. 
The thread that is being executed will have the name set (context) with:

- **Id**: Unique Id. This is best suited for the Request Id rather than the Session Id. The session Id could be set in the context.

- **Date**: Date/time for the request.

- **Method**: Ideally this returns the fully qualified method name.

- **Context**: Context may contain items such as the htto method, the http request path, session Id etc.

- **Method params**: the name of the params will be set to ARGx, e.g. ARG0, ARG1

## ThreadContextHandler
 
[__ThreadContextHandler__](https://github.com/imamchishty/thread-context-handler/blob/master/src/main/java/com/shedhack/thread/context/handler/ThreadContextHandler.java) API for handling thread contexts. This will store the type T to the current thread.

Several implementations have been provided:

- [**JsonThreadContextHandler**](https://github.com/imamchishty/thread-context-handler/blob/master/src/main/java/com/shedhack/thread/context/handler/JsonThreadContextHandler.java) this stores the context as json string representation of a ([ThreadContextModel](https://github.com/imamchishty/thread-context-handler/blob/master/src/main/java/com/shedhack/thread/context/model/ThreadContextModel.java)). 

- [**ListThreadContextHandler**](https://github.com/imamchishty/thread-context-handler/blob/master/src/main/java/com/shedhack/thread/context/handler/ListThreadContextHandler.java) this stores a list (as a string).

- [**SimpleThreadContextHandler**](https://github.com/imamchishty/thread-context-handler/blob/master/src/main/java/com/shedhack/thread/context/handler/SimpleThreadContextHandler.java) this stores a simple string.

Each implementations could use a matching [**ThreadContextAdapter**](https://github.com/imamchishty/thread-context-handler/blob/master/src/main/java/com/shedhack/thread/context/adapter/ThreadContextAdapter.java) if you wish to provide a consistent API.
 
    public interface ThreadContextHandler<T> {

        /**
         * Sets the thread context based on the generic type.
         */
        void setThreadContext(T context);
    
        /**
         * Optional returns back the thread context.
         */
        Optional<T> getThreadContext();
    
        /**
         * Converts from a String to the type T.
         */
         Optional<T> convertFromString(String original) ;
    
        /**
         * Returns the raw value without any type conversions.
         */
        String getRawContext();
    
        /**
         * Method is called after the thread context has been set. Allows you to do some extra stuff with
         * the context, e.g. logging
         */
        void afterSettingThreadContext(String context, List<ThreadContextAfterSet> afterSetList);
    }
 
## Java 8 'Optional'
The eagle-eyed have probably already notice the use of 'Optional'. This provides a safe way to obtain (if available) the payload, type T, and thus preventing the dreaded Null Pointer Exception.
 
## ThreadContextAfterSet

Once a context has been set you have the ability to do something with it, for example logging. A handler can be 
passed a list which are called after the context has been set, please refer to [ThreadContextAfterSet](https://github.com/imamchishty/thread-context-handler/blob/master/src/main/java/com/shedhack/thread/context/handler/ThreadContextAfterSet.java)
[__LoggingAfterSet__](https://github.com/imamchishty/thread-context-handler/blob/master/src/main/java/com/shedhack/thread/context/handler/LoggingAfterSet.java) logs the context (String) to your desired log file at INFO level.
 
## ThreadContextAdapters
[**ThreadContextAdapter**](https://github.com/imamchishty/thread-context-handler/blob/master/src/main/java/com/shedhack/thread/context/adapter/ThreadContextAdapter.java) provides a consistent API wrapper for dealing with ThreadContextHandler.
As ThreadContextHandler uses generics a new ThreadContextAdapter implementation will be required to 'wrap' it up.
Several implementations have been provided, one for each of the ThreadContextHandlers.

## Examples

Creating the context handlers, adapters and a dummy service:

    // --------------------
    // ThreadContextHandlers
    // --------------------
    private JsonThreadContextHandler jsonHandler = new JsonThreadContextHandler();
    
    // --------------------
    // ThreadContextAdapter
    // --------------------
    private ThreadContextAdapter jsonAdapter = new JsonThreadContextAdapter(jsonHandler);
    
    // -----------
    // Foo Service
    // -----------
    private FooService fooJson = new FooService(jsonAdapter);

The code below shows how the most simplest way the context could be set using an adapter. FooService isn't aware
of the underlying implementation of the ThreadContextHandler.

    public class FooService {
    
        public FooService(ThreadContextAdapter adapter) {
            this.adapter = adapter;
        }
    
        final ThreadContextAdapter adapter;
    
        public int calc(int a, int b) {
    
            // Adapter could be called via a servlet, aspect. This example shows how
            // a servlet/adapter could use the adapter without knowing the underlying implementation of the
            // ThreadContextHandler. Values passed to the adapter have been hardcoded but in reality would be dynamically set.
            adapter.setContext("1234567890", new Date(), "com.shedhack.thread.context.adapter.FooService.calc",
                    new HashMap<String, Object>(){{put("session-id", "ABC");}},
                        new HashMap<String, Object>(){{put("ARG0", a); put("ARG1", b);}});
    
            return a+b;
        }
    }

Thread Context would be set to

    {id='1234567890', timestamp=Tue Apr 05 16:20:43 GST 2016, methodName='com.shedhack.thread.context.adapter.FooService.calc', params={ARG0=199, ARG1=200}, context={session-id=ABC}}


In the above example you'd probably wouldn't call the adapter from each method. It would be better to use a servlet or an aspect to do the same thing, but without intruding with your business logic. A thread-context-aspect project has been created which does exactly this (also available in Maven Central). Please refer to this project here:

https://github.com/imamchishty/thread-context-aspect

When the method is called the thread context is set, for example using JMC:

![alt tag](https://github.com/imamchishty/thread-context-handler/blob/master/resources/thread-name-image.png?raw=true "JMC threads list")

### Dependencies used

GSON is used by JsonThreadContextHandler, it creates an instance of GSON internally. The library is used to create a JSON string representation of ThreadContextModel, and also for doing the reverse.

	<dependency>
	    <groupId>com.google.code.gson</groupId>
	    <artifactId>gson</artifactId>
	    <version>2.6.1</version>
	</dependency>

Utility functions such as ToString rely on apache commons:

	<dependency>
	    <groupId>org.apache.commons</groupId>
	    <artifactId>commons-lang3</artifactId>
	    <version>3.4</version>
	</dependency>
	
SLF4J:

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.21</version>
        </dependency>

	
### Java requirements
Java 8+ is required due to the use of Optional. The code was built and tested using the Oracle JDK, for build details please click on the build image (Travis) at the top of the page.

### Maven repository

This artifact is available in [Maven Central](https://maven-badges.herokuapp.com/maven-central/com.shedhack.thread/thread-context-handler).
 
    <dependency>
        <groupId>com.shedhack.thread</groupId>
        <artifactId>thread-context-handler</artifactId>
        <version>x.x.x</version>
    </dependency>    

### Further reading

This project is perfect to use with servlets or aspects. A spring-aspect library has been created with annotations which makes the creation of thread contexts even easier.
Please refer to the project, ([thread-context-aspect](https://github.com/imamchishty/thread-context-aspect))

Contact
-------

	Please feel free to contact me via email, imamchishty@gmail.com
