## REST APIs

The main characteristic of REST APIs is that they are stateless: this means that no client session is ever stored on the server side; instead, the session is always handled by the client itself. The main advantage of statelessness is that it allows us to back the service with any pattern of servers: if the service is delivered by multiple redundant servers, it being stateless guarantees that any other server can be used in case the current one is suddenly under heavy load. This of course will be much harder if dealing with session state, because now all sessions of all users need to be continuously replicated among all servers.

RESTful APIs should properly use HTTP methods according to their semantics:
- `GET` is for sending queries to the service, and retrieve an object or a list of objects. No side effect must occurr on the server, and thus all `GET` requests are idempotent, meaning that no matter how many times the same request is performed, the effect on the data is always the same. The result may not be always the same, of course, because I can make a query that returns more or less results if new data has been added or removed in the meanwhile.
- `POST` is for creating new objects. In particular, it should not be used to modify existing objects, because that one is another idempotent operation, and idempotence is not required for `POST`, since if I repeatedly call `POST`, new objects are repeatedly added.
- `DELETE` is for deleting existing objects. It should be idempotent, too, because if it's called multiple times, the end result with the data is that the first time the selected object is deleted, while the following time nothing happens, so the end result after multiple calls is always the same: that object is deleted. It's important to note that this is true only as long as the unique identifier of the object to delete is passed. A request like "delete the most recent object" would of course not be idempotent.
- `PUT` is to completely replace an object. It's not for changing only specific fields of an existing object, but rather to replace it all. Thus, with `PUT` the whole object should be passed, and not only the value of a field. `PUT` is idempotent, because no matter how many times it is called, the end result is always the same: the specified object is replaced with the given one. As `DELETE`, this stands only when the unique identifier of the object is passed.
- `PATCH` is to update an object, i.e. to change a subset of its fields. It's also idempotent (if the object is uniquely identified), because updating the same fields with the same values multiple times has of course always the same effect.

An important part of the design of a RESTful API is choosing the URL paths. A common principle is that of avoid using query parameters as much as possible, thus:

```
GET /search?type=User&id=3749
```

would be better designed as:

```
GET /search/user/3749
```

or, more generally, as:

```
GET /search/user/{id}
```

Typical operations provided by an API are the CRUD ones:
- Create
```
POST /user
{
	"name": "Alan Turing",
	"age": 32
}
```
```json
{
	"id": 3,
	"name": "Alan Turing",
	"age": 32
}
```
- Retrieve
```
GET /user/3
```
```json
{
	"id": 3,
	"name": "Alan Turing",
	"age": 32
}
```
- Update (replace)
```
PUT /user/3
{
	"name": "Roy Fielding"
}
```
```json
{
	"id": 3,
	"name": "Roy Fielding"
}
```
- Update (partial)
```
PATCH /user/3
{
	"age": 41
}
```
```json
{
	"id": 3,
	"name": "Alan Turing",
	"age": 41
}
```
- Delete
```
DELETE /user/3
```
```json
```

Additionally, we may want to get lists of results:

```
GET /user
```
```json
{
	"datetime": "2017-02-27T11:43:37+0500",
	"elements": 3,
	"data": [
		{
			"id": 1,
			"name": "Tim Berners-Lee",
			"age": 61
		},
		{
			"id": 2,
			"name": "Ada Lovelace",
			"age": 36
		},
		{
			"id": 3,
			"name": "Roy Fielding"
		}
	]
}
```

Pagination is a typical example of usage of query parameters:
```
GET /user?size=1&page=2
```
```json
{
	"datetime": "2017-02-27T11:43:37+0500",
	"elements": 1,
	"data": [
		{
			"id": 2,
			"name": "Ada Lovelace",
			"age": 36
		}
	],
	"page": {
		"size": 1,
		"page": 2,
		"total": 3
	}
}
```

RESTful APIs leverage HTTP response codes to communicate the resulting status of a requested operation, instead of letting the developers come up with their own convention inside the JSON response body. These are the semantics of the HTTP response code families:
- `1xx`: informational
- `2xx`: success
- `3xx`: redirection
- `4xx`: client error
- `5xx`: server error

At the bare minimum, `200 OK` should be returned on successful operations, and `500 Internal Server Error` should be returned in case of problems processing the request. If a request is sent to a non existent ID, `404 Not Found` is the code to return. If a request is missing a mandatory field, return `400 Bad Request`; a JSON response body may be also sent in this case, with additional information about the validation error.


### Resources
- https://blog.qmo.io/ultimate-guide-to-api-design/
- https://en.wikipedia.org/wiki/Idempotence


## Progressive enhancement

*Progressive enhancement* and *graceful degradation* are Web design techniques tackling the problem of supporting multiple kinds of user agents, that might have quite different capabilities enabled, delivering an usable site or application in all cases.

