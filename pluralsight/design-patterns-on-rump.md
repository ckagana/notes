# Design patterns on rump

Design patterns come from the realization that many times we face the same kinds of problems over and over again, like the following:
- getting notification when a process completes or a state changes
- making a complex API easier to work with, so that clients don't need to deal with the inner implementation details
- adding functionality to an existing object, even when we don't have its source code, or we can't extend it (`sealed` classes in .NET)
- behaving in distinct ways according to a current mode or state
- looping through elements of a collection or sequence

One important thing to notice about these problems statements, is that they are all quite generic: this is because we are talking about broad kinds of problems, rather than of specific situations, and because we want to define solutions to generic problems, so that they can be reused many times: a solution to a very specific situation, in fact, can hardly be reused outside of that same situation.

Design patterns, thus, don't give us the specific implementation of a solution, but only the core of it: they describe the major parts of the implementation, and how they interact with each other. This solution, then, can be reused many times, but never the same way twice.

Many SDKs provide implementations of common design patterns in their libraries. However, often these implementations don't just fit our needs very well: in those cases, we need to come out with different implementations of the same design patterns.

## Observer pattern
This pattern describes publish/subscribe relationships. The publisher, also known as the *subject*, is the object that we want to watch for a state change. Subscribers are the observers, and they are notified when the subject changes its state. For instance, this is the behaviour of the follow button on twitter.

In the .Net framework, event handlers are implementations of the observer pattern:

```c#
public partial class MainWindow: Window
{
    public MainWindow()
    {
        InitializeComponent();
        ClickMeButton.Click += Observer1;
        ClickMeButton.Click += Observer2;
        ClickMeButton.Click += Observer3;
    }

    void Observer1(object sender, RoutedEventArgs e)
    {
        TextBlock1.Text = "Hello from Observer1";
    }

    void Observer2(object sender, RoutedEventArgs e)
    {
        TextBlock2.Text = "Hello from Observer2";
    }

    void Observer3(object sender, RoutedEventArgs e)
    {
        MessageBox.Show("Hello, from Observer 3");
    }
}
```

### Consequences of the Observer pattern
Since the notification from the subject may never come (because the state we are monitoring may happen to never change), observers should always be properly initialized, and they should never rely on getting a notification.

We may get repeated notifications, so observers should be prepared to deal with duplicated notifications. For instance, the observer may start a long process in reaction to the first notification: thus, the second, duplicated, notification may come when the process kicked off by the first one, is still running. The problem here is that we may need to start the process again, or to check if the state change is really relevant.

In certain circumstances, notifications can come very quickly, like hundreds or thousands of times per second. In these cases, it could be better to process only samples of notifications, like every two seconds.

Sicne the subject keeps a reference of every subscriber, after one subscriber is no longer used, it can't be disposed by the garbage collector, because a reference to it is still held by the subject (which tipically outlives observers).

## Event aggregators
Event aggregators are designed to work along with publishers and subscribers that don't know anything about each other. A subscriber would subscribe to an event, and the publisher would publish that event.

```c#
using Microsoft.Practices.Prism.Events;
using System.Collections.Generic;
using System.Windows;

namespace Observer_StockTicker
{
    public partial class App : Application
    {
        private List<Window> receivers = new List<Window>();

        protected override void OnStartup(StartupEventArgs e)
        {
            base.OnStartup(e);

            var eventAggregator = new EventAggregator();

            // The two windows, generator and receiver, don't know anything
            // about each other: they communicate only through the event
            // aggregator.
            var generator = new StockGenerator(eventAggregator);
            var receiver1 = new StockReceiver(eventAggregator);
            var receiver2 = new StockReceiver(eventAggregator, "ABC");
            var receiver3 = new StockReceiver(eventAggregator, "XYZ");

            Application.Current.MainWindow = generator;

            generator.Show();
            receiver1.Top = 200;
            receiver1.Owner = generator;
            receiver1.Show();
            receiver2.Owner = generator;
            receiver2.Top = 300;
            receiver2.Show();
            receiver3.Owner = generator;
            receiver3.Top = 500;
            receiver3.Show();
        }
    }
}
```

