[Jakarta EE] (formerly Java EE) is the primary web standard for Java.
It features a whole host of standards ranging from [JPA](Jakarta-EE-8-Persistence) to [XML Binding](Jakarta-EE-8-Bind) to
[Websockets](Jakarta-EE-8-WebSocket), which is what I want to talk about in this post.

Websockets were originally standardized in [RFC 6455] in december 2011 and were (quite speedily) adopted into the Java EE 7
specifications in may 2013. This specification includes a couple main things:

* Client Endpoint (either by annotation or inheritance)
* Server Endpoint (either by annotation or inheritance)
* Endpoint Configuration
* Message Encoding and Decoding
* Websocket Sessions

The specification versions 1.1 and 2.0 only added minor changes.
Version 1.1 added some methods for registering message handlers on a `Session` and version 2.0 moved to the `jakarta.*` namespace.

Implementing websockets in Java EE is now quite simple.
We can simply define a class to receive an open websocket connection and receive messages from it. We're also given an object to send messages:

```java
@javax.websocket.server.ServerEndpoint("/bananas")
public class BananasEndpoint {
    @javax.websocket.OnOpen
    public void onOpen(javax.websocket.Session s, javax.websocket.EndpointConfig endpointConfig) {
    }

    @javax.websocket.OnMessage
    public void onMessage(javax.websocket.Session s, String message) {
        // Simply echo back the message
        s.getBasicRemote().sendText(message);
    }

    @javax.websocket.OnClose
    public void onClose(javax.websocket.Session s, javax.websocket.CloseReason reason) {
    }

    @javax.websocket.OnError
    public void onError(javax.websocket.Session s, Throwable t){
    }
}
```

One might think that this is all the API you need, and you might, but I need more.
I would like to pull a bearer token from the headers in the handshake request.
You might notice that the `Session` object does not contain those headers, and neither does the `EndpointConfig` object.
We can pull those out of the handshake by defining an `EndpointConfigurator`:

```java
public class BananaEndpointConfigurator extends javax.websocket.server.ServerEndpointConfig.Configurator {
     @Override
     public void modifyHandshake(
             javax.websocket.server.ServerEndpointConfig config,
             javax.websocket.HandshakeRequest request,
             javax.websocket.HandshakeResponse response) {
         
         List<String> authHdrs = request.getHeaders().getOrDefault("Authorization", new java.util.ArrayList<>());
         
         if (authHdrs.size() == 1) {
             // Authenticate user here
             Object user = new Object();
             config.getUserProperties().put("User", user);
         }
     }
}
```

And then you can specify this on the endpoint class like this:

```java
@javax.websocket.server.ServerEndpoint("/bananas", configurator=BananasEndpointConfigurator.class)
public class BananasEndpoint {
    @javax.websocket.OnOpen
    public void onOpen(javax.websocket.Session s, javax.websocket.EndpointConfig endpointConfig) {
        if (!s.getUserProperties().contains("User")) {
            s.close(new javax.websocket.CloseReason(
                javax.websocket.CloseReason.CloseCodes.VIOLATED_POLICY,
                "Unauthenticated"
            ));
        }
    }

    @javax.websocket.OnMessage
    public void onMessage(javax.websocket.Session s, String message) {
        // Echo back the message prefixed with some user information
        s.getBasicRemote().sendText(String.format(
            "%s: %s",
            s.getUserProperties().get("User").toString(),
            message
        ));
    }

    @javax.websocket.OnClose
    public void onClose(javax.websocket.Session s, javax.websocket.CloseReason reason) {
    }

    @javax.websocket.OnError
    public void onError(javax.websocket.Session s, Throwable t){
    }
}
```

Now you might say: "But what if I want to send a 401 response during the handshake as opposed to closing the
socket with a VIOLATED_POLICY status?".
Well... That's a whole other story.
One might think they could simply throw a `new WebApplicationException("Unauthorized", 501)` in the `modifyHandshake` function,
but you can't.
Now, you will actually have to configure actual Java EE authentication.
Well... Let's go!

```java
@javax.enterprise.context.RequestScoped
public class BananaAuthenticationMechanism implements HttpAuthenticationMechanism {
    @Override
    public javax.security.enterprise.AuthenticationStatus validateRequest(
            javax.servlet.http.HttpServletRequest request,
            javax.servlet.http.HttpServletResponse response,
            javax.security.enterprise.authentication.mechanism.http.HttpMessageContext context)
            throws javax.security.enterprise.AuthenticationException {
        
        if (request.getHeader("Authorization").toLowerCase().contains("banana")) {
            return context.notifyContainerAboutLogin(new java.security.Identity("BananaUser"));
        }
        return context.responseUnauthorized();
    }
}
```

There! And now we can:

```java
@javax.websocket.server.ServerEndpoint("/bananas", configurator=BananasEndpointConfigurator.class)
public class BananasEndpoint {
    @javax.websocket.OnOpen
    public void onOpen(javax.websocket.Session s, javax.websocket.EndpointConfig endpointConfig) {
    }

    @javax.websocket.OnMessage
    public void onMessage(javax.websocket.Session s, String message) {
        // Echo back the message prefixed with some user information
        s.getBasicRemote().sendText(String.format(
            "%s: %s",
            s.getUserPrincipal().toString(),
            message
        ));
    }

    @javax.websocket.OnClose
    public void onClose(javax.websocket.Session s, javax.websocket.CloseReason reason) {
    }

    @javax.websocket.OnError
    public void onError(javax.websocket.Session s, Throwable t){
    }
}
```

Done! Now the handshake will return a 401 when authentication fails. That was a job... Pfew.
A nice side-effect is that we don't need that configurator thing anymore.
Personally, I would've preferred to throw a 401 in the configurator, or (even better) the ability
to run filters on the handshake, but that appears to be completely impossible.

Also note that you now need a servlet container that supports both the enterprise CDI spec and
and the enterprise security spec. Neither Tomcat nor Jetty do this (although I belive TomEE does).
You can use JBoss Weld and Glassfish Soteria as drop-ins though.

[Jakarta EE]: https://jakarta.ee/
[Jakarta EE 8]: https://jakarta.ee/specifications/platform/8/
[Jakarta-EE-8-Persistence]: https://jakarta.ee/specifications/persistence/2.2/
[Jakarta-EE-8-Bind]: https://jakarta.ee/specifications/xml-binding/2.3/
[Jakarta-EE-8-WebSocket]: https://jakarta.ee/specifications/websocket/1.1/

[RC 6455]: https://tools.ietf.org/html/rfc6455
