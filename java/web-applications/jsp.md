# JSP

JSP is a technology designed to simplify writing the body of the servlet responses, namely, to avoid having to write the body inside `String` literals, and to `append()` them to the `PrintWriter` response object. JSP is a template engine, that allows you to combine HTML tags, Java code, and JSP tags in the same source. Other template engines exist of course, for instance Facelets, which are part of the JavaServer Faces (JSF) framework, or Velocity, Freemarker, SiteMesh, Tiles, etc.

JSP code is interpreted by the JSP compiler, which creates a new servlet class for each JSP file. In particular, the compiled JSP extends `HttpServlet.HttpJspBase`, which in turn is a servlet. What this servlet does, is practically turn the HTML you wrote in the JSP files into strings, appended to the servlet's output stream. All of this is done in the `_jspService()` method, which is called by the servlet `service()` method.

Usually JSP are compiled just in time the moment of the first request to the specific JSP. It's also possible to pre compile all JSP during the application deploy.

In addition to regular HTML tags, JSP provides JSP tags, that are classified as *directives*, *declarations*, *scriptlets* and *expressions*:

```xml
<%@ this is a directive %>
<%! this is a declaration %>
<% this is a scriptlet %>
<%= this is an expression %>
```

Directive tags are used to perform actions, making assumptions, import classes or include other JSP or tags. For example, JSP have default content type of `text/html` and character encoding of `ISO-8859-1`. We can change these settings directly in the JSP file, instead of changing the header on the response object in the servlet, with the `page` directive:

```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
```

This directive is put at the beginning of a JSP file, and allows to specify the content type of the request of the current JSP servlet. Additionally, the `language` attribute select the scripting language used by the current JSP to provide dynamic features (although Java is currently the only supported scripting language).

Directives are also used to add imports to JSP:

```xml
<%@ page import="java.util.*,java.io.IOException" %>
```

You can use a different directive for each attribute:

```xml
<%@ page contentType="text/html;charset=UTF-8" %>
<%@ page language="java" %>
<%@ page import="java.util.Map" %>
<%@ page import="java.util.List" %>
<%@ page import="java.io.IOException" %>
```

Everything that is not included in a JSP tag is part of the output. This means that, in the previous example, five newline characters will be printed to the output, because they are located outside of any JSP tag. To avoid this, we can do like:

```xml
<%@ page import="java.util.Map"
%><%@ page import="java.util.List"
%><%@ page import="java.io.IOException" %>
```

Among other features of the `page` directive, `pageEncoding` allows you to directly specify the page encoding, so that instead of writing `contentType="text/html;charset=UTF-8"`, you can write `contentType="text/html" pageEncoding="UTF-8"`. `session` can either be `true` or `false` and indicates if the HTTP session should be enabled for this JSP (default `true`). The `isErrorPage` attribute can be set to `true` in JSP that should be used to handle errors.

The `include` directive allows to include other JSP inside the current one:

```xml
<%@ include file="/path/to/some/file.jsp" %>
```

Absolute paths are resolved from the web root of the application, otherwise it resolves to the same directory containing the current JSP. The inclusion is resolved when the JSP is compiled, and the content of the included file are added in place of the `include` directive.

Alternatively, you can also have dynamic (run time) inclusion, with the `<jsp:include>` tag:

```xml
<jsp:include page="/path/to/some/page.jsp" />
```

In this case, the included file is compiled separately, and at run time the request is forwarded to the included JSP, its output is written to the response, and the control is sent back to the original JSP. This approach can be useful when dealing with very large JSP, because the bytecode of a compiled Java method cannot exceed 65,534 bytes, and the bigger the JSP, the bigger the `_jspService` method.

Declaration tags are used to declare variables, methods or classes within the scope of a JSP (servlet).

Scriptlet tags contain Java code which is copied to the body of the `_jspService()` method. Scriptlets will thus have access to all identifiers declared within the scope of that method, and will be able to declare new ones.

Expression tags contain Java expressions, returning a value that will be directly written to the output, at the point where the expression tag is located.

```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%!
    private final int five = 0;
    protected String cowboy = "rodeo";
    
    // The assignment below is not declarative and is a syntax error if uncommented
    // cowboy = "test";
    
    public long addFive(long number) {
        return number + 5L;
    }
    
    public class MyInnerClass
    {
    }
    
    MyInnerClass instanceVariable = new MyInnerClass();
    
    // WeirdClassWithinMethod is in method scope, so the declaration below is
    // a syntax error if uncommented
    // WeirdClassWithinMethod bad = new WeirdClassWithinMethod();
%>
<%
    class WeirdClassWithinMethod
    {
    }
    
    WeirdClassWithinMethod weirdClass = new WeirdClassWithinMethod();
    MyInnerClass innerClass = new MyInnerClass();
    int seven;
    seven = 7;
%>
<%= "Hello, World" %><br />
<%= addFive(12L) %>
```

JSP also provides comment tags, with this form:

```xml
<%-- This is a JSP comment --%>
```

JSP comments are not sent to the browser (they are not part of the HTML output), and the Java code contained in there isn't even executed (it's ignored by the translator altogether).

The `<jsp:forward>` tag enables to forward a request to another JSP, but, unlike `<jsp:include>`, the control doesn't return to the original JSP:

```xml
<jsp:forward page="/some/other/page.jsp" />
```

JSP files provide a set of implicit variables, that you don't need to declare. The JSP specification indicates that the JSP translator and compiler must provide them. These variables are declared within the JSP servlet method (for example, `_jspService`).