Below is defined the Stock Ticker event, carrying the Stock Ticker payload, which contains all data we are interested to send to our subscribers:

```c#
using Microsoft.Practices.Prism.Events;

namespace Observer_StockTicker
{
    public class StockTickerEvent : CompositePresentationEvent<StockTickerPayload>
    {

    }

    public class StockTickerPayload
    {
        public string StockId { get; set; }
        public float StockValue { get; set; }
    }
}
```

```c#
using Microsoft.Practices.Prism.Events;
using System;
using System.Collections.Generic;
using System.Windows;

namespace Observer_StockTicker
{
    public partial class StockGenerator : Window
    {
        private int _currentValue = 100;
        private IEventAggregator _eventAggregator;
        private Random _random;
        private List<string> _stockIds;
    }

    public StockGenerator(IEventAggregator eventAggregator)
    {
        InitializeComponent();
        _eventAggregator = eventAggregator;
        _random = new Random();
        _stockIds = new List<string>()
        {
            "ABC", "RUFF", "XYZ",
        };
        StockIdCombo.ItemsSource = _stockIds;
        StockIdCombo.SelectedIndex = 0;
    }

    private void GenerateOneButton_Click(object sender, RoutedEventArgs e)
    {
        GenerateTickerEvent();
    }

    private void GenerateOneHundredButton_Click(object sender, RoutedEventArgs e)
    {
        for (int i = 0; i < 100; i++) {
            GenerateTickerEvent();
        }
    }

    private void GenerateTickerEvent()
    {
        var delta = _random.Next(-1, 2);
        var stockId = StockIdCombo.Text;
        currentValue += delta;
        var output = string.Format("{0}: {1}", stockId, _currentValue.ToString());
        StockValueList.Items.Insert(0, output);

        var payload = new StockTickerPayload
        {
            StockId = stockId,
            StockValue = _currentValue,
        };

        // Here we are using the event aggregator to create a new
        // StockTickerEvent event, passing the payload to it, so that it will
        // carry it. All subscribers are notified here with the generated
        // event.
        _eventAggregator.GetEvent<StockTickerEvent>().Publish(payload);
    }
}
```

```c#
using Microsoft.Practices.Prism.Events;
using System.Windows;

namespace Observer_StockTicker
{
    public partial class StockReceiver : Window
    {
        int receiver = 0;
        string stockFilter = "";

        public StockReceiver(IEventAggregator eventAggregator, string stockID = "")
        {
            InitializeComponent();

            stockFilter = stockID;
            if (string.IsNullOrEmpty(stockFilter))
            {
                eventAggregator.GetEvent<StockTickerEvent>()
                    .Subscribe(ProcessStockTickerData, false);
            }
            else
            {
                // The last lambda tells which events we want to be notified of
                eventAggregator.GetEvent<StockTickerEvent>()
                    .Subscribe(
                        ProcessStockTickerData,
                        ThreadOption.UIThread,
                        false,
                        p => p.StockId == this.stockFilter
                    );
            }
        }

        public void ProcessStockTickerData(StockTickerPayload payload)
        {
            received++;
            TextBlock1.Text = payload.StockId;
            TextBlock2.Text = payload.StockValue.ToString();
            TextBlock3.Text = string.Format("Total Received: {0}", received);
        }
    }
}
```

When we create a new event in the receiver, during the subscription, if that is a strong reference, we are always maintaining a reference to each event, even though after they are fired, they are not being used anymore. If we start firing thousands of events, we end up maintaining thousands of events in memory, without ever releasing any of them. The solution is unsubscribing from that event, or using a weak reference (the `false` parameter in the `Subscribe` methods).

