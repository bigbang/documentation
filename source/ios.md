Big Bang Client SDK
=================

The Big Bang Client SDK for iOS and OSX helps you create realtime applications in seconds!  It makes event streaming and data synchronization a snap!


Installation - XCode
============

The Big Bang SDK is provided as a Swift Framework and is compatible with Swift and Objective-C projects for iOS and OSX.  The easiest way to get started is by using [Cocoapods](https://cocoapods.org/) to help manage your dependencies.

Once you have added Cocoapods to your project, add the BigBang Framework as a dependency.


##Podfile

Add the dependency for BigBang in your Podfile

```
target 'MyApp' do
  pod 'BigBang', '~> 0.0.1'
end
```
##Install the depndencies

Open up your terminal, switch to your workspace directory, and install.

```bash
pod install
```


Servers
=======

Big Bang manages your realtime infrastructure for you. Simply connect your clients and apps to your Big Bang URL. You can use `http://demo.bigbang.io` to try things out. When you are ready, you can create your own application at [https://www.getbigbang.com/](https://www.getbigbang.com/#pricing).


Overview
========

You will work with three resources when using Big Bang. First, you will need to manage your connection to our servers. Once you have established a connection, you will subscribe to a Channel. All shared information is scoped to a Channel. You can publish and subscribe one-time messages. If you want to give all subscribers a constantly updated state of your data, you can publish and subscribe ChannelData.


#Connection
Connecting your app to Big Bang is easy.
##Basics

```swift
var client = DefaultBigBangClient(appURL: "https://demo.bigbang.io")
client?.connect({ (err) -> Void in
    if let connectErr = err  {
        println("Connection error: " + connectErr)
    }
    else {
        println("Connected!")
    }
})
```

### client.connect(callback:ConnectCallback)
Connect to your Big Bang application.

**Params**
- callback (`Error`)


### client.disconnect()
Disconnect from the server.


### client.getClientId() -> String
Your unique identifier for this session. This identifies you to the server and to other users.

Returns `String` clientId


##Subscribe

```swift
client.subscribe( channelName, callback:{ (err, channel) in
    if let subscribeErr = err  {
        println("Subscribe error: " + subscribeErr)
    }
    else {
        println("Subscribed to " + channel.name)
    }
})
```

##Disconnect

```swift
client.disconnected({() -> Void in
    println("Disconnected!")
});
```
### disconnected( callback: DisconnectCallback ) -> Void


Fired when the client has been disconnected, either from calling disconnect() or for reasons beyond your control.

#Channel
Group together multiple clients in a channel to share information. Channels are publish/subscribe. You can subscribe to a Channel to get any messages that are published to it. You can publish a message to send it to all subscribers.
##Basics
```swift
var channel = client.getChannel("example-channel");
```
### client.getChannel( name:String ) -> Channel?
Get a reference to the Channel object for the subscribed channel called *name*.

**Params**

- name `String`

Returns `Channel`

```swift
client.subscribe("test_channel", callback: { (cerr, channel) -> Void in
    
    if let subscribeErr  = cerr  {
        println("Subscribe error: " + subscribeErr )
    }
    else {
        println("Subscribed to " + channel!.name );
    }
})
```
### client.subscribe( name: String, callback: SubscribeCallback) -> Void
Subscribe to a  channel called *name*. *channel* will be a Channel object.

**Params**

- name `String`
- callback (`ChannelError`,`Channel`)


### channel.unsubscribe(Action&lt;Void> callback)
Unsubscribe from the current channel.


### channel.getSubscribers() -> [String]
Returns a `[String]` containing the clientIds of the current subscribers on this channel.


##Publish
```swift
var json = JSON.newJSONObject()
json["message"] = "hello"

channel!.publish(json)
```

### channel.publish(message:JSON) ->Void
Publish *content* to the channel. *content* must be a JSON object or array.

**Params**

- content `JSON` (JSON)


##Subscribe
```swift
channel!.onMessage({ (channelMessage) in
    println(channelMessage.payload.getBytesAsJson())
})
```
### channel.onMessage( callback: MessageCallback) ->Void
Fired when a message is received on the channel.

```swift
channel!.onJoin({(joined) in
    println("clientId " + joined + " joined the channel.")
})
```
###  channel.onJoin( callback: PresenceCallback) -> Void
Fired when a subscriber joins the channel.


```swift
channel!.onLeave({(left) in
    println("clientId " + left + " left the channel.")
})
```
### channel.onLeave( callback: PresenceCallback ) -> Void
Fired when a subscriber leaves the channel.



#ChannelData
ChannelData objects are used to store the state of your data. ChannelData persist as long as the Channel is active and they are automatically synchronized to all subscribers of the channel.
##Basics
### channel.getNamespaces()
Get the current *ChannelData* namespace names as an `java.util.Set<java.lang.String>`.

Returns `ChannelData` unless no namespaces exist. Returns null if no namespaces exist.


```swift
ChannelData channelData = channel.getChannelData();
```
### channel.getChannelData()
Returns a *ChannelData* object for the default namespace.

```swift
ChannelData channelData = channel.getChannelData("my-namespace");
```
### channel.getChannelData(String namespace)
Returns a *ChannelData* object for the given namespace. Namespaces can be used to organize your channel's data.

**Params**

- namespace `java.lang.String namespace`

Returns `ChannelData`


### channelData.get(String key)

**Params**

- key `java.lang.String`

Returns `io.bigbang.protocol.JsonElement` unless the key doesn't exist. Returns null if the key doesn't exist.


```swift
JsonObject json = channelData.get("myKey", JsonObject.class);
```
### channelData.get(java.lang.String key, java.lang.Class&lt;T> type)

**Params**

- key `java.lang.String`

Returns the type `Class<T>` unless the key doesn't exist. Returns null if the key doesn't exist.


##Publish

```swift
JsonObject msg = new JsonObject();
msg.putString("message", "hello channeldata!");
channelData.put("myKey", msg);
```
### channelData.put(String key, JsonElement value)
Set the *value* for *key*.

**Params**

- key `java.lang.String`
- value `io.bigbang.protocol.JsonElement`


##Subscribe

```swift
channelData.onAdd(new Action2<String, JsonElement>() {
    @Override
    public void result(String key, JsonElement val) {
        System.out.println("added " + key + " => " + val);
    }
});
```
### channelData.onAdd(Action2&lt;String, JsonElement> add)
Fires when a new key and value is added.

```swift
channelData.onUpdate(new Action2<String, JsonElement>() {
    @Override
    public void result(String key, JsonElement val) {
        System.out.println("updated " + key + " => " + val);
    }
});
```
### channelData.onUpdate(Action2&lt;String, JsonElement> update)
Fires when a key's value is updated.

```swift
channelData.onRemove(new Action<String>() {
    @Override
    public void result(String key) {
        System.out.println("removed " + key);
    }
});
```
### channelData.onRemove(Action&lt;String> remove)
Fired when a key (and it's value) is removed.


```swift
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

```swift
channelData.remove("myKey");
```
### channelData.remove(java.lang.String key)
Remove the value associated with *key*.

**Params**

- key `java.lang.String`