The `request` variable is an instance of `HttpServletRequest`, and the `response` variable is an instance of `HttpServletResponse`. You should not call `getWriter` or `getOutputStream`, because the JSP is already writing to the response body. Also, you shouldn't set the content type, or the characater encoding, or deal with the buffer, because they are already dealt with by the JSP itself.

The `session` variable is an instance of `HttpSession`. It is present only if the `session` attribute of the `page` directive is set to `true` (which is the default).

The `out` variable is an instance of `JspWriter`, that you can use to directly write to the response, instead of calling `getWriter` on `response`.

The `application` variable in an instance of `ServletContext`, giving access to the application configuration, like context init parameters.

The `config` variable is an instance of `ServletConfig`, giving access to the specific servlet configuration.

The `pageContext` variable is an instance of `PageContext`, giving access to request attributes, and other things.

The `page` variable is a reference to the JSP servlet object itself. It is a `java.lang.Object`, but can be cast to `Servlet` or to `javax.servlet.jsp.JspPage` (extending `Servlet`), or to `javax.servlet.jsp.HttpJspPage` (extending `JspPage`).

The `exception` variable is a `Throwable`, and it's available when the current JSP has `isErrorPage` set to `true`.

Instead of placing the `page` directive at the top of each JSP, if several different JSPs have the same configuration, we can add them to the deployment descriptor:

```xml
<jsp-config>
    <jsp-property-group>
        <url-pattern>*.jsp</url-pattern>
        <url-pattern>*.jspf</url-pattern>
        <page-encoding>UTF-8</page-encoding>
        <scripting-invalid>false</scripting-invalid>
        <include-prelude>/WEB-INF/jsp/base.jspf</include-prelude>
        <trim-directive-whitespaces>true</trim-directive-whitespaces>
        <default-content-type>text/html</default-content-type>
    </jsp-property-group>
</jsp-config>
```

JSP configurations are organized in *groups*, identified by the URL pattern. In this example, we defined configurations for all JSP having `.jsp` and `.jspf` extensions. The URL pattern contains a server path, relative to the document root, and not a Web path. This means that you can use as a URL pattern `WEB-INF/jsp/admin/*.jsp` if you want to apply certain configuration to all JSP found inside the `WEB-INF/jsp/admin` folder.

Consider this example:

```xml
<jsp-property-group>
    <url-pattern>*.jsp</url-pattern>
    <url-pattern>*.jspf</url-pattern>
    <page-encoding>UTF-8</page-encoding>
    <include-prelude>/WEB-INF/jsp/base.jspf</include-prelude>
</jsp-property-group>
<jsp-property-group>
    <url-pattern>/WEB-INF/jsp/admin/*.jsp</url-pattern>
    <url-pattern>/WEB-INF/jsp/admin/*.jspf</url-pattern>
    <page-encoding>ISO-8859-1</page-encoding>
    <include-prelude>/WEB-INF/jsp/admin/include.jspf<include-prelude>
</jsp-property-group>
```

A file named `/WEB-INF/jsp/admin/user.jsp` would match both property groups. In this case, the more specific match is applied, and thus the file would have character encoding of `ISO-8859-1`. However, all matching `<include-prelude>` are applied, which means that in this case both `/WEB-INF/jsp/bsae.jspf` and `/WEB-INF/jsp/admin/include.jspf` will be included at the beginning of the JSP.

While `<include-prelude>` includes files at the beginning of all matching JSPs, the `<include-coda>` tag includes files at the end fo all matching JSPs. Both tags can be used more than once in the same group, to specify several patterns.

The `<trim-directive-whitespaces>` property instructs the JSP translator to remove whitespaces created by any JSP tags in the JSPs. The `<scripting-invalid>` tag can be used to prevent any Java from being executed from within matching JSPs in that group. The `<el-ignored>` corresponds to the `isELIgnored` property of the `page` directive, and is used to prohibit expression language in the matching JSPs.

The `<url-pattern>` tag is the only one mandatory. Other tags must appear in this order (unless they're not used, in which case they must be omitted): `<url-pattern>`, `<el-ignored>`, `<page-encoding>`, `<scripting-invalid>`, `<is-xml>`, `<include-prelude>`, `<include-coda>`, `<deferred-syntax-allowed-as-literal>`, `<trim-directive-whitespace>`, `<default-content-type>`, `<buffer>`, `<error-on-undeclared-namespace>`.

A typical pattern when combining servlets and JSPs is to have the servlet accepting the request, do some business logic with it, and then forward the request to a JSP (acting as the view):

```java
request.getRequestDispatcher("/WEB-INF/jsp/view/ticketForm.jsp").forward(request, response);
```

The call `request.getRequestDispatcher()` returns a `javax.servlet.RequestDispatcher` object, which handles internal forwards and includes related to the given JSP path. After calling `forward`, the servlet should never manipulate the response again, because this would cause errors: the request has been handed to a different part of the application, the JSP in this case.

It's possible to inject data from the servlet to the JSP:

```java
request.setAttribute("ticketId", idString);
request.setAttribute("ticket", ticket);
```

Since the request is present in the JSP as well, you can retrieve these values from the JSP with `(String)request.getAttribute("ticketId")` and `(Ticket)request.getAttribute("ticket")`. However, any object in the life cycle of the request, which has access to the request object, will also have access to these attributes, so they're not specific for the given view.