In .NET 4.0 there are `IObserver<T>` and `IObservable<T>`, where `T` is the payoload: this is another implementation of the Observer pattern.
- `IObserver<T>`
    + `OnNext(T value)`: called by the publisher when a notification is available, `value` contains the payload of the notification.
    + `OnCompleted()`: let the observer know that there will be no more notifications.
    + `OnError(Exception exception)`: an error occurred.
- `IObservable<T>`
    + `IDisposable Subscribe(IObserver<T> observer)`: when the subject want to notify a new event, it just calls `OnNext` of each observer, passing the payload to it. This method is called by observers that want to be registered to a subject. After calling this method, a `IDisposable` object is returned: calling `Dispose` on it will unsubscribe the current observer from the subject.

## Proxy pattern
When we want to use an object, we instead have another placeholder object to interact with. Thus, we have more than one way to access an object, that is transparent to the user. From the outside, interacting with the proxy, and interacting with the original object look the same. The proxy can add functionality that for instance improve performance, such as controlling access to an object across a network.

This happens in real world when person A gives power of attorney to person B. If someone wants to interact legally with person A, for instance to sign a contract, she can choose to talk directly to A, which may be unavailable or hard to reach though (for instance has moved to another country), or she can interact with B, and it would be exactly the same from a legal standpoint.

Let's consider SOAP services references in Visual Studio. Consider a SOAP service application `Proxy-SOAPService`:

```C#
namespace Proxy_SOAPService
{
    [ServiceContract]
    public interface IPersonService
    {
        [OperationContract]
        List<Person> GetPeople();
    }

    public class Person
    {
        public string FirstName;
        public string LastName;
        public DateTime StartDate;
        public int Rating;
    }
}
```

```C#
namespace Proxy_SOAPService
{
    public class PersonService : IPersonService
    {
        #region IPersonService Members

        public List<Person> GetPeople()
        {
            var p = new List<Person>()
            {
                new Person() { FirstName = "John", LastName = "Koenig" ,
                    StartDate = DateTime.Parse("10/17/1975"), Rating = 6 },
                new Person() { FirstName = "Dylan", LastName = "Hunt",
                    StartDate = DateTime.Parse("10/02/2000"), Rating = 8 },
                new Person() { FirstName = "John", LastName = "Chrichton",
                    StartDate = DateTime.Parse("03/19/1999"), Rating = 7 },
                new Person() { FirstName = "Dave", LastName = "Lister",
                    StartDate = DateTime.Parse("02/15/1988"), Rating = 9 },
                new Person() { FirstName = "John", LastName = "Sheridan",
                    StartDate = DateTime.Parse("01/26/1994"), Rating = 6 },
                new Person() { FirstName = "Dante", LastName = "Montana",
                    StartDate = DateTime.Parse("11/01/2000"), Rating = 5 },
                new Person() { FirstName = "Isaac", LastName = "Gampu",
                    StartDate = DateTime.Parse("09/10/1977"), Rating = 4 }
            };
            return p;
        }

        #endregion
    }
}
```

So this just creates a list of seven people, and returns it. The consumer of this service is the `Proxy-ServiceProxy` application. Visual Studio has an interface to create a service reference, which means that it automatically generates a class `Proxy_ServiceProxy.MyService.PersonServiceClient`.

```C#
using Proxy_ServiceProxy.MyService;
using System.Windows;

namespace Proxy_ServiceProxy
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        private void ClickMeButton_Click(object sender, RoutedEventArgs e)
        {
            PersonServiceClient serviceProxy = new PersonServiceClient();   
            PersonListBox.ItemsSource = serviceProxy.GetPeople();
        }
    }
}
```

The SOAP service is a resource that is living somewhere on the network, and it expects that we communicate with it with SOPacket (?), which is an XML document defining the specific elements included. To communicate directly with the service, without a proxy, we would need to create an XML SOAP request packet with our request, make an HTTP call to send it across the wire, wait for response, parse the XML SOAP response packet, and turn it into a CLR object to use in our code. The proxy simplify all this, in particular making so that we can create an instance of the proxy, and call `GetPeople` on it, just as if the service class originally featuring `GetPeople` was local, instead of living somewhere else on the network.

