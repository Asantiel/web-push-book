---
title: Subscribing a User
---
# Subscribing a User

The first step we need to take, is gett permission for the user to send
them push messages. Once we have permission, we can get our hands on a `PushSubscription`.

The JavaScript API to do this is pretty straight forward, so let's go through the flow.

## Feature Detection

The first we need to do is check if the current browser supports push messaging or not. We can check push is support with two simple checks.

1. Check the *serviceWorker* API on the *navigator*.
1. Check *PushManager* on  the *window*.

<% include('../../demos/node-server/frontend/app.js', 'feature-detect') %>

While browser support is growing quickly for both service worker and
push messaging support, it's always a good idea to feature detect for both and
[progressively enhance](https://en.wikipedia.org/wiki/Progressive_enhancement).

## Register a Service Worker

With the feature detect we knowing that service workers and Push are supported. The next step is to "register" our service worker.

When we register a service worker, we are telling the browser what file to use for push events. The file is still just JavaScript, but the browser will "give it access" to the service worker APIs. To be more exact, the browser runs the file in a service worker context.

To register a service worker, call `navigator.serviceWorker.register()`, passing in the path to our file. Like so:

<% include('../../demos/node-server/frontend/app.js', 'register-sw') %>

This code above tells the browser that we have a service worker file and where its located. In this case, the service worker file is at `/service-worker.js`. Behind the scenes the browser will take the following steps after calling `register()`:

1. Download the service worker file.

1. Run the JavaScript.

1. If everything ran correctly and there were no error, the promise returned by `register()` will resolve. If there are errors of
any kind, the promise will reject.

> If `register()` does reject, double check your JavaScript for typos / errors in Chrome DevTools.

When `register()` does resolve, it returns a `ServiceWorkerRegistration`. With this registration, we have access to the `PushManager` API.

## Requesting Permission

The next step to take is to get permission from the user to show notifications.

The API for getting permission is relatively simple, the only downside is that
the API [recently changed from taking a callback to returning a Promise](
    https://developer.mozilla.org/en-US/docs/Web/API/Notification/requestPermission).

The problem with this, is that we can't tell what version of the API will be supported by the current browser, so you have to implement both and handle both.

<% include('../../demos/node-server/frontend/app.js', 'request-permission') %>

In the above code, the important snippet of code is the call to `Notification.requestPermission()`. This method will display a prompt to the user:

![Permission Prompt on Desktop and Mobile Chrome.](/images/permission-prompt.png)

Once the permission has been accepted / allowed, closed (i.e. clicking the cross on the
pop-up) or blocked, we'll be given the result as a string: 'granted',
'default' or 'denied'.

In the sample above, the promise only resolves if the permission is granted,
otherwise we throw an error making the promise reject.

If the user clicks the 'Block' button, your web app will not be able to ask the user again, unless they "unblock" you app by manually changing the permissions in a buried settings panel. Think carefully about how and when you ask the user for permission. The good news is that most users are happy to give permission as
long as it's asked in a way that they *know* why the permission is being asked.

We'll look at how some popular sites ask for permission later on.

## Subscribe a User with PushManager

Once the permission is granted and we have our service worker registration we can subscribe a user by calling `registration.pushManager.subscribe()`.

<% include('../../demos/node-server/frontend/app.js', 'subscribe-user') %>

When calling the `subscribe()` method, we pass in an *options* object, which has required and optional parameters.

Lets look at the options we can pass in.

### userVisibleOnly

When push was first added to browsers, there was uncertainty about whether
developers should be able to send a push message and not show a notification. This is commonly referred to as silent push, due to the user not knowing anything happened.

The concern was that developers could do nasty things like track a user's
location on an ongoing basis without the user knowing.

To avoid this scenario and to give spec authors time to consider what to do
with this matter, the `userVisibleOnly` option was added and passing in
a value of **true** is making an agreement with the browser that we'll show
a notification every time a push is received (i.e. no silent push).

At the moment you can't pass in anything else. If you don't include the
`userVisibleOnly` key you'll get the following error:

> Failed to subscribe the user. DOMException: Registration failed -
> missing applicationServerKey, and manifest empty or missing

If you pass in a value of **false**, i.e. you want the ability to send silent
pushes, you'll get this error in Chrome:

> Chrome currently only supports the Push API for subscriptions that
> will result in user-visible messages. You can indicate this by
> calling pushManager.subscribe({userVisibleOnly: true}) instead.
> See [https://goo.gl/yqv4Q4](https://goo.gl/yqv4Q4) for more details.

It's currently looking like blanket silent push will never be implemented,
but instead some kind of budget API will be used to give web apps a certain
number of silent push messages and the budget for these will change over time
based on the use of a web app.

### aplicationServerKey Option

We briefly mentioned that application server keys are used by a push service
to know what application is subscribing a user and ensuring that the same
application is messaging that user, but we didn't explain how that's done.

Application server keys are a public and private key pair that are unique to your application. The private key should be kept a secret to your application and the public key can be shared with push services.

The `applicationServerKey` parameter we pass into the `subscribe()` options should be the public key. With this the browser can pass it to a push service before generating a `PushSubscription` endpoint.

The diagram below illustrates these steps.

1. You web app is loaded in a browser and you call `subscribe()` passing in your public application server key.
1. The browser then makes a network request to a push service who will generate an endpoint, associate this endpoint with that public key and return the
endpoint to the browser.
1. The browser will add this endpoint to the `PushSubscription` which is returned via the `subscribe()` promise.

![Illustration of the public application server key is used in subscribe method.](/images/svgs/application-server-key-subscribe.svg)

When you later want to send a push message to a *PushSubscription*, you'll
need to create an **Authorization** header which will contain information
signed with your application server's **private key**. When the push service
receives a request to send a push message, it can decrypt this signed **Authorization** header by looking up the public key linked to the endpoint receiving the request. If the decryption step worked and some additional check pass, the push service knows that it must have come from the application server with the **matching private key**. It's basically a security measure that prevents anyone else sending messages to your users.

![Illustration of how the private application server key is used when sending a message.](/images/svgs/application-server-key-send.svg)

Technically, the `applicationServerKey` is optional. However, the easiest
implementation on Chrome requires it and other browsers may require it in
the future. It's optional on Firefox.

The specification that defines *what* the application server key should be is
the [VAPID spec](https://tools.ietf.org/html/draft-thomson-webpush-vapid).
Whenever you read something referring to *"application server keys"* or
*"VAPID keys"*, just remember that they are the same thing.

That's all of the subscribe options, but there is one last side effect of calling `subscribe()`. If your web app doesn't have permissions for notifications at the time of calling `subscribe()`, the browser will request the permissions for you. This is useful if your UI works with this flow, but if you want more control (and I think most developers will), stick to the
`Notification.requestPermission()` API.

## What is a PushSubscription?

We've called `subscribe()`, passed in some options and in return we get a promise that resolves to a `PushSubscription`.

<% include('../../demos/node-server/frontend/app.js', 'subscribe-user') %>

This `PushSubscription` object contains all the required information needed to
send a push messages to that user. If you print
out the `JSON.stringify()` version of the subscription object, we'd see
the following:

```json
{
  "endpoint": "https://some.pushservice.com/something-unique",
  "keys": {
    "p256dh": "BIPUL12DLfytvTajnryr2PRdAgXS3HGKiLqndGcJGabyhHheJYlNGCeXl1dn18gSJ1WAkAPIxr4gK0_dQds4yiI=",
    "auth":"FPssNDTKnInHVndSTdbKFw=="
  }
}
```

The **endpoint** is the push services URL, you call this URL to trigger a push
message.

The **keys** object contains the values used to encrypt data that you'll send
with your push message (which we'll discuss later on in this book).

## Send a Subscription to Your Server

Once you have a subscription you'll want to send it to your server. It's up to you how you do that but a tiny tip is to use `JSON.strinigify()` to get all the necessary data out of the subscription object, otherwise you can piece together the same result manually:

```javascript
const subscriptionObject = {
  endpoint: subscription.endpoint,
  keys: {
    p256dh: subscription.getKeys('p256dh'),
    auth: subscription.getKeys('auth')
  }
};

// The above is the same output as:

const subscriptionObjectToo = JSON.stringify(subscription);
```

In the demo used throughout this book, we make a POST request to send a
subscription to our node server that stores the subscription in a database.

Sending the subscription is done like so:

<% include('../../demos/node-server/frontend/app.js', 'send-subscription-to-server') %>

With the `PushSubscription` details on our server we are good to send our user
a message whenever we want.

## FAQs

A few common questions people have asked at this point:

> Can I change the push service a browser uses?

No. The push service is selected by the browser and as we saw with the
`subscribe()` call, the browser will make network requests to the push service
to retrieve details used to make a *PushSubscription*.

> Each browser uses a different Push Service, don't they have different API's?

All push services will have the same API available for you to use.

This common API is called the `Web Push Protocol` and describes the network
request your server will need to make to trigger a push message.

> If I subscribe a user on their desktop, are they subscribed on their phone
as well?

Unfortunately not. A user must register for push on each browser they wish to
receive messages from. It's also worth noting that this will require
the user granting permission on each device as well.
