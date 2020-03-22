## Overview

The ```lib/connection/Connection``` class is central to the SDK's object model. It allows you to:

* Open and close WebSocket connections to Barchart's servers,
* Subscribe and unsubscribe from market data streams,
* Automatically track market state, and
* Access the current market state

## Instantiation

The ```Connection``` class constructor does not accept any arguments:

	const connection = new Connection();

In general, you'll want to treat the ```Connection``` as a singleton. You only need one instance. In fact, Barchart's servers typically reject simultaneous connections using the same credentials.

## Monitoring

After you create a connection, you'll probably want to monitor state changes.

You can do this by subscribing to the *events* topic, passing a callback. The callback will be invoked each time the connection's state changes.

	const eventsHandler = (data) => {
		// Invoked when connection state changes
	}

	connection.on('events', eventsHandler);

The callback will receive an object with a *event* property, as follows:

	{
		event: 'login success'
	}

Possible *event* values are:

* *login success* - Generated after calling ```Connection.connect```, remote server accepted credentials
* *login fail* - Generated after calling ```Connection.connect```, remote server rejected credentials
* *disconnecting* - Generated after calling ```Connection.disconnect```
* *disconnect* - Generated after loss of connection -- whether intentional or not
* *feed paused* - Generated after calling ```Connection.pause```
* *feed resumed* - Gernated after calling ```Connection.resume```

## Opening

Calling ```Connection.connect``` opens a connection. The function requires that you specify a hostname for Barchart's servers and credentials.

**Contact Barchart at solutions@barchart.com or (866) 333-7587 to obtain the correct hostname and a free username/password.**

You'll also need to provide a strategy for creating WebSocket connections. Depending on your environment, different strategies are used. The ```lib/connection/adapter/WebSocketAdapterFactory``` provides an interface for this purpose. Two implementations exist in the SDK:

* ```lib/connection/adapter/WebSocketAdapterFactoryForBrowsers``` - For web browsers
* ```lib/connection/adapter/WebSocketAdapterFactoryForNode``` - For Node.js

By default, the ```Connection``` class assumes you're running in a web browser. However, there is nothing wrong with being explicit.

So, invoke  as follows:

	connection.connect(host, username, password, new WebSocketAdapterFactoryForBrowsers());

Or:

	connection.connect(host, username, password, new WebSocketAdapterFactoryForNode());

## Closing

Closing a connection is simple:

	connection.disconnect();

This causes the WebSocket connection to be severed. It also clears **all** existing market data subscriptions.

## Reconnecting

Unexpected network conditions may cause the WebSocket connection to close. When this happens, a *disconnect* event will be generated (see the [Monitoring](#monitoring) section). The SDK will **automatically** begin attempting to reestablish the connection.

Once the connection is reestablished, a *login success* event will be generated and subscriptions will be automatically restarted. However, market state should be considered to be outdated until you receive the first notification for each subscription.

To abort the automated reconnection process, simply call the ```Connection.disconnect``` function.