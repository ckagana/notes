# Notes on Java features


## Language features

Java SE 7 came with the following new features:
- support for dynamic languages to be running on the JVM
- compressed 64-bit pointers to improve performance on 64-bit JVMs.
- *diamonds*: avoid repeating the type in the right side of declarations:

```java
Map<String Map<String, Map<Integer, List<MyBean>>>> map = new Hashtable<>();
```

- *try-with-resources*: classes implementing `java.lang.AutoCloseable` have their resources automatically closed (the close method is automatically called) in an implicit `finally` block, and any exception thrown at that moment are added to an existing exception, or thrown after the resources have been closed.

```java
try (Connection connection = dataSource.getConnection(); PreparedStatement statement = connection.prepareStatement(...)) {
    // set up statement
    try (ResultSet resultSet = statement.executeQuery()) {
        // do something with result set
    } catch (Exception e) {
    }
} catch (SQLException e) {
    // do something with exception
}
```

- *multi-catch*: multiple kinds of exceptions can be caught in a single `catch` block, provided that one is not of a subclass of the other.

```java
try {
    // do something
} catch (MyException | YourException e) {
    // handle these exceptions the same way
}
```

- *binary literals*: such as `0b11110001000`.
- *underscores in numeric literals*: such as `1_928` or `0b111_1000_1000`.
- `String`s can be used as `switch` arguments.

Java SE 8 comes with the following new features:
- *lambda expressions*:
```java
Thread thread = new Thread(() -> {
    // do something
});
```
- *method references*:
```java
Thread thread = new Thread(this::doSomething);
```
- *JSR 310*: a new date and time API


## Java APIs
Several Java technologies will be explored during the development of this application. Here's a reference of technologies that may be found:

Included in Java SE:
- JDBC: *Java DataBase Connector*
- JNDI: *Java Naming and Directory Interface*
- JAAS: *Java Authentication and Authorization Service*
- JAXP: *Java API for XML Processing*
- JMX: *Java Management Extensions*
- SAAJ: *SOAP with Attachments API for Java*
- StAX: *Streaming API for XML*
- JAF: *Java Beans Activation Framework*
- JAXB: *Java Architecture for XML Binding*

Included in Java EE, Web Services Technologies:
- Web Services for J2EE
- JAX-RPC: *Java API for XML-based RPC*
- JAXR: *Java API for XML Registries*
- JAX-WS: *Java API for XML-based Web Services*
- Web Service Metadata for the Java Platform
- JAX-RS: *Java API for RESTful Web Services*
- JAXM: *Java APIs for XML Messaging*

Included in Java EE, Web Application Technologies:
- Servlet
- JSP: *Java Server Pages*
- JSTL: *Java Server Pages Standard Tag Library*
- JSF: *Java Server Faces*
- JUEL: *Java Unified Expression Language*
- Java API for WebSockets
- Java API for JSON Processing

Included in Java EE, Enterprise Application Technologies:
- EJB: *Enterprise Java Beans*
- Connector Architecture
- JMS: *Java Message Service*
- JTA: *Java Transaction API*
- Java Mail
- JAP: *Java Persistence API*
- Common Annotations API
- CDI: *Contexts and Dependency Injection for Java*
- Dependency Injection for Java
- Bean Validation
- Managed Beans
- Interceptors
- Batch Applications for the Java Platform 1.0 and Concurrency Utilities for Java EE

Included in Java EE, Management and Security Technologies
- JACC: *Java Authorization Service Provider Contract for Containters*
- Enterprise Edition Management API
- Enterprise Edition Deployment API
- JASPIC: *Java Authentication Service Provider Interface for Containers*
