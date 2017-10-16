Microservices are distinct services used together to assemble a single application. These services run on their own process, and talk together with a lightweight mechanism, such as HTTP requests and responses, and are thus fully decoupled. Each service has its own layers, like data persistence, caching, and business logic. They can be written with different programming languages, use their own data storage technologies and be deployed indipendently.

In general different services could be located under different domains: for example you could login at `myapplogin.com` and get redirected to `myapporders.com` to see your orders. However, to make this work the browser should be always compliant to the CORS standard, sending an HTTP `OPTIONS` request header each time to ask which HTTP methods are supported by the new domain, and only after that making the actual requests. Furthermore, this would be impossible to do with browser not supporting CORS and adopting the *same-origin security policy*. To avoid all this complication, services are usually located under the same domain.

Every request coming from the client is catched first and foremost by the load balancer, whose role, other than dealing with performance optimization, is also to understand which service a request belongs to. For instance, to which service a request like `http://example.com/do/something` should be mapped? We could create some kind of routing table, but an easiest way is to use well structured service names, like:

```
http://example.com/[service_identifier]/[service_specific_path]
```

Once the balancer identifies the service using the first part of the path, the second part can just be sent to it, since the actual service knows how to handle it. For instance:

```
http://example.com/catalog_service/products/123
http://example.com/user_service/users/3341
http://example.com/orders_service/orders/99812
```

Services talk to each other using inter-process communication means. The most common one is via HTTP, sending structured data like JSON. Services using HTTP messaging need to use both HTTP socket client and server, and this type of messaging is usually synchronous, because a service sending an HTTP request to another service needs to wait until its response arrives before proceeding. To have an asynchronous communication one, the client service usually employs a message broker like RabbitMQ or Redis. Maybe it could be possible to have the server service send push notifications to the client, avoiding it to wait for a response.

There are two ways to use a message broker to implement asynchronous communication: via *choreography* or *orchestration*.

Say we have a component that models the inventory of a store. An external software notifies this component about products' availability. Each time information about a product is received, not only the inventory must be updated, but also the catalog and data related to users (whishlists for instance). We have, thus, at least three services: the inventory service, the catalog service and the user service. All these three use a message broker to send and receive messages to and from the outside, so that we have three queues: `inventory_service_queue`, `catalog_service_queue` and `user_service_queue`. After the inventory service has processed information about the new product, it should add a message into the three queues, so that the other services can deal with the new information as well.

In the choreography architecture, there is only one kind of message that is being sent to the three queues, so every queue receives the same message. The inventory service is itself one of the consumer of that message: this is because it could need to perform some extra calculations asynchronously; the catalog service updates availability of products in the catalog based on the information coming from the inventory; the user service sends email notification when a wishlisted product becomes available.

Since there's only one kind of message being sent, the inventory service is decoupled from the need of knowing what each other service is going to do with that information: it doesn't need to know that the wishlist has to be updated, for example, or that products need to be set as available or not.

In the orchestration architecture, on the other hand, each queue is being sent a message specific to it. For example, the `catalog_service_queue` is being sent a `sku_set_availability` message, while the `user_service_queue` is being sent a `sku_update_wishlist` message. This forces the sender to know what each service should do in response of this event, and send the right message to everyone.