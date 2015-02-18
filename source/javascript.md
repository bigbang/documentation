Big Bang Client SDK
=================

Big Bang lets you create realtime applications in seconds.  It makes event streaming and data synchronization a snap!


Installation
============

    npm install bigbang.io --save
    
or

    bower install bigbang.io --save

Servers
=======

Big Bang manages your realtime infrastructure for you. Simply connect your clients and apps to your Big Bang URL. You can use `http://demo.bigbang.io` to try things out. When you are ready, you can create your own application at [https://cloud.bigbang.io/](https://cloud.bigbang.io/).


Overview
========

You will work with three resources when using Big Bang. First, you will need to manage your connection to our servers. Once you have established a connection, you will subscribe to a Channel. All shared information is scoped to a Channel. You can publish and subscribe one-time messages. If you want to give all subscribers a constantly updated state of your data, you can publish and subscribe ChannelData.


#Connection
Connecting your app to Big Bang is easy.
##Basics
```javascript
var client = new BigBang.Client();
client.connect('https://demo.bigbang.io', function (err) {
    if (err) {
        return;
    }
    console.log('Connected as ' + client.getClientId());
});
```
### client.connect(url, function(err))
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
```javascript
client.subscribe('example-channel', function (err, channel) {
    if (err) {
        return;
    }
    console.log('Subscribed to channel ' + channel.getName());
});
```
##Disconnect

```javascript
client.on('disconnected', function () {
    console.log('Client disconnected.');
});
```
### client.on('disconnected', function(reason))

Fired when the client has been disconnected, either from calling disconnect() or for reasons beyond your control.

#Channel
Group together multiple clients in a channel to share information. Channels are publish/subscribe. You can subscribe to a Channel to get any messages that are published to it. You can publish a message to send it to all subscribers.
##Basics
```javascript
var channel = client.getChannel('example-channel');
```
### client.getChannel(channelName)
Get a reference to the Channel object for the subscribed channel called *channelName*.

**Params**

- channelName `string`

Returns `Channel` 


```javascript
client.subscribe('example-channel', function (err, channel) {
    if (err) {
        return;
    }
    console.log('Subscribed to channel ' + channel.getName());
});
```
### client.subscribe(channelName, function(err, channel))
Subscribe to a  channel called *channelName*. *channel* will be a Channel object.

**Params**

- channelName `string`
- callback (`Error`,`Channel`)


### channel.unsubscribe(function())
Unsubscribe from the current channel.


### channel.getSubscribers()
Returns an `Array` containing the clientIds of the current subscribers on this channel.


##Publish
```javascript
channel.publish({ message: 'hello' });
```
### channel.publish(obj, function(err))



Publish *obj* to the channel. *obj* must be an object or array.

**Params**

- obj `object` (JSON) or `array`
- callback `Error` if publish fails or is rejected


##Subscribe

```javascript
channel.on('message', function (msg) {
   console.log(JSON.stringify(msg.payload.getBytesAsJSON()));
});
```
### channel.on('message', function(msg))



Fired when a message is received on the channel.


```javascript
channel.on('join', function(joined) {
   console.log('clientId ' + joined + ' joined the channel.');
});
```
###  channel.on('join', function(clientId))

Fired when a subscriber joins the channel.



```javascript
channel.on('leave', function(left) {
    console.log('clientId ' + left + ' left the channel.');
});
```
### channel.on('leave', function(clientId)) 



Fired when a subscriber leaves the channel.



#ChannelData
ChannelData objects are used to store the state of your data. ChannelData persist as long as the Channel is active and they are automatically synchronized to all subscribers of the channel.
##Basics
### channel.getNamespaces()
Get the current *ChannelData* namespace names as an Array.

Returns `ChannelData` unless no namespaces exist. Returns null if no namespaces exist.

```javascript
var channelData = channel.getChannelData();
```
### channel.getChannelData(namespace)
Returns a *ChannelData* object for the given namespace. If no namespace is supplied a default will be used. Namespaces can be used to organize your channel's data.

**Params**

- namespace `string`

Returns `ChannelData`

```javascript
var val = channelData.get('myKey');
```
### channelData.get(key)

**Params**

- key `string`

Returns the `object` (JSON) or `Array` value associated with key, or null if the key does not exist.


##Publish

```javascript
channelData.put('myKey', {message: 'hello channeldata!'});
```
### channelData.put(key, value)
Set the *value* for *key*.

**Params**

- key `string`
- value `object` (JSON) or `Array`


##Subscribe

```javascript
channelData.on('add', function (key, val) {
   console.log('added ' + key + ' => ' + JSON.stringify(val));
});
```

### channelData.on('add', function(key, value))
Fires when a new key and value is added.


```javascript
channelData.on('update', function (key, val) {
   console.log('updated ' + key + ' => ' + JSON.stringify(val));
});
```

### channelData.on('update', function(key, value))
Fires when a key's value is updated.

```channelData.on('remove', function (key) {
   console.log('removed ' + key);
});
```
### channelData.on('remove', function(key))
Fired when a key (and it's value) is removed.

```javascript
channelData.on('myKey', function(val, op) {
   console.log( 'key operation is ' + op );
});
```
### channelData.on(key, function(value, operation))
Fired when anything happens to key. *value* will be the new value, except in the case of a remove *operation* returning null instead. *operation* is one of add, update or remove. This event is an easy way to monitor a single key.


##Maintenance
### channelData.remove(key)
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
  