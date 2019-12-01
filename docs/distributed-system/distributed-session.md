## Interview questions
How to implement distributed sessions during cluster deployment?

## Psychnological analysis of interviewers
The  interviewer asked you how to play a bunch of dubbo. You can play dubbo to make a monolithic system into a distributed system, and then after distribution, a bunch of problems, the biggest problem is **Distributed transaction**, **interface idempotency**, **distributed lock**, and the last one is **distributed session**.

Of course, the problems in distributed systems are more than that. They are very many and the complexity is high. Here are just a few common questions, but also a few frequently asked during interviews.

## Analysis of interview questions

What is session? The browser has a cookie. This cookie has existed for a period of time, and then each time a request comes, a special `jsessionid cookie` is brought. Based on this, a correspondiing session domain can be maintained on the server. Put some data.

In general, as long as you do not close your browser and the cookie is still present, the corresponding, session is there, but if the cookie is gone, the session is gone. It's common in things like shopping carts, and login status preservation.

Not much to say about this, anyone who knows Java should know this.

It's okay to play sessions like this on a monolithic system, but if you are a distributed system, where are so many services and where is the session state maintained?

In fact, there are many methods, but the following are commonly used:

### No session at all
Use JWT Token to store user identity, and then obtain other information from the database or cache. This doesn't matter which server the request is assigned to.

### tomcat + redis
This is actually quite convenient. The code that uses the session is the same as before, or based on tomcat's native session support. Then use a thing called `Tomcat RedisSessionManager` to let all the tomcats we deploy store session data. Just go to redis.

Configure in tomcat's configuration file:

```xml
<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />

<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
         host="{redis.host}"
         port="{redis.port}"
         database="{redis.dbnum}"
         maxInactiveInterval="60"/>
```

Then specify the host and port of redis.

```xml
<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />
<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
	 sentinelMaster="mymaster"
	 sentinels="<sentinel1-ip>:26379,<sentinel2-ip>:26379,<sentinel3-ip>:26379"
	 maxInactiveInterval="60"/>
```

You can also use the above method to save session data based on the redis high-availability cluster supported by the redis sentry, which is ok.

### spring session + redis
The second method mentioned above will be recoupled with the tomcat container. If I want to migrate the web container to jetty, do I have to reconfigure the jetty?

Because the above tomcat + rediis method is easy to use, but 

So now the better one-stop solution based on Java, which is spring. People's Spring basically contracts most of the frameworks we need to use, spring cloud is used for microservices, and spring boot is used for scaffolding, so using sping session is a good choice.

Configure in pom.xml:
```xml
<dependency>
  <groupId>org.springframework.session</groupId>
  <artifactId>spring-session-data-redis</artifactId>
  <version>1.2.1.RELEASE</version>
</dependency>
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>2.8.1</version>
</dependency>
```

Configure in the spring configuration file:
```xml
<bean id="redisHttpSessionConfiguration"
     class="org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration">
    <property name="maxInactiveIntervalInSeconds" value="600"/>
</bean>

<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <property name="maxTotal" value="100" />
    <property name="maxIdle" value="10" />
</bean>

<bean id="jedisConnectionFactory"
      class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory" destroy-method="destroy">
    <property name="hostName" value="${redis_hostname}"/>
    <property name="port" value="${redis_port}"/>
    <property name="password" value="${redis_pwd}" />
    <property name="timeout" value="3000"/>
    <property name="usePool" value="true"/>
    <property name="poolConfig" ref="jedisPoolConfig"/>
</bean>
```

Configure in web.xml:
```xml
<filter>
    <filter-name>springSessionRepositoryFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSessionRepositoryFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

Sample code:
```java
@RestController
@RequestMapping("/test")
public class TestController {

    @RequestMapping("/putIntoSession")
    public String putIntoSession(HttpServletRequest request, String username) {
        request.getSession().setAttribute("name",  "leo");
        return "ok";
    }

    @RequestMapping("/getFromSession")
    public String getFromSession(HttpServletRequest request, Model model){
        String name = request.getSession().getAttribute("name");
        return name;
    }
}
```


The above code is ok. Configure the redis to store session data for the spring session, and then configure a spring session filter. In this case, the session related operations will be handled by the spring session. Then in the code, use the native session operation, which is to get data from redis directly based on spring session.

There are many ways to implement distributed sessions. What I am talking about are just a few of the more common ones. Tomcat + redis is more commonly used in teh early days, but it will be recoupled into tomcat . In recent years, it has been implemented through spring sessions.







