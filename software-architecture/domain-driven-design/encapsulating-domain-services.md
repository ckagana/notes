# Encapsulating domain services

Let's consider the case when a domain is defining a service interface whose implementation needs to be provided by the user of the domain. Usually the application would take the responsibility of implementing this service: however, it may happen that the implementation actually depends on a specific adapter, and thus cannot be provided by the application layer. In this case the easiest solution would be to let the adapter implement the domain service interface.
```java
package domain.Service;

public interface TaskPerformerService
{
    public void performTask();
}
```
```java
package domain.Service;

import domain.Service.TaskPerformerService;

public class ServiceUser
{
    public void use(TaskPerformerService service)
    {
        // use the given service...
    }
}
```
```java
package infrastructure.RemoteTaskPerformer;

import domain.Service.TaskPerformerService;

public class RemoteTaskPerformerService implements TaskPerformerService
{
    @Override
    public void performTask()
    {
        // implementation depends on details of the remote task performer adapter
    }
}
```
```java
package application.PrimaryPort;

import domain.Service.TaskPerformerService;
import domain.Service.ServiceUser;

public class UseCase
{
    public void execute()
    {
        // ...
        TaskPerformerService service = container.make(TaskPerformerService);
        ServiceUser user = container.make(ServiceUser.class);
        user.use(service);
    }
}
```
```java
package infrastructure.App;

import infrastructure.RemoteTaskPerformer.RemoteTaskPerformerService;
import domain.Service.TaskPerformerService;

public class App
{
    // ...
    public void run()
    {
        // ...
        container.bind(TaskPerformerService, RemoteTaskPerformerService);
    }
}
```

This solution, however, it's not as clean as it could be, because the role of an adapter is to connect to a port, and ports are defined by the application, not by the domain. To sort this out, we can define an application port that just wraps the domain service interface:
```java
package application.TaskPerformer;

public interface TaskPerformerService implements domain.Service.TaskPerformerService
{
}
```
```java
package infrastructure.RemoteTaskPerformer;

import application.TaskPerformer.TaskPerformerService;

public class RemoteTaskPerformerService implements TaskPerformerService
{
    @Override
    public void performTask()
    {
        // implementation depends on details of the remote task performer adapter
    }
}
```
```java
package application.PrimaryPort;

import domain.Service.ServiceUser;
import application.TaskPerformer.TaskPerformerService;

public class UseCase
{
    public void execute()
    {
        // ...
        TaskPerformerService service = container.make(TaskPerformerService);
        ServiceUser user = container.make(ServiceUser.class);
        user.use(service);
    }
}
```
```java
package infrastructure.App;

import application.TaskPerformer.TaskPerformerService;
import infrastructure.RemoteTaskPerformer.RemoteTaskPerformerService;

public class App
{
    // ...
    public void run()
    {
        // ...
        container.bind(TaskPerformerService, RemoteTaskPerformerService);
    }
}
```

Here we are abiding by the clearer structure according to which adapters connect to ports, instead of just being implementations of random interfaces defined somewhere.

This might seem a hack, or a stubborn desire to design by the book: instead, it's a solution that is extremely common on a different design level, and that's because it brings important benefits. Let's consider the following code:
```java
public class Time
{
    public int seconds;
}
```

Of course we all know the limitations of this design: the moment we'll decide to change the implementation of `Time` from raw `seconds` to three distinct properties: `seconds`, `minutes` and `hours`, we'll have to change the public interface of the class, even though its meaning didn't change. So, we learn to apply encapsulation:
```java
public class Time
{
    private int seconds;
    
    public int getSeconds()
    {
        return seconds;
    }
}
```

where we can freely change the implementation, without users being affected by any change. Of course this solution also allows us to add all kinds of logic to the method that returns the seconds, like validation, formatting, etc.

Now, at a higher design level, the same principle is used when we encapsulate the domain service within an application port. In the most simple situation the application port is just renaming the domain service, but it's not limited to that: it could be use to add decorators to the adapter for example:
```java
package application.TaskRunner;

public interface TaskRunner
{
    public void run();
}
```
```java
package application.PrimaryPort;

import domain.Service.TaskPerformerService;
import domain.Service.ServiceUser;
import application.TaskRunner.TaskRunner;
import application.PrimaryPort.RunnerDecorator;

public class UseCase
{
    public void execute()
    {
        // ...
        TaskRunner runner = container.make(TaskRunner.class);
        TaskPerformerService service = new RunnerDecorator(runner);
        ServiceUser user = container.make(ServiceUser.class);
        user.use(service);
    }
}
```

So what happens here is that the application service defines the interface that parties external to the application will see, which is the exact definition of a port. In this specific case, the implementation behind this interface is to directly delegate to the domain service, but this is of no interest to external parties, because we want to be able to change this implementation, for example adding some more logic to the given concrete adapter, which would be impossible if we let the adapter directly implement the domain service interface.
