# Introduction to Java servlets


In Java EE, *Servlets* are the software components responsible for receiving Web requests, and responding to them. Servlets can only be instantiated and run by a Web container, which is the one providing them with requests, and taking responses from them.

To use servlets in your application, you need to link the Java servlet library to your application. Using Maven, this is a matter of adding the following dependency:

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
```

The `provided` scope means that this library shouldn't be added to the application package, since it is already provided by the Web container.

Servlets are Java classes implementing the `javax.servlet.Servlet` interface. In actuality, however, servlets just extends one predefined servlet class, instead of directly implementing `Servlet`, specialized for various kinds of requests, like HTTP (in the future they may also support FTP).

For HTTP requests, you would usually extend `javax.servlet.http.HttpServlet`. This class provides empty implementations of methods used to respond to specific kinds of HTTP requests, like `doGet()`, `doPost`, etc. These methods take a `javax.servlet.http.HttpServletRequest` object as the request, and a `javax.servlet.http.HttpServletResponse` object as the response.

If a request comes from a method that hasn't been overriden, for example if a `GET` request comes, and `doGet()` hasn't been overriden in the servlet class, then a 405 Method Not Allowed error response will be returned.

The most simple and useful thing to do when overriding a servlet method, is writing some text in the response object. `HttpServletResponse` provides a `java.io.PrintWriter` output stream through the `getWriter()` method (inherited from the more generic `ServletResponse` interface). This stream can be used to write objects (which will be converted to `String`'s) to the body of the HTTP response:

```java
package net.slc.jgroph;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class HelloServlet extends HttpServlet
{
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException
    {
        response.getWriter().println("Hello, World!");
    }
}
```

The default content type of the response is `text/plain`. The servlet response automatically takes care of using default headers and properly formatting the HTTP for the underlying socket. Additionally, you don't have to call `close()` on the `PrintWriter` stream you're using: this is because the best practice suggests that the owner of a resource, meaning who created it, is responsible for closing it; in this case the Web container created the stream, and as such it will also close it.

All methods dealing with user requests suffer from the same problem: they are called too late in the execution flow. Usually, any slightly complex application needs to do some setup work before it's able to properly handle user requests, like setting up database connections, reading configuration files, etc. The way we do these kinds of things with servlets, is through the `init()` method. This is called by the Web container when the servlet is started, typically during the deployment (but this really depends on the configuration of the servlet).

Analogous to it is the `destroy()` method, called by the Web container when the servlet cannot accept requests anymore, typically when the application is stopped or the Web container is shut down. Since at this moment the application is being terminated, you don't have to wait for the garbage collector to call `finalize()` to release the open resources: hence, this should be done directly inside `destroy()`. Should you try to release resources inside `finalize()`, instead, you'd have to wait several minutes before the garbage collector runs, because it is tied to the Web container, which is still running after your servlet has been terminated; the consequence of this may be that your application is partially undeployed, or that its undeployment fails.

```java
@Override
public void init() throws ServletException
{
    System.out.println("Servlet " + this.getServletName() + " has started.");
}

@Override
public void destroy()
{
    System.out.println("Servlet " + this.getServletName() + " has stopped.");
}
```

The `init` method should throw a `ServletException` if something went wrong during the initialization, that prevents the servlet from running properly. If this happens, any request handled by this servlet will be responded with `503 Service Unavailable`.

Writing a servlet class is not enough to have a working Web application. What we need to do now is configuring the deployment descriptor so that our servlet is properly deployed to the Web container. The deployment descriptor defines the listeners, servlets, filters and settings that should be deployed with the application.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                             http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <display-name>Hello World Application</display-name>
</web-app>
```

These are the required schema URIs for Java EE 7. The `version` specified refers to the Servlet API version, `3.1` in this case. Let's take a look at the servlet definition:

```xml
<servlet>
    <servlet-name>helloServlet</servlet-name>
    <servlet-class>net.slc.jgroph.HelloServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
```

The name of the class matches the one we created before. The `load-on-startup` configuration instructs the Web container to load this servlet as soon as the Web application starts: this is useful because the servlet loading, involving calling the `init()` method, may take quite a bit since it may need to open database connections, load configuration files, etc. Without this configuration, the servlet would be loaded the first time a request for it comes: this means that the first user of this servlet may be forced to wait for several seconds for the loading to complete. The `1` is the order of loading of the servlets: if this application had several servlets, we could use this tag to specify in which order to load them, with higher numbers coming later. If multiple servlets share the same order value, they'll be loaded in the order they are defined in the descriptor.

Another important configuration is specifying what URL requests should be mapped to a specific servlet:

