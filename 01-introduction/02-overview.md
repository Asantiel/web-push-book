# Overview of Push

Before getting into the API, it'll be useful to have an idea of how everything
works, from start to finish, at a high level so as we step through each
topic you'll be familiar with how it fits in to achieve the end goal of
support push messages.

The three key steps when implementing push notifications are:

1. The client side logic to subscribe a user to push (i.e. JavaScript and UI
  in your web site / web app)
1. The logic / API calls your back end will need to trigger a push
  notification
1. The service worker code that will be triggered in the background when a push
  message is recived on a users device.

## Client Side

When you want to subscribe a user to push messages, the main thing you'll want
to do is ask the user for permission to send them push messages and
then subscribe them, resulting in a `PushSubscription` object, which
you'll need to send and store on your servers.

A `PushSubscription` contains all the information you need to know to send that
device a push message.

![Once a user is subscribed, the browser will give you a PushSubscription which you need to send and store on your server](build/images/browser-to-server.png)

## Server Side

A `PushSubscription` will have the following structure:

    {
      endpoint: 'https://my-push-service.com/some-kind-of-unique-id-1234',
      keys: {
        "p256dh" : "BNcRdreALRFXTkOOUHK1EtK2wtaz5Ry4YfYCA_0QTpQtUbVlUls0VJXg7A8u-Ts1XbjhazAkj7I99e8QcYP7DkM=",
        "auth"   : "tBHItJI5svbpez7KI4CCXg=="
      }
    }

The `endpoint` is the URL of the **push service**, which is essentially the
service the will take requests to send push messages and deliver them to the
appropriate device.

`keys` contains the base64 url encoded strings needed to encrypt a payload in
a push message. When I say payload, I am referring to the little bit of data
that you can send with a push message (we'll learn more about that later).

When you want to send a push message, you'll take a `PushSubscription` and
make an API call to it's `endpoint`, giving the request a specific format
which is known as the **Web Push Protocol**. It basically defines what
headers and format the body of the request should have.

![When your server wishes to send a push message, it makes a web push protocol request to a push service](build/images/server-to-push-service.png)

## Service Worker Code

When a push message is received by a browser, the browser dispatches an event
to a JavaScript file known as a Service Worker, which we'll cover in the next
section.

It's inside the `push` event that you'll need to show a notification
to the user.

// TODO: Diagram of Browser -> Notification