With the proxy pattern we are often hiding the fact that the object that we are using is in a different process, or even across the network. Using methods that do inter process or network communication without realizing it may lead to performance issues. Other than this, the proxy class must kept synchronized with the real object: if we add a method to the service, we must also add the same method to the proxy.

Consider using the proxy pattern when dealing with low bandwidth networks: you show a placeholder image, for instance, and you download the real image only if the user clicks on the placeholder.

## Iterator pattern
Consider a television remote control: it has a built-in iterator, which is the channel-up button. Whenever we click that button, we're asking for the next available channel in the collection, without caring how the television determins what the next channel should be: for example it will skip channels that don't have a signal, or maybe we programmed it to skip certain channels, but we are not interesterd with this when we click the button.

We have an aggregate, and an iterator, and we interact only with the iterator, to have the next element. The foreach loop is an implementation of the iterator pattern.

```c#
using System;
using System.Collections.Generic;

namespace Iterator_foreach
{
    public static class People
    {
        public static List<Person> GetPeople()
        {
            var p = new List<Person>()
            {
                new Person() { FirstName = "John", LastName = "Koenig",
                    StartDate = DateTime.Parse("10/17/1975"), Rating = 6 },
                new Person() { FirstName = "Dylan", LastName = "Hunt",
                    StartDate = DateTime.Parse("10/02/2000"), Rating = 8 },
                new Person() { FirstName = "John", LastName = "Chricton",
                    StartDate = DateTime.Parse("03/19/1999"), Rating = 7 },
                new Person() { FirstName = "Date", LastName = "Lister",
                    StartDate = DateTime.Parse("02/15/1988"), Rating = 9 },
                new Person() { FirstName = "John", LastName = "Sheridan",
                    StartDate = DateTime.Parse("01/26/1994"), Rating = 6 },
                new Person() { FirstName = "Dante", LastName = "Montana",
                    StartDate = DateTime.Parse("11/01/2000"), Rating = 5 },
                new Person() { FirstName = "Isaac", LastName = "Gampu",
                    StartDate = DateTime.Parse("09/10/1977"), Rating = 4 }
            };
            return p;
        }
    }

    public class Person
    {
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public DateTime StartDate { get; set; }
        public int Rating { get; set; }
    }
}

```

```c#
using System.Collection.Generic;
using System.Windows;

namespace Iterator_foreach
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        provide void ClickMeButton_Click(object sender, RoutedEventArgs e)
        {
            IEnumerable<Person> people = People.GetPeople();

            //foreach (var person in people)
            //    PersonListBox.Items.Add(person);

            var enumerator = people.GetEnumerator();
            while(enumerator.MoveNext()) {
                PersonListBox.Items.Add(enumerator.Current);
            }
        }
    }
}
```

`foreach` works on anything that implements `IEnumerable`, which has a method `GetEnumerator()` that returns an `IEnumerator<T>` object, which is another way to call iterators. `MoveNext()` returns a boolean that tells if it successfully moved, `Current` gets the current item the iterator is pointing to.

Adding or removing items from the aggregate may invalidate the iterator, thus, we should never allow that when using an iterator, for instance inside a foreach loop. We can also have multiple iterators working on the same collection at the same time.

`IEnumerator<T>` contains `T Current`, `bool MoveNext()`, `void Reset()` and `void Dispose()`. `Dipose()` clear resources possibly used by the collection.

Iterators can also be used to create calculated sequences (generators!).

```c#
using System;
using System.Collections;
using System.Collections.Generic;

namespace Iterator_SequenceLibrary
{
    public class FibonacciSequence : IEnumerable<int>
    {
        public int NumberOfValues { get; set; }

        public FibonacciSequence(int numberOfValues)
        {
            NumberOfValues = numberOfValues;
        }

        public IEnumerator<int> GetEnumerator()
        {
            return new FibonacciEnumerator(NumberOfValues);
        }

        IEnumerator IEnumerable.GetEnumerator()
        {
            return this.GetEnumerator();
        }
    }
}
```

