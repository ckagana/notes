# Java Web applications


Java Web applications are run by a *Web container*, also known as *Servlet container*. More complex Java Enterprise applications are instead run by *Application containers*. While Web containers only implement a limited subset of the Java EE technologies, such as Servlet, JSP and JSTL, application containers implement the full Java EE specification.

Standard Java EE Web applications are deployed either as WAR files, or as directories. A WAR file is much like regular JAR files, meaning compressed archives with a specific directory structure recognized by the JVM. Both WAR files and raw Web directories have the same structure:
- a `META-INF` directory
- a `WEB-INF` directory
- other static files accessible from the Web
Thus, the difference between a WAR file and a raw Web directory are just the `META-INF` and `WEB-INF` directories (in addition to the fact that the WAR is a compressed archive of course). All resources contained inside `META-INF` and `WEB-INF` are not directly accessible from the Web.

The `WEB-INF` directory is responsible for containing files needed by the Web container to determine how to deploy and run the application. It contains:
- a `classes` directory, containing in turn all compiled Java classes, and representing the root of the code package. Additionally, it contains a `META-INF` directory providing various application resources,
- a `lib` directory, containing libraries as JAR files, used by the core package classes- a `tags` and a `tld` directory, containing JSP tag files and tag library descriptors, respectively
- a `i18n` directory, containing internationalization and localization files.

The `META-INF` directory contains the application manifest files, and additional resources for specific application servers or Web containers.

Only files contained inside `WEB-INF` are included in the application classpath: this means that they can be loaded with the `ClassLoader`.

In addition to the application classes and resources necessary to run the application, we also need to provide information to the Web container about how the application should be deployed. This is achieved by the *deployment descriptor* which is the set of metadata describing how the application should be deployed in terms of servlet, listeners, filters, and all the other Java EE technologies. Traditionally, the deployment descriptor was all contained inside `/WEB-INF/web.xml`: however, since Servlet 3.0 it's possible to add descriptor elements also through additional means:
- annotations can be added directly inside servlet classes
- servlet fragments can be used, which are JAR files containing the `/META-INF/web-fragment.xml` descriptor, in addition to annotations as well, and can also contain servlet, filters, etc. Web fragments can be activated in the desired order, specified inside `web-fragment.xml`, or inside the application's `web.xml`.

The JVM specification requires three different class loaders to be used. The root class loader is used to load Java core classes, the `java.*` ones; then comes the extension class loader, which loads classes from extension JARs located in the JRE installation directory; last comes the application class loader, responsible to load the application classes. These loaders are related to each other in a hierarchy where the extension class loader is parent of the application class loader, and the root one is parent of the extension one.

Regular Java applications use the *parent-first class loader delegation model*: when a class loader is requested to load a class, it first asks its parent, and only if it can't load the class, it will try to do it itself. This means that when a class is requested to be loaded by the application, the application class loader is asked first: this one first checks with the extension loader to see if it can load the class, and in turn the latter asks the root loader; the first finding the class, loads it. This way, it's impossible to overwrite the root classes, because they will always be the first ones to be loaded.

This model, however, poses some problems when it comes to Web applications. According to the Java EE specification, Web application are run by a Web container, or application server: this component is a Java application in its own rights, and as such it will use its own classes and libraries. However, a Web container is designed to handle many different Web applications: each one of them can be regarded as another Java application with its own classes and libraries. A very common problem here is that the same library, with different versions, is used by more than one of these coexisting Web application, or even also by the application server itself.

For this reason, Java EE applications need to use the *parent-last class loader delegation model*. Web containers are requested to assign each Web application it's own class loader: the regular container class loader is parent of each application's class loader. Unlike the previous model, though, when an application class loader is required to load a class, it asks its parent (the container class loader) only if it cannot load it: this way if an application is shipped with a specific version of a library, its classes are always guaranteed to be loaded first, and thus with the correct version. The only exception to this rule regards core Java classes, that are still loaded by the root class loader first, to avoid security issues with someone trying to override core Java classes. Additionally, since each application class loader is child of the server one, there is no risk that an application loads a class from another one.

