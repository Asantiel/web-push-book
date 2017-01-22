---
title: Notification Behaviour
---
# Notification Behaviour

So far we've looked at the options that alter the visual appearance of a notification. There are options that alter the behaviour of notifications.

Be default, calling `showNotification()` with just visual options will have
the following behaviours:

1. Clicking on the notification does nothing.
1. Each new notification is shown one after the other. The browser will not  collapse the notifications in anyway.
1. The platform may play a sound or vibration the users devices (depending on the platform).
1. On some platforms the notification will disappear after a short
period of time while others will show the notification unless the user interacts with it (i.e. compare Android and Desktop with your notifications).

In this section we are going to look at how we can alter these default behaviours using options alone. These are relatively easy to implement and take advantage of.

### Notification Click Event

When a user clicks on a notification the default behaviour is for nothing
to happen, it doesn't even close / remove the notification.

The common practice for a notification click is for it to close and perform some other logic (i.e. open a window or make some API call to the application).

To achieve this we need to add a 'notificationclick' event listener to our service worker. This will be called when ever a notification is clicked.

<% include('../../demos/node-server/frontend/service-worker.js', 'simpleNotification') %>

As you can see in this example, the notification was clicked can be accessed via the `event.notification parameter`. From this we can via the properties on the notification, in this case we call it's `close()` method and then we are free to perform any task we wish in the background like a normal event.

> Remember: You still need to make use of event.waitUntil() to keep the service worker running while your code is busy.

### Actions

Actions allow you to give users another level of interaction with your users
over just clicking the notification.

In the previous section you saw how to define actions when calling
`showNotification()`:

<% include('../../demos/notification-examples/notification-examples.js', 'actionsNotification') %>

If / when the user clicks an action button, check the `event.action` value in the `noticationclick` event to tell if / which action button was clicked.

`event.action` will contain the `action` value set in the options. In the example about the `event.action` values would be one of the following: 'coffee-action', 'doughnut-action', 'gramophone-action' or 'atom-action'.

With this we would detect notification clicks or action clicks like so:

<% include('../../demos/notification-examples/service-worker.js', 'notificationActionClickEvent') %>

![Logs for action button clicks and notification click.](/images/notification-screenshots/action-button-click-logs.png){: .center-image }

### Tag

The *tag* option is a essentially a String ID that "groups" notifications together, providing an easy way to determine how multiple notifications are displayed to the user. This is easiest to explain with an example.

Let's display a notification and give it a tag, of
'message-group-1'. We'd display the notification with this code:

<% include('../../demos/notification-examples/notification-examples.js', 'tagNotificationOne') %>

Which will show our first notification.

![First notification with tag of message group 1.](/images/notification-screenshots/desktop/chrome-first-tag.png){: .center-image }

Let's display a second notification with a new tag of 'message-group-2', like so:

<% include('../../demos/notification-examples/notification-examples.js', 'tagNotificationTwo') %>

 This will display a second notification to the user.

![Two notifications where the second tag is message group 2.](/images/notification-screenshots/desktop/chrome-second-tag.png){: .center-image }

Now let's show a third notification but re-use the first tag of 'message-group-1'. Doing this will close the first notification and replace it with our new notification.

<% include('../../demos/notification-examples/notification-examples.js', 'tagNotificationThree') %>

Now we have 2 notifications even though `showNotification()` was 3 times.

![Two notifications where the first notification is replaced by a third notification.](/images/notification-screenshots/desktop/chrome-third-tag.png){: .center-image }

The `tag` option is simply a way of group messages like this so that any old notifications that are currently displayed will be closed if they have the same tag as a new notification.

A subtlety to using `tag` is that the browser will replace any old notification without any sound and vibration that would normally be played for a new notification.

This is where the `renotify` option comes in.

### Renotify

This largely applies to mobile devices at the time of writing. Setting this option makes new notifications vibrate and play a system sound.

There are scenarios where you might want a replacing notification to notify
the user rather than silently update. Chat applications are a good example where you would want one notification but would want to inform the user a new message has been received. In this case you could use `tag` with `renotify` set to true.

<% include('../../demos/notification-examples/notification-examples.js', 'renotifyNotification') %>

**Note:** If you set `renotify: true` on a notification without a tag, you'll get the following error:

    TypeError: Failed to execute 'showNotification' on 'ServiceWorkerRegistration': Notifications which set the renotify flag must specify a non-empty tag

### Silent

This option allows you to show a new notification but prevents the default
behavior of vibration, sound and turning on the devices display.

This is ideal if your notifications that don't require immediate attention
from the user.

<% include('../../demos/notification-examples/notification-examples.js', 'silentNotification') %>

**Note:** If you define both *silent* and *renotify*, silent will take precedence.

### Requires Interaction

Chrome on desktop will show notifications for a set time period before hiding it. Chrome on Android doesn't have this behaviour, notifications are displayed until the user interacts with it.

To force a notification to stay visible until the user has interacted with it you can define the `requireInteraction` option. This will show the notification permanently until the user dismisses / clicks your notification.

<% include('../../demos/notification-examples/notification-examples.js', 'requireInteraction') %>

Please use this option with consideration. Showing a notification and forcing the user to stop what they are doing to dismiss you notification can be frustrating.

In the next section we are going to look at some of the common patterns used on the web for managing notifications and performing actions liking open pages when a notification is clicked.
