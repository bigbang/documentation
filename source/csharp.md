Big Bang Client SDK
=================

Big Bang lets you create realtime applications in seconds.  It makes event streaming and data synchronization a snap!


Installation
============

    ##TODO direct download link
or

    ##TODO nuget


Servers
=======

Big Bang manages your realtime infrastructure for you. Simply connect your clients and apps to your Big Bang URL. You can use `http://demo.bigbang.io` to try things out. When you are ready, you can create your own application at [https://cloud.bigbang.io/](https://cloud.bigbang.io/).


Overview
========

You will work with three resources when using Big Bang. First, you will need to manage your connection to our servers. Once you have established a connection, you will subscribe to a Channel. All shared information is scoped to a Channel. You can publish and subscribe one-time messages. If you want to give all subscribers a constantly updated state of your data, you can publish and subscribe ChannelData.


#Connection
Connecting your app to Big Bang is easy.
##Basics

```csharp
BigBangClient client = new DefaultBigBangClient ();

client.Connect ("https://demo.bigbang.io", (err) => {

    if (null != err) {
        Console.WriteLine (err);
    } else {
        Console.WriteLine ("Connected as " + client.ClientId);
    }
});
```

### client.Connect(string url, Action\<ConnectionError> connectHandler)
Connect to a Big Bang application at *url*.

**Params**

- url `string` HTTP or HTTPS URL to your application.
- callback (`Error`)


### client.Disconnect()
Disconnect from the server.


### client.GetClientId()
Your unique identifier for this session. This identifies you to the server and to other users.

Returns `string` clientId


##Subscribe
```csharp
client.Subscribe ("example-channel", (err, channel) => {
    if (null != err) {
        //We had a problem subscribing.
        Console.WriteLine (err);
        return;
    } else {
        Console.WriteLine ("Subscribed to channel " + channel.Name);
    }
});
```
##Disconnect
```charp
client.Disconnected( () => Console.WriteLine("Client disconnected!"));
```
### client.Disconnected(Action callback)

Fired when the client has been disconnected, either from calling disconnect() or for reasons beyond your control.
#Channel
Group together multiple clients in a channel to share information. Channels are publish/subscribe. You can subscribe to a Channel to get any messages that are published to it. You can publish a message to send it to all subscribers.
##Basics
```csharp
Channel channel = client.GetChannel ("example-channel");
```
### client.GetChannel(string channelName) 
Get a reference to the Channel object for the subscribed channel called *channelName*.

**Params**

- channelName `string`

Returns `Channel`

```csharp
client.Subscribe ("example-channel", (err, channel) => {
    if (null != err) {
        //We had a problem subscribing.
        Console.WriteLine (err);
        return;
    } else {
        Console.WriteLine ("Subscribed to channel " + channel.Name);
    }
});
```
### client.Subscribe(string channelName, Action<ChannelError, Channel> callback) 
Subscribe to a  channel called *channelName*. *channel* will be a Channel object.

**Params**

- channelName `string`
- callback (`ChannelError`,`Channel`)


### channel.Unsubscribe(Action callback)
Unsubscribe from the current channel.


### channel.GetSubscribers()
Returns an `List<string>` containing the clientIds of the current subscribers on this channel.


##Publish

```csharp
JsonData json = new JsonData ();
json ["message"] = "hello";

channel.Publish (json);
```csharp

### channel.Publish(JsonData content, Action\<ChannelError> callback)
Publish *content* to the channel. *content* must be an object or array.

**Params**

- content `object` (JSON)
- callback `ChannelError` if publish fails or is rejected


##Subscribe

```csharp
channel.OnMessage ((msg) => {
    Console.WriteLine (msg.Payload.GetBytesAsJSON ());
});
```
### channel.OnMessage(Action<ChannelMessage> handler)
Fired when a message is received on the channel.


```csharp
channel.OnJoin( (joined) => {
    Console.WriteLine( "clientId " + joined + " joined the channel.");
};

```
###  channel.OnJoin(Action<string> join)
Fired when a subscriber joins the channel.

```csharp
channel.OnLeave( (left) => {
    Console.WriteLine( "clientId " + left  +" left the channel.");
};
```
### channel.OnLeave(Action<string> leave) 
Fired when a subscriber leaves the channel.



#ChannelData
ChannelData objects are used to store the state of your data. ChannelData persist as long as the Channel is active and they are automatically synchronized to all subscribers of the channel.
##Basics
### channel.GetNamespaces()
Get the current *ChannelData* namespace names as an `List<string>`.

Returns `ChannelData` unless no namespaces exist. Returns null if no namespaces exist.


### channel.GetChannelData()
Returns a *ChannelData* object for the default namespace.


### channel.GetChannelData(namespace)
Returns a *ChannelData* object for the given namespace. Namespaces can be used to organize your channel's data.

**Params**

- namespace `java.lang.String namespace`

Returns `ChannelData`

```csharp
JsonData val = channelData.Get("myKey");
```
### channelData.Get(string key)

**Params**

- key `string`

Returns the type `JsonData` unless the key doesn't exist. Returns null if the key doesn't exist. 


##Publish

```csharp
JsonData m = new JsonData ();
m ["message"] = "hello channeldata!";
channelData.Put ("myKey", m);
```
### channelData.Put(string key, JsonData value)
Set the *value* for *key*.

**Params**

- key `string`
- value `JsonData`


##Subscribe
```csharp
channelData.OnAdd ((key, val) => Console.WriteLine ("added " + key + " => " + val));
```
### channelData.OnAdd(Action<string, JsonData> add)
Fires when a new key and value is added.


```csharp
channelData.OnUpdate ((key, val) => Console.WriteLine ("updated " + key + " => " + val));
```
### channelData.OnUpdate(Action<string, JsonData> update)
Fires when a key's value is updated.

```csharp
channelData.OnRemove ((key) => Console.WriteLine ("removed " + key));
```
### channelData.OnRemove(Action\<string> remove)
Fired when a key (and it's value) is removed.

```csharp
channelData.On("myKey", (val,op) => Console.WriteLine("key operation is " + op));
```
### channelData.On(string key, Action<JsonData, string> value)
Fired when anything happens to key. *value* will be the new value, except in the case of a remove *operation* returning null instead. *operation* is one of add, update or remove. This event is an easy way to monitor a single key.


##Maintenance
### channelData.Del(string key)
Remove the value associated with *key*.





==============
Example
=======

  var client = new BigBang.Client();
  client.connect('http://demo.bigbang.io', function(err) {
      if (err) return;
      console.log('Connected as ' + client.getClientId());
      
      client.subscribe('my-channel', function(err, channel) {
          if (err) return;
          channel.on('message', function(message) {
              console.log(message.senderId + ' said ' + message.payload.getBytesAsJSON().message);
          });         
          channel.publish({ message : 'Hi everybody!' });

          var channelData = channel.getChannelData();
          channelData.on('add', function(key, value) {
              console.log('Someone set ' + key + ' to ' + value);
          });
          channelData.put('colors', ['red', 'green', 'blue']);
      });
  });







API
===

BigBang.Client
--------------

Client manages your connection to the server and lets you interface with Channels.

  var client = new BigBang.Client();
  client.connect('http://demo.bigbang.io', function(err) {
      if (err) return;
      console.log('Connected as ' + client.getClientId());
  });
  



BigBang.Channel
---------------

    client.subscribe('my-channel', function(err, channel) {
        if (err) return;
        channel.on('message', function(message) {
            console.log(message.senderId + ' said ' + message.payload.getBytesAsJSON().message);
        });         
        channel.publish({ message : 'Hi everybody!' });
    });





BigBang.ChannelData
-------------------

    var channelData = channel.getChannelData();
    channelData.on('add', function(key, value) {
        console.log('Someone set ' + key + ' to ' + value);
    });
    channelData.put('colors', ['red', 'green', 'blue']);
  