```c#
using System;
using System.Collections;
using System.Collections.Generic;

namespace Iterator_SequenceLibrary
{
    internal class FibonacciEnumerator : IEnumerator<int>
    {
        private int _numberOfValues;
        private int _currentPosition;
        private int _previousTotal;
        private int _currentTotal;

        public FibonacciEnumerator(int numberOfValues)
        {
            _numberOfValues = numberOfValues;
        }

        public int Current
        {
            get { return _currentTotal; }
        }

        object IEnumerator.Current
        {
            get { return Current; }
        }

        public bool MoveNext()
        {
            if (_currentPosition == 0)
            {
                _currentTotal = 1;
            }
            else
            {
                int _newTotal = _previousTotal + _currentTotal;
                _previousTotal = _currentTotal;
                _currentTotal = _newTotal;
            }
            _currentPosition++;

            return _currentPosition <= _numberOfValues;
        }

        public void Reset()
        {
            _currentPosition = 0;
            _previousTotal = 0;
            _currentTotal = 0;
        }

        public void Dispose()
        {
            GC.SuppressFinalize(this);
        }
    }
}
```

```c#
using System.Windows;
using Iterator_SequenceLibrary;

namespace Iterator_SequenceViewer
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        private void ClickMeButton_Click(object sender, RoutedEventArgs e)
        {
            OutputListBox.Items.Clear();
            var sequence = new FibonacciSequence(10);
            foreach (var item in sequence)
                OutputListBox.Items.Add(item);
        }
    }
}
```

We can create this generator just using the `yield return` statement, that returns an internal enumerator.

```c#
//...
public IEnumerator<int> GetEnumerator()
{
    int _previousTotal = 0;
    int _currentTotal = 0;

    for (int i = 0; i < NumberOfValues; i++)
    {
        if (i == 0)
        {
            _currentTotal = 1;
        }
        else
        {
            int _newTotal = _previousTotal + _currentTotal;
            _previousTotal = _currentTotal;
            _currentTotal = _newTotal;
        }
        yield return _currentTotal;
    }
}
//...
```

## Chain of responsibility
In tech support you have level 1 tech support, asking basic questions: if it can solve the problem, the issue is closed, and everything stops there; otherwise, you go to level 2 support, doing more advanced operations. Again, if the problem is not solved again, it gets passed to the next step of the chain, etcetera.

The sender of the message is not responsible for making sure it gets to the right person: the message is send to the first message of the chain, and if it can handle the message, it stops, otherwise it goes on.

The try/catch block is an implementation of the chain of responsibility. If the message is not handled by any step of the chain, it falls down the chain, so we must be sure to have generic handlers that can handle any message.

For instance, consider a user submitting a request for a quantity of a particular item; rules determine which of these requests require additional approvement from the management team. Rules can get quite complex. When a request came in, it's compared with the first rule: if the rule determines that the request requires approval, it is put in the queue list for approval. Otherwise, the request goes to the second rule: if it requires approval, it's put into pending approval state, and so on.

## Facade pattern
Think of an entertainment system: you have many remote controls. If you want to play DVD you have to turn on the TV, set TV to Component Input, turn on stereo, set stereo to Aux Input, turn on DVD Player, hit "Play" on DVD Player. This is a complex subsystem where we have to know a lot of details about the interfaces of the subsystem. A universal remote control can be programmed to make all these steps automatically, hiding the complexity.

The background worker component is an example of the facade pattern. This component is used to execute tasks in the background, keeping the UI thread responsive, while hiding the complexity of threading management from the user. However, complex threading operations cannot be done, because all threading is hidden.

Thus, one of the consequences of the facade pattern is that if we want to expose more functionality from the subsystem, we may need to update the current facade, or create a new one. In addition to that, since the facade hides the complexity of the API, it also hides its functionality, so we cannot access the API to do something complex with that.

