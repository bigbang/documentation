Big Bang Client SDK
=================

Big Bang lets you create realtime applications in seconds.  It makes event streaming and data synchronization a snap!


Installation
============

    ##TODO put in a maven repository.  Also with ponies.
or

    ##TODO put in a gradle repository
or

    ##TODO direct download link


Servers
=======

Big Bang manages your realtime infrastructure for you. Simply connect your clients and apps to your Big Bang URL. You can use `http://demo.bigbang.io` to try things out. When you are ready, you can create your own application at [https://cloud.bigbang.io/](https://cloud.bigbang.io/).


Overview
========

You will work with three resources when using Big Bang. First, you will need to manage your connection to our servers. Once you have established a connection, you will subscribe to a Channel. All shared information is scoped to a Channel. You can publish and subscribe one-time messages. If you want to give all subscribers a constantly updated state of your data, you can publish and subscribe ChannelData.


#Connection
Connecting your app to Big Bang is easy.
##Basics

```java
final BigBangClient client = new DefaultBigBangClient();

client.connect("https://demo.bigbang.io", new Action<ConnectionError>() {
    @Override
    public void result(ConnectionError err) {
        if (err != null) {
            System.err.println(err);
        } else
            System.out.println("Connected as " + client.getClientId());
    }
});
```

### client.connect(java.lang.String url, Action\<ConnectionError> connectHandler)
Connect to a Big Bang application at *url*.

**Params**

- url `string` HTTP or HTTPS URL to your application.
- callback (`Error`)


### client.disconnect()
Disconnect from the server.


### client.getClientId()
Your unique identifier for this session. This identifies you to the server and to other users.

Returns `string` clientId


##Subscribe

```java
client.subscribe("example-channel", new Action2<ChannelError, Channel>() {
    @Override
    public void result(ChannelError err, Channel channel) {
      if (err != null) {
            System.err.println(err);
        } else {
            System.out.println("Subscribed to channel " + channel.getName());
      }
    }
});
```

##Disconnect

```java
client.disconnected(new Action<Void>() {
    @Override
    public void result(Void result) {
       System.out.println("Client disconnected");
    }
});
```
### client.disconnected(Action<java.lang.Void> callback)




Fired when the client has been disconnected, either from calling disconnect() or for reasons beyond your control.



#Channel
Group together multiple clients in a channel to share information. Channels are publish/subscribe. You can subscribe to a Channel to get any messages that are published to it. You can publish a message to send it to all subscribers.
##Basics
```java
Channel channel = client.getChannel("example-channel");
```
### client.getChannel(java.lang.String channelName)
Get a reference to the Channel object for the subscribed channel called *channelName*.

**Params**

- channelName `string`

Returns `Channel`

```java
client.subscribe("example-channel", new Action2<ChannelError, Channel>() {
    @Override
    public void result(ChannelError err, Channel channel) {
      if (err != null) {
            System.err.println(err);
        } else {
            System.out.println("Subscribed to channel " + channel.getName());
      }
    }
});
```
### client.subscribe(java.lang.String channelName, Action2<ChannelError, Channel> callback)
Subscribe to a  channel called *channelName*. *channel* will be a Channel object.

**Params**

- channelName `string`
- callback (`ChannelError`,`Channel`)


### channel.unsubscribe(Action\<Void> callback)
Unsubscribe from the current channel.


### channel.getSubscribers()
Returns an `java.util.Set<java.lang.String>` containing the clientIds of the current subscribers on this channel.


##Publish
```java
JsonObject json = new JsonObject();
json.putString("message", "hello");

channel.publish(json);
```

### channel.publish(io.bigbang.protocol.JsonObject content, Action\<ChannelError> callback)
Publish *content* to the channel. *content* must be an object or array.

**Params**

- content `object` (JSON)
- callback `ChannelError` if publish fails or is rejected


##Subscribe
```java
channel.onMessage(new Action<ChannelMessage>() {
    @Override
    public void result(ChannelMessage msg) {
        System.out.println(msg.getPayload().getBytesAsJSON());
    }
});
```
### channel.onMessage(Action<ChannelMessage> handler)
Fired when a message is received on the channel.

```java
channel.onJoin(new Action<String>() {
    @Override
    public void result(String result) {
        System.out.println("clientId " + result + " joined the channel.");
    }
});
```
###  channel.onJoin(Action<java.lang.String> join)
Fired when a subscriber joins the channel.


```java
channel.onLeave(new Action<String>() {
    @Override
    public void result(String result) {
        System.out.println("clientId " + result + " left the channel.");
    }
});
```
### channel.onLeave(Action<java.lang.String> leave)
Fired when a subscriber leaves the channel.



#ChannelData
ChannelData objects are used to store the state of your data. ChannelData persist as long as the Channel is active and they are automatically synchronized to all subscribers of the channel.
##Basics
### channel.getNamespaces()
Get the current *ChannelData* namespace names as an `java.util.Set<java.lang.String>`.

Returns `ChannelData` unless no namespaces exist. Returns null if no namespaces exist.


```java
ChannelData channelData = channel.getChannelData();
```
### channel.getChannelData()
Returns a *ChannelData* object for the default namespace.

```java
ChannelData channelData = channel.getChannelData('my-namespace');
```
### channel.getChannelData(namespace)
Returns a *ChannelData* object for the given namespace. Namespaces can be used to organize your channel's data.

**Params**

- namespace `java.lang.String namespace`

Returns `ChannelData`


### channelData.get(java.lang.String key)

**Params**

- key `java.lang.String`

Returns the type `io.bigbang.protocol.JsonElement` unless the key doesn't exist. Returns null if the key doesn't exist.


### channelData.get(java.lang.String key, java.lang.Class\<T> type)

**Params**

- key `java.lang.String`

Returns the type `Class<T>` unless the key doesn't exist. Returns null if the key doesn't exist.


##Publish

```java
JsonObject msg = new JsonObject();
msg.putString("message", "hello channeldata!");
channelData.put("myKey", msg);
```
### channelData.put(java.lang.String key, io.bigbang.protocol.JsonElement value)
Set the *value* for *key*.

**Params**

- key `java.lang.String`
- value `io.bigbang.protocol.JsonElement`


##Subscribe

```java
channelData.onAdd(new Action2<String, JsonElement>() {
    @Override
    public void result(String key, JsonElement val) {
        System.out.println("added " + key + " => " + val);
    }
});
```
### channelData.onAdd(Action2<String, JsonElement> add)
Fires when a new key and value is added.

```java
channelData.onUpdate(new Action2<String, JsonElement>() {
    @Override
    public void result(String key, JsonElement val) {
        System.out.println("updated " + key + " => " + val);
    }
});
```
### channelData.onUpdate(Action2<String, JsonElement> update)
Fires when a key's value is updated.

```java
channelData.onRemove(new Action<String>() {
    @Override
    public void result(String key) {
        System.out.println("removed " + key);
    }
});
```
### channelData.onRemove(Action\<String> remove)
Fired when a key (and it's value) is removed.


```java
channelData.on("myKey", new Action2<JsonElement, ChannelData.Operation>() {
    @Override
    public void result(JsonElement e, ChannelData.Operation op) {
        System.out.println("key operation is " + op);
    }
});
```
### channelData.on(String key, Action2<JsonElement, String> value)
Fired when anything happens to key. *value* will be the new value, except in the case of a remove *operation* returning null instead. This event is an easy way to monitor a single key.


##Maintenance

```java
channelData.remove("myKey");
```
### channelData.del(java.lang.String key)
Remove the value associated with *key*.
