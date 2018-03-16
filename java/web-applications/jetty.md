# Jetty

Eclipse Jetty is a Web server and Java servlet container. It's main advantages are it being very lightweight when compared to alternatives like Apache Tomcat or Wildfly, and the fact that it can be embedded into applications instead of having to run as a separated process.


## Setup

After downloading Jetty from https://www.eclipse.org/jetty/download.html, the first thing we can do is running the demo files provided with the package: to do this, just go to the `demo` directories, and start Jetty from there. However, it may happen that the default configuration provided with the downloaded package doesn't allow to run Jetty from other directories. To fix this, delete `start.ini`, and launch Jetty with the options `--create-startd` and `--add-to-start=jsp,http,webapp,deploy` to build a proper configuration.

As a Java servlet container, the same Jetty instance can server different applications under different Web paths. If everything we want to do is just running a single application under the root path, however, we should just provide our application inside a single `ROOT.war` file, or `ROOT` directory, located in the `webapps` directory of the Jetty installation (symlinks are usually employed here). In particular, to only serve static contents, the `ROOT` directory is enough, while to serve Java servlets the WAR file is required.


## Basic usage as integrated Web server

To develop a simple Web application with Jetty, we first need to obtain the Java Servlet API. It's important to check that the correct versions of the Servlet API are used: for example, Jetty version 9 and above uses the Servlet API 3 or superior. Using an older Servlet version with Jetty 9 will cause errors to be produced, due to required classes not being found: for example, the `javax.servlet.FilterRegistration` class was introduced by the Servlet 3 API, and it's used by Jetty 9.
```
$ wget -U none http://www.java2s.com/Code/JarDownload/servlet/servlet-api-3.0.jar.zip
$ unzip servlet-api-3.0.jar.zip
$ JETTY_VERSION=9.2.8.v20150217
$ wget -U none http://repo1.maven.org/maven2/org/eclipse/jetty/aggregate/jetty-all/$JETTY_VERSION/jetty-all-$JETTY_VERSION.jar
```

Now we can proceed to create a basic Jetty Web application. Let's create a `HelloWorld.java` file:
```java
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.ServletException;

import java.io.IOException;

import org.eclipse.jetty.server.Server;
import org.eclipse.jetty.server.Request;
import org.eclipse.jetty.server.handler.AbstractHandler;

public class HelloWorld extends AbstractHandler
{
    public void handle(String target,
                       Request baseRequest,
                       HttpServletRequest request,
                       HttpServletResponse response)
        throws IOException, ServletException
    {
        response.setContentType("text/html;charset=utf-8");
        response.setStatus(HttpServletResponse.SC_OK);
        baseRequest.setHandled(true);
        response.getWriter().println("<h1>Hello World</h1>");
    }

    public static void main(String[] args)
        throws Exception
    {
        Server server = new Server(8080);
        server.setHandler(new HelloWorld());
        server.start();
        server.join();
    }
}
```

compile it:
```
javac -cp jetty-all-9.2.8.v20150217.jar:servlet-api-3.0.jar HelloWorld.java
```

and execute it:
```
$ java -cp .:servlet-api-3.0.jar:jetty-all-9.2.8.v20150217.jar HelloWorld
```

This is an example of usage of Jetty as an integrated Web server, meaning that the Web server is embedded inside our application, rather than being run as a separate process, like Apache Tomcat would do for example.

When our application is started, we build a new `org.eclipse.jetty.server.Server` object with `Server server = new Server(8080);`, passing the port number we want to use as constructor argument (the default host is `localhost`). Then we need to set the delegate object that will handle requests and responses coming to the server: this delegate must implement `org.eclipse.jetty.server.handler.AbstractHandler`, and in particular it must provide an `handle()` method. Finally, we can start the server with `server.start()`, and forcing the application to wait with the server with `server.join()`.

The `handle()` method takes the Web path it will respond to (like `"/"`), a `org.eclipse.jetty.server.Request` server request, a `javax.servlet.http.HttpServletRequest` that is the Servlet API request wrapping the original server one, and finally a `javax.servlet.http.HttpServletResponse` servlet response.

Withing the `handle()` method we use basic methods of the servlet response to set the content type and the status of the response. With `baseRequest.setHandled(true)` we confirm that the original request was actually handled. Finally, we write the response body.


## Usage for development with Maven

The typical use of Jetty is as external Web server and servlet container, rather than embedded in the application. In this case, in production environments Jetty will be launched as a separate process, which will be fed with WAR files containing the applications to run. During development, if we are using the build tool Maven, we can use the `jetty-maven-plugin` to ease the usage of Jetty to great extent. In particular, we are able to run a single command to create WAR pacakges of the current development files, and spin up a new Jetty instance reading those files, and we can configure it so that this is done automatically before launching integration tests.

To use `jetty-maven-plugin`, inside our `pom.xml` we add:
```xml
<dependencies>
    ...
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>RELEASE</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
<build>
    ...
    <plugins>
        <plugin>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>maven-jetty-plugin</artifactId>
            <version>9.4.0.v20161208</version>
            <dependencies>
                <dependency>
                    <groupId>javax.servlet</groupId>
                    <artifactId>javax.servlet-api</artifactId>
                    <version>3.1.0</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

here we just need to require the servlet API, and not the Jetty API as well, since we're not integrating Jetty inside our application, and so we don't need to directly interface with its API.

Let's create a simple servlet now, `src/main/java/jetty/test/SimpleServlet.java`:
```java
package jetty.test;

import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class SimpleServlet extends HttpServlet
{
    public void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException
    {
        PrintWriter out = response.getWriter();
        out.println("SimpleServlet Executed");
        out.flush();
        out.close();
    }
}
```

the setup for our Web application is contained in `src/main/webapp/WEB-INF/web.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
            http://java.sun.com/xml/ns/javaee/web-app_3_1.xsd"
         id="jgroph"
         version="3.1">

    <display-name>Test</display-name>
    <servlet>
        <servlet-name>simple</servlet-name>
        <servlet-class>jetty.test.SimpleServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>simple</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

Now, to compile and run our simple application, suffices to do:
```
$ mvn jetty:run
```

and if we browse to http://localhost:8080/ we'll be able to see the expected "SimpleServlet Executed" message.

The `jetty-maven-plugin` can also monitor our project's file and rebuild the application each time a file change is detected, allowing us not to have to repeatedly restart the server. This feature, however, is only limited to the Web files, meaning the files under the `webapp` directory. Having the Web application being automatically redeployed at every file change is a bit trickier. First, we have to enable the file scan, setting the `scanIntervalSeconds` configuration property of the `jetty-maven-plugin` at a certain number of seconds (for example, `2`), and restart the Jetty server to load the new configuration. From now on, each time we rebuild Java files, for example with `mvn:compile`, the Jetty plugin will automatically detect the code changes and restart the server.

An example configuration is:
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-maven-plugin</artifactId>
            <version>9.4.0.v20161208</version>
            <dependencies>
                <dependency>
                    <groupId>javax.servlet</groupId>
                    <artifactId>javax.servlet-api</artifactId>
                    <version>3.1.0</version>
                </dependency>
            </dependencies>
            <configuration>
                <scanIntervalSeconds>2</scanIntervalSeconds>
                <stopPort>8005</stopPort>
                <stopKey>STOP</stopKey>
            </configuration>
        </plugin>
    </plugins>
</build>
```