```xml
<servlet-mapping>
    <servlet-name>helloServlet</servlet-name>
    <url-pattern>/greeting</url-pattern>
</servlet-mapping>
```

This means that all requests to the URL `/greeting` will be handled by this servlet. More than one URL can be mapped to the same servlet.

```xml
<servlet-mapping>
    <servlet-name>helloServlet</servlet-name>
    <url-pattern>/greeting</url-pattern>
    <url-pattern>/salutation</url-pattern>
    <url-pattern>/wazzup</url-pattern>
</servlet-mapping>
```

In this case all three URLs are mapped to the same servlet instance. The path `/` is special, in that it matches all possible other paths. For instance, if you register the path `/greeting`, you would get a match only if you request exactly `/greeting` (`/greeting/` won't match either). However, if you register the path `/`, you would get a match with every path starting with `/`, namely, every path. (@todo: this can depend on the specific Web container used).

We can instantiate the same servlet multiple times:

```xml
<servlet>
    <servlet-name>oddsStore</servlet-name>
    <servlet-class>net.slc.jgroph.StoreServlet</servlet-class>
</servlet>
<servlet>
    <servlet-name>endsStore</servlet-name>
    <servlet-class>net.slc.jgroph.StoreServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>oddsStore</servlet-name>
    <url-pattern>/odds</url-pattern>
</servlet-mapping>
<servlet-mapping>
    <servlet-name>endsStore</servlet-name>
    <url-pattern>/ends</url-pattern>
</servlet-mapping>
```

In this case, instead, `oddsStore` and `endsStore` are two distinct servlets, that are two different instances of the same servlet class, `net.slc.jgroph.StoreServlet`. This can be done, for instance, to access the same logic, with different configuration. It's possible to get the name of the servlet instance from inside the servlet class calling `this.getServletName()`. Beware that instantiating the same servlet class multiple times using annotations only is not possible.

If you're using the servlet API 3.0 or greater, you can avoid defining your servlets in the deployment descriptor `web.xml`, and adding annotations to your servlet class instead:

```java
...
import javax.servlet.annotations.WebServlet
@WebServlet(name="helloServlet", urlPatterns={"/greeting", "/salutation", "/wazzup"}, loadOnStartup=1)
public class HelloServlet extends HttpServlet
{
    ...
}
```

The deployment descriptor must still exist, though, and have a valid servlet API 3.0 header. The `<display-name>` tag must also stay there.

Along with a Web request, come *request parameters*. They contain information sent by the client to the application. This can be done via query parameters, or via form variables. Query parameters are supported by several request methods (`GET`, `POST`, `HEAD`, ...) like in:

```
GET /index.jsp?productId=9781118656464&category=Books HTTP/1.1
```

or

```
POST /index.jsp?returnTo=productPage HTTP/1.1
Host: www.example.com
Content-Length: 48
Content-Type: application/x-www-form-urlencoded

addToCart&productId=9781118656464&category=Books
```

In this last example we are seeing both query parameters (`returnTo`), and form variables (`addToCart` and `productId`).

The servlet API does not differentiate between these two kinds of parameters. The `getParameter()` method takes the parameter name, and returns a single value for a paramter. If a parameter has more than one value, the first one is returned. The `getParameterValues()` method returns an array of values instead. If the parameter has only one value, an array with only one element is returned. The `getParameterMap()` method returns a `java.util.Map<String, String[]>` object containing the parameter names mapped to their values. The `getParameterNames()` returns an enumeration of the parameter names.

The input stream of a request can be read only once. If you call `getInputStream()` or `getReader()` on the request object, and later try to call any of the parameters methods, you'll get an `IllegalStateException`. The same happens vice-versa.

The `getContentType()` method returns the MIME content type of the request, such as `application/json`. The `getContentLength()` and `getContentLengthLong()` methods return the content length of the request body (the latter works also when the content exceeds 2 gigabytes). The `getCharacterEncoding()` method returns the character encoding of the request, such as UTF-8, whenever the request is of character type.

The `getRequestURL()` method returns the entire request URL, excluding the query string. The `getRequestURI()` method returns only the server path of the request. If the request is `/hello-world/greeting?foo=world`, and the application is deployed as `/hello-world`, and the servlet mappings is `/greeting`, then `getServletPath()` will return only `/greeting`. The `getHeader()` method returns the value of a header, given its name, the header name being case-insensitive; if one header has multiple values, this method only returns the first one. The `getHeaders()` method returns an enumeration of all values of a specified header. The `getHeaderNames()` method returns an enumeration of all the headers in the request. The `getIntHeader()` method returns an header value as an integer; if it cannot be converted to integer, a `NumberFormatException` is thrown. The `getDateHeader()` method returns the Unix timestamp in milliseconds of a header value representing a valid timestamp (otherwise an `IllegalArgumentException` is thrown).

To write to the response body, you can choose between `getOutputStream()` and `getWriter()`, according to you wanting to write raw bytes, or text. As with the request object, you can write to the response only once: if you attempt to do it twice, you'll get an `IllegalStateException`.

The `setContentType()` and `setCharacterEncoding()` methods allow you to specify the response content type and character encoding. You can call them many times, each overwriting the previous one, but they must be called before writing to the response body (otherwise they're ignored).

You can use `setHeader()`, `setIntHeader()` and `setDateHeader()` to specify the response headers. To avoid overwriting existing values, and add new ones instead, you can use `addHeader()`, `addIntHeader()` and `addDateHeader()`. You can inspect which headers are already defined with `getHeader()`, `getHeaders()`, `getHeaderNames()` and `containsHeader()`.

Additionally, you can use `setStatus()`, `getStatus()`, `sendError()` and `sendRedirect()`.

There are many ways of handling configuration for your servlets. Two of these are *context init parameteters*, and *servlet init parameters*.

Context init parameters can be defined only inside the deployment descriptor, not with annotations:

```xml
<context-param>
    <param-name>settingOne</param-name>
    <param-value>foo</param-value>
</context-param>
<context-param>
    <param-name>settingTwo</param-name>
    <param-value>bar</param-value>
</context-param>
```

To get context parameters from inside a servlet:

```java
ServletContext c = this.getServletContext();
response.getWriter()
    .append("settingOne: ").append(c.getInitParameter("settingOne"))
    .append(", settingTwo: ").append(c.getInitParameter("settingTwo"));
```

Every servlet in the application share these context parameters, because they are bound to the *context* instead of to a specific servlet. Servlet init parameters are, instead, tied to specific servlets:

```xml
<servlet>
    <servlet-name>servletParameterServlet</servlet-name>
    <servlet-class>net.slc.jgroph.ServletParameterServlet</servlet-class>
    <init-param>
        <param-name>database</param-name>
        <param-value>CustomerSupport</param-value>
    </init-param>
    <init-param>
        <param-name>server</param-name>
        <param-value>10.0.12.5</param-value>
    </init-param>
</servlet>
<servlet-mapping>
    <servlet-name>servletParameterServlet</servlet-name>
    <url-pattern>/servletParameters</url-pattern>
</servlet-mapping>
```

```java
ServletConfig c = this.getServletConfig();
response.getWriter()
    .append("database: ").append(c.getInitParameter("database"))
    .append(", server: ").append(c.getInitParameter("server"));
```

Servlet init parameters can also be added as annotation, like:

```java
@WebServlet(
    name = "servletParameterServlet"
    urlPatterns = {"/servletParameters"},
    initParams = {
        @WebInitParam(name = "database", value = "CustomerSupport"),
        @WebInitParam(name = "server", value = "10.0.12.5")
    }
)
public class ServletParameterServlet extends HttpServlet
{
    ...
}
```

This, however, makes little sense, since the purpose of configurations is to be able to change them without recompiling the application (you just need to restart the deployed application). Adding servlet parameters as annotations to the servlet class would require to recompile the application at each configuration change (not to talk about saving sensitive information like database connection credentials to your VCS repository).

To allow file uploads, we enable it with the `multipart-config` definition:

```xml
<servlet>
    ...
    <multipart-config>
        <location>/tmp/myapp</location>
        <file-size-threshold>5242880</file-size-threshold>
        <max-file-size>20971520</max-file-size>
        <max-request-size>41943040</max-request-size>
    </multipart-config>
</servlet>
```

The `location` parameter tells the Web server where to store temporary files, if needed, during the upload procedure; this parameter can be left empty, in which case the default server temporary directory will be used. The `file-size-threshold` parameter contains the size in bytes under which files being uploaded will be kept in memory: files exceeding this size will be stored in the temporary directory, and deleted after the upload is complete. The `max-file-size` and `max-request-size` parameters are the limits of the uploaded files sizes, and of the total size of requests. It's also possible to define these configurations using the `@MultipartConfig` annotation, with paramters `location`, `fileSizeThreshold`, `maxFileSize` and `MaxRequestSize`; of course, this would mean tying those configurations to the servlet source file.

When you send POST request, by default the data you are sending is encrypted by the browser with the `application/x-www-form-urlencoded` method, which just employs the usual URL encoding. However, if you're uploading files you need to [basically send raw bytes](https://www.w3.org/TR/html401/interact/forms.html#h-17.13.4), and you'll have to set up a form with encryption type `multipart/form-data`, and inputs of type `file`. In the default `application/x-www-form-urlencoded` encryption, input values are all crammed in the same URL-encoded string, like `scalar%3Dvalue1%26list%5B%5D%3Dvalue2%26list%5B%5D%3Dvalue3` (which contains a `scalar` field with `value1` and a `list` field with values `value2` and `value3`). This way of encrypting data is very inefficient to send large quantities of binary data (as files are), or strings containing unicode characters. With `multipart/form-data`, instead, each input value is sent separately, as a *part*, in the same order as the respective inputs appear in the HTML.

The `multipart-config` is enough to handle data coming from such type of forms. However, using them at the Java code level requires to handle the specific parts instead of the higher level parameters. Usual non-file input values are stored as parameters, exactly like usual `application/x-www-form-urlencoded` forms. However, you can't receive binary data as parameters, since `ServletRequest.getParameter()` returns a string, so you have to use the `getPart()` method, taking the field name as well, and returning an instance of `javax.servlet.http.Part`, which in turn can be read with the `getInputStream()` method. The `getSubmittedFileName()` returns the original file name before it was uploded.

Then, if you want the user to download some binary data, you have to craft a special response, having the `Content-Disposition` header set to `attachment; filename=your-file-name` (forcing the user's browser to download the file instead of opening it inside the browser) and content type set to `application/octet-stream`. Then you can write the binary data directly to `response.getOutputStream()`. If you want to allow downloading of large files, storing them in memory as a whole may not be performance savy: in this case you would want to open an `InputStream` from that file, and directly copy bytes from it to the `ResponseOutputStream`, and flush it frequently, so that data is continuously been streamed to the user instead of buffering in memory.

Web applications are handled by Web containers using a different thread per client connection. Thus, Web containers employ a *connector pool*, also called *executor pool*. When a new request is incoming, the container looks for an available thread in the pool: if no thread is available because the pool has reached its maximum size, the request is added to a queue, waiting for an available thread (usually the queue also has a maximum size, after which the container starts rejecting connections). When a request has completed, and the response body has been sent back to the client, the thread handling that request will become free to be used by other requests. The usage of a pool is an obvious design choice to avoid having to create and destroy threads at each request, which is a very expensive operation to perform.

The Servlet 3.0 specification adds asynchronous request contexts: the `ServletRequest.startAsync()` method can be used to retrieve an `AsyncContext` object, after which the code can return without responding to the request. In another part of the code, responding to a certain event, that `AsyncContext` object can be used to resume the processing procedure and send the proper response to the client. This can be used to implement asynchronous execution, for instance to avoid blocking threads during I/O calls.

Static and instance variables of a servlet are shared amongst all threads using that servlet. This means that these variables need to be synchronized to avoid corruption of that data. For instance, declaring a variable as `private volatile int TICKET_ID_SEQUENCE = 1;` ensures that when a new request for reading that value comes, the latest write operation (if any) is waited to be completed before doing the actual read (in normal Java multithread execution this is not guaranteed). Additionally, we can write code like:

```java
synchronized(this) {
    id = this.TICKET_ID_SEQUENCE++;
    this.ticketDatabase.put(id, ticket);
}
```

When multiple threads try to execute this code at the same time, the synchronized block prevents them to step on each other's toe: when a thread enters the synchronized block, all threads coming later are forced to wait at the beginning of that block, after that thread finished its execution, before the next one has a chance of executing it as well. This ensures that no two threads can execute this code at the same time. This is important in this case, because if two threads were executing simultaneously, this could happen:
- `TICKET_ID_SEQUENCE` is 0
- the first thread starts updating `TICKET_ID_SEQUENCE`, from 0 to 1
- the second thread also starts updating `TICKET_ID_SEQUENCE`, before the first one finished doing it, so the second thread still sees that `TICKET_ID_SEQUENCE` is equal to 0, and tries to update it to 1
- the first thread finishes updating `TICKET_ID_SEQUENCE`, to the value of 1
- the second thread finishes updating `TICKET_ID_SEQUENCE`, to the value of 1
- `TICKET_ID_SEQUENCE` is 1, instead of 2

At this point, we would have two threads both writing to the id 1 of the database, meaning that instead of adding two tickets next to each other, the second ticket would override the first.

Additionally, requests and responses objects obviously belong to the request being handled by the current thread. This means that sharing references to these objects to the other threads will have disastrous consequences, so never store references to the current request or response object into static or instance servlet variables.