Redis Session Manager for Apache Tomcat
=======================================

Overview
--------

An session manager implementation that stores sessions in Redis for easy distribution of requests across a cluster of Tomcat servers. Sessions are implemented as as non-sticky--that is, each request is able to go to any server in the cluster (unlike the Apache provided Tomcat clustering setup.)

Sessions are stored into Redis immediately upon creation for use by other servers. Sessions are loaded as requested directly from Redis (but subsequent requests for the session during the same request context will return a ThreadLocal cache rather than hitting Redis multiple times.) In order to prevent collisions (and lost writes) as much as possible, session data is only updated in Redis if the session has been modified.

The manager relies on the native expiration capability of Redis to expire keys for automatic session expiration to avoid the overhead of constantly searching the entire list of sessions for expired sessions.

Data stored in the session must be Serializable.

This project supports  Tomcat 7,jedis 2.5.1,and redis sentinel.

Architecture
------------

* RedisSessionManager: provides the session creation, saving, and loading functionality.
* RedisSessionHandlerValve: ensures that sessions are saved after a request is finished processing.

Note: this architecture differs from the Apache PersistentManager implementation which implements persistent sticky sessions. Because that implementation expects all requests from a specific session to be routed to the same server, the timing persistence of sessions is non-deterministic since it is primarily for failover capabilities.

Usage
-----

Add the following into your Tomcat context.xml (or the context block of the server.xml if applicable.)

    <Valve className="cp149.github.com.RedisSessionHandlerValve" />
	<Manager className="cp149.github.com.RedisSessionManager"
             host="localhost" <!-- optional: defaults to "localhost" -->
             port="6379" <!-- optional: defaults to "6379" -->
             database="0" <!-- optional: defaults to "0" -->
             maxInactiveInterval="60" <!-- optional: defaults to "60" (in seconds) --> />

The Valve must be declared before the Manager.

To support sentinel.
```html	
<Valve className="cp149.github.com.RedisSessionHandlerValve" />
<Manager className="cp149.github.com.RedisSessionManager"
		 masterName="mymaster" 
         host="localhost"
         port="26379" 
         database="2"
         maxtotal="10"
         maxInactiveInterval="60"/>
```

Copy the tomcat-redis-session-manager.jar,commons-pool2-2.2.jar and jedis-2.5.1.jar files into the `lib` directory of your Tomcat installation.

Reboot the server, and sessions should now be stored in Redis.

Acknowledgements
----------------

The architecture of this project was based on the tomcat-redis-session-manager project found at 
https://github.com/jcoleman/tomcat-redis-session-manager
