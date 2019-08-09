Restful Services
---

I'm sure not many people know about how the browser security plays with the Restful webservices, but here is a small summary of it.

I've tested the below rule with Chrome and Firefox

While creating a RESTFul web service, keep the following in mind:
1. Create [Etag](http://en.wikipedia.org/wiki/HTTP_ETag)
2. Support the following methods: `GET`, `POST`, `HEAD`, `OPTIONS`, `PUT`, `DELETE`

`ETag` is one of several mechanisms that HTTP provides for web cache validation, and which allows a client to make conditional requests. This allows caches to be more efficient, and saves bandwidth, as a web server does not need to send a full response if the content has not changed. ETags can also be used for optimistic concurrency control, as a way to help prevent simultaneous updates of a resource from overwriting each other.

The reason to support the above methods are:
- `GET` - Used to fetch a list/single entity from the server
- `POST` - Create a new entity with the parameters supplied on the server
- `PUT` - Update the entity with the values specified and the ETag
- `DELETE` - Delete the entity by specifying the ID and the ETag
- `HEAD` - Retrieve the ETag for the entity created. Always append the ID to the URL
- `OPTIONS` - provide all the headers used to make the cross site calls successful

Most of operations performed using Javascript ajax calls, first call the `OPTIONS` methods to check if the server supports the calls, and then on receiving the headers the client browser is interest in, the actual `GET`/`POST`/`PUT`/`DELETE` call is made.

Inorder to allow the cross site calls successful, make sure that the OPTIONS returns the following HEADERS populated:
```
Access-Control-Allow-Origin
Access-Control-Allow-Methods
Access-Control-Allow-Headers
Access-Control-Max-Age
```

Once the response is populated with these HEADERS, you should be all set!!!

Hope this was helpful!!

References:
- http://blog.furiousbob.com/2011/12/06/hateoas-restful-services-using-spring-3-1/

Please look at `Spring HATEOS` for you implement something of your own.