For instance, the `foreach` construct is a facade in front of the iterator pattern: in particular, `foreach` allows us to move over each element of the collection, like we were calling `moveNext`, with a very simplified interface, but at the same time prevent us from doing more complex operations, like resetting the iterator at a certain point.

## Factory method pattern
A factory method let a class defer instantiation to subclasses. We want to create an abstraction on how a particular object is created: the factory method will choose which particular instantiation to use to create the object.

Repositories create an abstraction between the application and the data access layer, so that we can access different data sources without updating the application code. The application makes call into the repository; there are different implementations of the repository: one access data from a WCF service, one from a CSV file, another one from a SQL server. All these share the repository interface, so they can be easily plugged into the application.

```c#
using PersonRepository.CSV;
using PersonRepository.Interface;
using PersonRepository.Service;
using PersonRepository.SQL;
using System.Windows;

namespace PeopleViewer
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        private void ServiceFetchButton_Click(object sender, RoutedEventArgs e)
        {
            ClearListBox();

            IPersonRepository repository = new ServiceRepository();
            var people = repository.GetPeople();
            foreach (var person in people)
            {
                PersonListBox.Items.Add(person);
            }

            ShowRepositoryType(repository);
        }

        private void CSVFetchButton_Click(object sender, RoutedEventArgs e)
        {
            ClearListBox();

            IPersonRepository repository = new CSVRepository();
            var people = repository.GetPeople();
            foreach (var person in people)
            {
                PersonListBox.Items.Add(person);
            }

            Show RepositoryType(repository);
        }

        private void SQLFetchButton_Click(object sender, RoutedEventArgs e)
        {
            ClearListBox();

            IPersonRepository repository = new SQLRepository();
            var people = repository.GetPeople();
            foreach (var person in people)
            {
                PersonListBox.Items.Add(person);
            }

            ShowRepositoryType(repository);
        }

        private void ClearButton_Click(object sender, RoutedEventArgs e)
        {

        }
    }
}
```

Here, the UI code is responsible to choose which concrete repository to instantiate. In addition to this, the code to populate the list is repeated among three methods, where only the repository class used changes. We need to create a factory method which will be responsible to choosing the right class to instantiate.

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using PersonRepository.CSV;
using PersonRepository.Service;
using PersonRepository.SQL;
using PersonRepository.Interface;

namespace PersonRepository.Factory
{
    public static class RepositoryFactory
    {
        public static IPersonRepository GetRepository(string repositoryType)
        {
            IPersonRepository repo = null;

            switch (repositoryType)
            {
                case "Service": repo = new ServiceRepository();
                    break;
                case "CSV": repo = new CSVRepository();
                    break;
                case "SQL": repo = new SQLRepository();
                    break;
                default:
                    throw new ArgumentException("Invalid Repository Type");
            }

            return repo;
        }
    }
}
```

```c#
//...
using PersonRepository.Factory;
//...
private void SQLFetchButton_Click(object sender, RoutedEventArgs e)
{
    FetchData("SQL");
}

//...
private void FetchData(stirng repositoryType)
{
    ClearListBox;

    IPersonRepository repository = 
        RepositoryFactory.GetRepository(repositoryType);

    var people = repository.GetPeople();
        foreach (var person in people)
        {
            PersonListBox.Items.Add(person);
        }

        ShowRepositoryType(repository);
}
//...
```

## Decorator pattern
We wrap an existing class, add our own functionality, and then expose the same interface to the outside world. The decorator looks exactly like the original class to clients. For instance, we can wrap the WCF service repository with a new class that expose the same interface, but adding new behaviour, for instance adding client side caching to each repository.

To add client-side caching functionality, we could subclass the WCF service repository, adding the caching functionality, but this would require us to do the same thing to all other classes. A better approach is using a class that can add the functionality to any repository: this is what the decorator pattern does.

```c#
using PersonRepository.Interface;
using System;
using System.Collections.Generic;
using System.Linq;

