Design patterns come from the realization that many times we face the same kinds of problems over and over again, like the following:
- getting notification when a process completes or a state changes
- making a complex API easier to work with, so that clients don't need to deal with the inner implementation details
- adding functionality to an existing object, even when we don't have its source code, or we can't extend it (`sealed` classes in .NET)
- behaving in distinct ways according to a current mode or state
- looping through elements of a collection or sequence

One important thing to notice about these problems statements, is that they are all quite generic: this is because we are talking about broad kinds of problems, rather than of specific situations, and because we want to define solutions to generic problems, so that they can be reused many times: a solution to a very specific situation, in fact, can hardly be reused outside of that same situation.

Design patterns, thus, don't give us the specific implementation of a solution, but only the core of it: they describe the major parts of the implementation, and how they interact with each other. This solution, then, can be reused many times, but never the same way twice.

Many SDKs provide implementations of common design patterns in their libraries. However, often these implementations don't just fit our needs very well: in those cases, we need to come out with different implementations of the same design patterns.

### Observer pattern
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

#### Consequences of the Observer pattern
Since the notification from the subject may never come (because the state we are monitoring may happen to never change), observers should always be properly initialized, and they should never rely on getting a notification.

We may get repeated notifications, so observers should be prepared to deal with duplicated notifications. For instance, the observer may start a long process in reaction to the first notification: thus, the second, duplicated, notification may come when the process kicked off by the first one, is still running. The problem here is that we may need to start the process again, or to check if the state change is really relevant.

In certain circumstances, notifications can come very quickly, like hundreds or thousands of times per second. In these cases, it could be better to process only samples of notifications, like every two seconds.

Sicne the subject keeps a reference of every subscriber, after one subscriber is no longer used, it can't be disposed by the garbage collector, because a reference to it is still held by the subject (which tipically outlives observers).

### Event aggregators
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