Graceful degradation is probably the easier solution to implement, because it starts from the idea that the most feature-rich product should be developed first, since it's the most appealing to the client and users, only to add some tweaks, or maybe disabling some functionality, later on when problems with different user agents are detected. This solution, however simple, is the most fragile, because many categories of quirks are usually left undetected, and because it leads to a messy and hard to extend and reuse codebase.

Progressive enhancement, instead, provides a perspective centered around the concept of content, or functionality, that must be delivered. The first development iterations are focused on delivering the content, or the core functionality, using only core Web technologies supported by all browsers, usually even with no CSS nor JavaScript. In later iterations, layer by layer, additional features are added, always checking if they are supported from the current user agent.

Progressive enhancement builds upon many core Web principles:
- *separation of concerns*: to allow CSS and JavaScript to be added only at later stages, it's mandatory that the HTML is written following the rules of semantic markup, thus being driven by the semantics of the content, instead of by the requirements of styles or scripts
- *accessibility* is also greately enhanced, because the core content is always guaranteed to be fully available, independently from styles and scripts being enabled, or usable (think of screen readers, ignoring scripts and styles)
- *unobstrusive JavaScript* is another core tenet of progressive enhancement, because JavaScript is only included as a surface addition, thus keeping the core content or functionality independent from it
- *Pperformance* and *usability* are also enhanced, because if the core content is always available, the site will be usable also from browsers having advanced features disabled (CSS, JavaScript), or mobile devices with slow or flaky internet connection, that might fail to download all styles and script files
- *defensive programming* is applied as soon as some CSS or JavaScript feature is added on top of the content, because they will always be tested for availability before being used.

Despite standing at opposite sides when it comes to Web design, progressive enhancement and graceful degradation can still be seen as the two sides of the same coin. In fact, when progressive enhancement is properly employed for the development of a Website or application, the end result, from the point of view of the users, will look just like it features graceful degradation, because moving from a modern user agent over a powerful connection, to an older browser over a slow connection, the Website will still be usable, just degrading to a state where less functionality is available.

The main realm where progressive enhancement shines is of course that of Websites, meaning those providing mainly content, as opposed to Web applications, where functionality and user interaction are the real product. The edge case is of course that of SPAs, where JavaScript simply must be available, because even the core HTML of the page is built with JavaScript on the client, instead of being downloaded from the server.

However, the use cases of hardcore SPAs are actually extremely scarce, because building everything on the client poses major performance issues. In most cases, most of the page can simply be dynamically generated from the server at the time of the first request, and then only the interactive components can use JavaScript to get the updated contents from the server with AJAX, and update themselves in real time. In addition to be much faster than the hardcore SPA technique, this allows progressive enhancement to be employed in SPAs (of this kind) as well: in fact, in those cases where JavaScript is not available (either because it's not enabled, or because some files failed to be downloaded), it would be easy to fall back to a classical request/response model for only those few interactive components that are present on the page.

### Resources
- https://en.wikipedia.org/wiki/Progressive_enhancement
- https://en.wikipedia.org/wiki/Unobtrusive_JavaScript
- https://en.wikipedia.org/wiki/Acid3
- https://alistapart.com/article/understandingprogressiveenhancement
- https://www.smashingmagazine.com/2015/12/reimagining-single-page-applications-progressive-enhancement/
- http://softwareengineering.stackexchange.com/questions/237537/progressive-enhancement-vs-single-page-apps
- http://molily.de/interaction-is-key/
- http://molily.de/single-page-apps/


## Progressive Web Applications

*Progressive Web Applications* refers to Web applications designed to seamlessly blend with the native applications of a device. To achieve this goal, PWAs need to:
- be reliable in all kind of connection situations, thus relying heavily on caching and local storage
- be as fast to respond to user input as native applications
- blend with native applications, thus providing a link from the home screen (or even from the applications list), and hiding the fact that it's running inside a browser

Due to the requirement of being able to work offline, not only must they provide strong caching and local storage leverage, they must also be smart enough to understand when they can ask the server for updates, or even to allow receiving push notifications from the server itself.

### Resources
- https://developers.google.com/web/progressive-web-apps/
- http://recordssoundthesame.com/blog/2017/01/27/a-practical-guide-to-pwas/
- https://auth0.com/blog/introduction-to-progressive-apps-part-one/
- https://auth0.com/blog/introduction-to-progressive-web-apps-instant-loading-part-2/


## Performance

### Resources
- https://hackernoon.com/10-things-i-learned-making-the-fastest-site-in-the-world-18a0e1cdf4a7#.87noadu3a


## CSS

### Resources
- http://maintainablecss.com/
- https://www.ckl.io/blog/css-architecture-first-steps/


## Other

### Resources
- https://medium.com/javascript-scene/top-javascript-frameworks-topics-to-learn-in-2017-700a397b711#.76xaw9rna
- https://hackernoon.com/type-checking-in-javascript-getting-started-with-flow-8532c11aceb3#.x4f55riw7
- https://ashleynolan.co.uk/blog/frontend-tooling-survey-2016-results