# Threads

TODO

## Synchronization

TODO

### Handling cross-thread exceptions *

When a parent thread spawns a child one, it can happen that the child thread throws an exception. If this isn't handled from inside the child thread, the thread itself will just die, giving no notice. This makes debugging this kind of programs quite hard: what we would like, is having the possibility of handling child exceptions from the parent thread, but to make this possible we have to devise a way to pass the exception from the child to the parent.

Let's consider a program where a `Server` class might throw an `IOException` at the construction. We want to start the server from inside a separate thread, for example because we want to communicate with it from the main one:
```java
public class Main
{
    public static void main(final String[] args)
    {
        new Main().run();
    }

    public void run()
    {
        new Thread(() -> {
            new Server().listen("0.0.0.0", port, callback);
        }).start();
    }
}
```

here, if `new Server()` throws an exception, the thread will just die, and no one will know.

To overcome this problem, we can create a `private volatile Throwable exception` property in the parent thread's class, which is meant to be available on all threads, and put the whole code of the child thread inside a `try/catch` block. When an exception is caught in the child, we assign its reference to the volatile property, so that it's available also from outside the child thread. This way, after creating the server, we can just check if the `exception` property is `null`, and if not we handle the exception:
```java
public class Main
{
    private volatile Throwable exception;

    ...

    public void run()
    {
        new Thread(() -> {
            try {
                new Server().listen("0.0.0.0", port, callback);
            } catch (IOException e) {
                exception = e;
            }
        }).start();

        // Leave some time for the thread to start, and for the exception to be thrown
        Thread.sleep(1000);

        if (exception != null) {
            // handle exception
        }
    }
}
```

An alternative solution is based on leveraging the closure defining the child thread behavior. We cannot define an `exception` local variable, and re-assign it from inside the closure, because enclosed variables must always be `final`. However, what we can do is define a container object inside of which our `exception` reference will live: this way the container will be `final`, but its `exception` property will be re-assignable:
```java
public class Main
{
    private class Status
    {
        Throwable exception;
    }

    ...

    public void run()
    {
        final Status status = new Status();

        new Thread(() -> {
            try {
                new Server().listen("0.0.0.0", port, callback);
            } cach (IOException e) {
                status.exception = e;
            }
        }).start();

        ...

        if (status.exception != null) {
            // handle exception
        }
    }
}
```