namespace PersonRepository.CachingDecorator
{
    public class CachingRepository : IPersonRepository
    {
        private TimeSpan _cacheDuration = TimeSpan.FromSeconds(30);
        private DateTime _dataDateTime;
        private IPersonRepository _personRepository;
        private IEnumerable<Person> _cachedItems;

        private bool IsCacheValid
        {
            get
            {
                var_cacheAge = DateTimeOffset.Now - _dataDateTime;
                return _cacheAge < _cacheDuration;
            }
        }

        private void ValidateCache()
        {
            if (_cachedItems == null || !IsCacheValid)
            {
                try
                {
                    _cachedItems = _personRepository.GetPeople();
                    _dataDateTime = DateTime.Now;
                }
                catch
                {
                    _cachedItems = new List<Person>()
                    {
                        new Person(){ FirstName = "No Data Available", LastName = string.Empty };
                    };
                }
            }
        }

        private void InvalidateCache()
        {
            _cachedItems = null;
        }

        public CachingRepository(IPersonRepository personRepository)
        {
            _personRepository = personRepository;
        }

        public IEnumerable<Person> GetPeople()
        {
            ValidateCache();
            return _cachedItems;
        }

        public Person GetPerson(string lastName)
        {
            ValidateCache();
            return _cachedItems.FirstOrDefault(p => p.LastName == lastName);
        }

        public void AddPerson(Person newPerson)
        {
            _personRepository.AddPerson(newPerson);
            InvalidateCache();
        }

        public void UpdatePerson(string lastName, Person updatedPerson)
        {

        }

        public void DeletePerson(string lastName)
        {

        }

        //...
    }
}
```

```c#
namespace PeopleViewer
{
    public partial class MainWindow : Window
    {
        private IPersonRepository repository;

        public MainWindow()
        {
            InitializeComponent();

            var wrappedRepository = new ServiceRepository();
            repository = new CachingRepository(wrappedRepository);
        }

        //...
    }
}
```

## Adapter pattern
Consider a HR system and a Finance system; they each have a `Employee` class, but with different interfaces, so we can't use an object from HR inside Finance and vice-versa. We can create an adapter object that make an HR employee look like a Finanance employee: this object wraps an existing HR object, providing data in the way Finance expects it.

```c#
using System;

namespace PersonRepository.Interface
{
    public class Person
    {
        public virtual string FirstName { get; set; }
        public virtual string LastName { get; set; }
        public virtual DateTime StartDate { get; set; }
        public virtual int Rating { get; set; }
    }
}
```

```c#
namespace PersonRepository.SQL
{
    using System;
    using System.Collections.Generic;

    public partial class DataPerson
    {
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public Nullable<System.DateTime> StartDate { get; set; }
        public Nullable<int> Rating { get; set; }
    }
}
```

`StartDate` and `Rating` in `DataPerson` are `Nullable`, so the two interfaces are incompatible.

```c#
//...
namespace PersonRepository.SQL
{
    public class SQLRepository : IPersonRepository
    {
        public IEnumerable<Person> GetPeople()
        {
            using (var ctx = new PeopleEntities())
            {
                var dataPeople = ctx.DataPersons.ToList();

                var people = new List<Person>();
                foreach (var dataPerson in dataPeople)
                {
                    people.Add(new Person
                        {
                            FirstName = dataPerson.FirstName,
                            LastName = dataPerson.LastName,
                            StartDate = dataPerson.StartDate.Value,
                            Rating = dataPerson.Rating.Value,
                        });
                }

                return people;
            }
        }

        public Person GetPerson(string lastName)
        {
            Person person = null;

            using (var context = new PeopleEntities())
            {
                var foundPerson = GetDataPerson(context, lastName);
                if (foundPerson !== null)
                {
                    person = new Person()
                    {
                        FirstName = foundPerson.FirstName,
                        LastName = foundPerson.LastName,
                        StartDate = foundPerson.StartDate.Value,
                        Rating = foundPerson.Rating.Value,
                    }
                }
            }

            return person;
        }

