Design patterns are very general, non generic, templates to how to solve common problems in software designs. They are not like software components or assemblys, because they are elements of reuse, and design patterns are not elements of reuse: they are meant to address how specific classes or class elements elated to each other to solve a very common problem type.

Design patterns are related to application and system design domains. There are patterns at both very high level, and low level. Design patterns are concerned with the interfaces of our classes, not with the details, like class names. Design patterns are not concerned with implementations of classes, they give only a starting point, because situations differ.

## The Adapter pattern
Consider a class that would be useful to your application, but which does not implement an interface you require; or, you are designing a class or a framework and you want to ensure it is usable by a wide variety of as-yet-unwritten classes and applications: in this case you would add support to adapter to other classes to you framework. Object adapters do not require multiple inheritance, as class adapters do.

The intent of the adapter pattern is to convert the interface of one class into another interface, its client expects. This allows classes to work together, that wouldn't otherwise. In this way we can future proof client implementations by having them depend on adapter interfaces, rather than concrete classes directly.

Consider using the adapter pattern whenever you want to use an existing class's functionality, but its interface is not the one that you require. Likewise if you want to create reusable code, but you don't want to bind it too tightly to a particular implementation, you should use some kind of adapter interface as what your code depends on, so that future clients could implement their own version of that adapter and still make use of your code. You'll also find the adapter pattern useful if there are several existing implementations of code that you want to be able to use and it's impractical to adapt each of their interfaces by subclassing every one. By implementing an adapter and writing an adapter for each of these subclasses, for instance if you have a data access class, one for SQL server, another for Oracle, another for DB2, you would be able to create a data adapter, and create just the adapter classes for each of those different implementations, rather than trying to change those implementations directly to expose the interface that you require.

The two basic players of the adapter pattern are the client and the adaptee. The client needs some of the logic provided by the adaptee, `adaptedOperation()`, but the client has been written in such a way that it can't directly call `adaptedOperation()`, because its interface is not the one the client expects: `operation()`. So we create an adapter interface, exposing an operation that has the interface the client expects, so `operation()`. Next, for each different implementation required, at a minimum one, a different concrete adapter is created, that takes that `operation()` and implements it such that that code calls `adaptedOperation()`. In this way the original client will now be able to use the original `adaptedOperation()`, while actually calling the `operation()` it knows about, on the adapter.

The way the adapter pattern is used is that client classes are written in such a way that they depend on an interface, rather than on a particular implementation. Then at least one concrete adapter class is created, which implements this interface, allowing the client to work with a particular class that it requires. Or, you may already have the client and the concrete adapter under your control, and you are just writing the adapter interface to decouple the client from the adapter interface: in this way we are future proofing the client, allowing for later changes to be implemented through its behaviour, by simply writing new adapters that work with new implementations of this code. This is a very effective way to achieve the Open/Closed principle.

In terms of collaborators within the Adapter pattern, clients simply call operations on an Adapter instance, through the Adapter interface. Likewise, the Adapter instance calls the adaptee operations that carry out the request.

Some of the consequences of using the Adapter design pattern are that a single adapter interface may work with many adaptees: for instance it could work with one adaptee and all of its subclasses, which is nice because it gives you some of reuse through the use of polymorphism; or you have the ability to write a different concrete adapter implementation for each different adaptee class that you want to be able to work with. This gives you some flexibility and also allows you to follow the open/closed principle again because you can just write a new class that implements the behaviour for a new concrete adaptee with the behavior that you need to be able to reference. Note that it can be difficult to override the behavior of the adaptee class using an object adapter, because you cannot simply inherit from it. Since we are using composition instead of inheritance, the only way to change the behavior of the adaptee is to subclass it directly, add the overriding behavior, and than change the concrete adapter implementation so that it referes to the adaptee class. We consider this to be difficult when compared to a programming language that would allow for multiple inheritance, where you would only have to do this in one place, which would be in our adapter itself.

`DataRenderer.cs`

```c#
using System.Data;
using System.IO;

namespace AdapterDemo.Model
{
    public class DataRenderer
    {
        private readonly IDbDataAdapter _dataAdapter;

        public DataRenderer(IDbDataAdapter dataAdapter)
        {
            _dataAdapter = dataAdapter;
        }

        public void Render(TextWriter writer)
        {
            writer.WriteLine("Rendering Data:");
            var myDataSet = new DataSet();

            _dataAdapter.Fill(myDataSet);

            foreach (DataTable table in myDataSet.Tables)
            {
                foreach (DataColumn column in table.Columns)
                {
                    writer.Write(column.ColumnName.PadRight(20) + " ");
                }
                writer.writeLine();
                foreach (DataRow in table.Rows)
                {
                    for (int i = 0; i < table.Columns.Count; i++)
                    {
                        writer.Write(row[i].ToString().PadRight(20) + " ");
                    }
                    writer.WriteLine();
                }
            }
        }
    }
}
```

This class expects an `IDbDataAdapter` as part of its construction. It exposes a single method called `Render` and using that `Render` method it's going to write out to a `TextWriter` the contents of the data that it gets from that source. Now the way that it's going to get that data is through the use of the `DataAdapter.Fill` method, which is going to populate a `DataSet`, that we're going to pass to that `Fill` method, empty. Once we have our data, we're simply going to loop through columns, and write out the column headers, and then for each row we're going to write out the data in each row.

Now, in order to make use of the Adapter, we need to be able to provide multiple different adapters to the same `DataRenderer` class. So this is also an example of the Strategy pattern in this case, because we're able to change how `DataRenderer` behaves based on what type of interface we pass into it.

This was a way to use an Adapter pattern included in the .NET framework. Now let's look at another example of how we would implement an adapter ourselves. Imagine that we have a model now that includes the notion of a `Pattern`, with its `Id`, `Name` and `Description`.

```c#
namespace AdapterDemo.Model
{
    public class Pattern
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Description { get; set; }
    }
}
```

And we need to be able to render that pattern. Somewhere in our application we already have code that is requiring this interface that expects to be able to call `PatternRenderer` and get back a string when it says `ListPatterns`. We want this to give us back some kind of tabular representation of our patterns, very similar to what the `DataRenderer` does given a `DataAdapter`. But `DataRenderer.Render` is a `void` that expects a `TextWriter`, and our `PatternRenderer` is a `string` that expects a collection of patterns. So the code to rendere our data in a tabular fashion exists in our `DataRenderer`, we don't have easy access to it, because it doesn't have the correct interface that we expect.

```c#
using System.Collections.Generic;

namespace AdapterDemo.Model
{
    public class PatternRenderer
    {
        public string ListPatterns(IEnumerable<Pattern> patterns)
        {
            return patterns.ToString();
        }
    }
}
```