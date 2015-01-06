---
title: Big Bang Client SDK

language_tabs:
  - java
  - javascript

toc_footers:
  - <a href='http://getbigbang.com/'>Home page</a>
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

search: true
---

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

Big Bang hosts your the coordinating server for you. Simply connect your clients and apps to your Big Bang URL. You can use `http://demo.bigbang.io` to try things out. When you are ready, you can create your own application at [https://cloud.bigbang.io/](https://cloud.bigbang.io/).


Overview
========

You will work with three resources when using Big Bang. First, you will need to manage your connection to our servers. Once you have established a connection, you will subscribe to a Channel. All shared information is scoped to a Channel. You can publish and subscribe one-time messages. If you want to give all subscribers a constantly updated state of your data, you can publish and subscribe ChannelData.


#Connection
Connecting your app to Big Bang is easy.
##Basics
- url `string` HTTP or HTTPS URL to your application.
- options `object`
- callback (`Error`)

```javascript
var client = new BigBang.Client();
client.connect('http://demo.bigbang.io', function(err) {
    if (err) return;
    console.log('Connected as ' + client.getClientId());
});
```

```java
##      ## ##     ## ##    ##  #######  
##  ##  ## ##     ##  ##  ##  ##     ## 
##  ##  ## ##     ##   ####         ##  
##  ##  ## #########    ##        ###   
##  ##  ## ##     ##    ##       ##     
##  ##  ## ##     ##    ##              
 ###  ###  ##     ##    ##       ## 
```

### client.connect(url, options, function(err))
Connect to a Big Bang application at *url*.

**Params**

### client.disconnect()
Disconnect from the server.


### client.getClientId()
Your unique identifier for this session. This identifies you to the server and to other users.

Returns `string` clientId


##Subscribe
### client.on('disconnected', function(reason))
Fired when the client has been disconnected, either from calling disconnect() or for reasons beyond your control.



#Channel
Group together multiple clients in a channel to share information. Channels are publish/subscribe. You can subscribe to a Channel to get any messages that are published to it. You can publish a message to send it to all subscribers.
##Basics
### client.getChannel(channelName)
Get a reference to the Channel object for the subscribed channel called *channelName*.

**Params**

- channelName `string`

Returns `Channel` 


### client.subscribe(channelName, options, function(err, channel))
Subscribe to a 	channel called *channelName*. *channel* will be a Channel object.

**Params**

- channelName `string`
- options `object`
- callback (`Error`,`Channel`)


### channel.unsubscribe(function())
Unsubscribe from the current channel.


### channel.getSubscribers()
Returns an `Array` containing the clientIds of the current subscribers on this channel.


##Publish
### channel.publish(obj, function(err))
Publish *obj* to the channel. *obj* must be an object or array.

**Params**

- obj `object` (JSON) or `array`
- callback `Error` if publish fails or is rejected


##Subscribe
### channel.on('message', function(msg))
Fired when a message is received on the channel.


###  channel.on('join', function(clientId))
Fired when a subscriber joins the channel.


### channel.on('leave', function(clientId)) 
Fired when a subscriber leaves the channel.



#ChannelData
ChannelData objects are used to store the state of your data. ChannelData persist as long as the Channel is active and they are automatically synchronized to all subscribers of the channel.
##Basics
### channel.getNamespaces()
Get the current *ChannelData* namespace names as an Array.

Returns `ChannelData` or null if namespace does not exist.


### channel.getChannelData(namespace)
Returns a *ChannelData* object for the given namespace. If no namespace is supplied a default will be used. Namespaces can be used to organize your channel's data.

**Params**

- namespace `string`

Returns `ChannelData`


### channelData.get(key)

**Params**

- key `string`

Returns the `object` (JSON) or `Array` value associated with key, or null if the key does not exist.


##Publish
### channelData.put(key, value)
Set the *value* for *key*.

**Params**

- key `string`
- value `object` (JSON) or `Array`


##Subscribe
### channelData.on('add', function(key, value))
Fires when a new key and value is added.


### channelData.on('update', function(key, value))
Fires when a key's value is updated.


### channelData.on('remove', function(key))
Fired when a key (and it's value) is removed.


### channelData.on(key, function(value, operation))
Fired when anything happens to key. *value* will be the new value or null in the case of a remove. *operation* is one of add, update or remove. This event is an easy way to monitor a single key.


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
	