        // ...
    }
}
```

Here the same procedure is repeated in multiple places: create a `Person`, and convert the existing data person calling `.Value` on nullable properties. We can solve this with an adapter class.

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace PersonRepository.SQL.Adapter
{
    class ApplicationPerson : Person
    {
        private DataPerson _dataPerson;

        public ApplicationPerson(DataPerson dataPerson)
        {
            _dataPerson = dataPerson;
        }

        public override string FirstName
        {
            get
            {
                return _dataPerson.FirstName;
            }
        }

        public override string LastName
        {
            get
            {
                return _dataPerson.LastName;
            }
            set
            {
                base.LastName = value;
            }
        }

        public override DateTime StartDate
        {
            get
            {
                if (_dataPerson.StartDate.HasValue)
                {
                    return _dataPerson.StartDate.Value;
                }
                else
                {
                    return DateTime.MinValue;
                }
            }
        }

        public override int Rating
        {
            get
            {
                if (_dataPerson.Rating.HasValue)
                {
                    return _dataPerson.Rating.Value;
                }
                else
                {
                    return int.MinValue;
                }
            }
        }
    }
}
```

With this adapter we can do:

```c#
//...
public class SQLRepository : IPersonRepository
{
    public IEnumerable<Person> GetPeople()
    {
        using (var ctx = new PeopleEntities())
        {
            var dataPeople = ctx.DataPersons.ToList();

            var people = new List<Person>();
            foreach (var dataPerson in dataPeople)
            {
                people.Add(new ApplicationPerson(dataPerson));
            }

            return people;
        }
    }

    public Person GetPerson(string lastName)
    {
        Person person = null;

        using (var context = new PeopleEntities())
        {
            var foundPerson = GetDataPerson(context, lastName);
            if (foundPerson != null)
            {
                person = new ApplicationPerson(foundPerson);
            }
        }
        return person;
    }

    //...
}
```

## WPF Event Bubbling
In WPF there is a hierarchical tree of UI elements, much like the DOM of HTML pages. For instance, a window contains a button, which contains an icon and a text block.

Event bubbling means that the lowermost object gets the first shot at handling an event: if that object doesn't handle the event, then it bubbles up to the parent; if that object does not handle the event, it bubbles up to its parent, and so forth.

If we hover the mouse over the image, first the image gets a chance to handle the MouseOver event: if it handles the event, then the event is marked as "handled" and everything stops. If the image does not handle the event, it bubbles up to the button. If the button handles the event, then the event is marked as "handled", otherwise, it bubbles up to the window.

This is the Chain of Responsibility pattern: the event is the message, and each element has a chance to handle it. But we previously saw that event handlers are implementations of the Observer pattern, so aren't we using here the Observer pattern? Actually, here the event handling part is secondary to the process, because we are interested in bubbling the events, so this is probably not an example of the Observer Pattern, but this is quite subjective.

## Other patterns
The Repository pattern is designed to act as a layer of abstraction between the core application and the underlying data store. There are a couple of subpatterns that describe how data access is broken down: the CRUD approach uses methods for Create, Read, Update and Delete; the CQRS approach has a division between the methods that read the data (query), and those that update the data (command).

Inversion of Control and Dependency Injection, concerned with how dependencies are managed: Constructor Injection, Property Injection, Service Locator.

Model-View-ViewModel, Model-View-Controller, Model-View-Presenter: separating the presentation logic from the application logic. MVVM is good for client applications; MVC is good for Web applications, with some processing on the server, that emits a view to the browser.

## Resources
- Robert C. Martin (Uncle Bob)
- Martin Fowler
- Dino Esposito (book about architecting enterprise architectures)
- Joshua Kerievsky (Refactoring to Patterns)
- "On the use and misuse of patterns" Rockford Lhotka (http://goo.gl/5ZXpZO)