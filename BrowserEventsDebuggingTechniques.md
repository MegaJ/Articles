# Debugging browser events, 2016

Here are some things I've learned from taking on maintenance cases for a web application whose logic has been written and updated by many people for decades and is too large to understand completely.

When I say javascript in this article, I mean unminified javascript.

## My event happens to a future element

You can try hooking up an event listener at the html/document level. You can then see if the event's target is what you expect, or at least see the highest parent reached. I think jQuery under the hood implements listeners for future elements doing something like this:

```javascript
// make sure $0 is the top level element, or body
$0.addEventListener('focus', function(evt) {
  var modal = document.querySelector('#importantModal');
  if (!modal) { return; }
  if (!modal.contains(evt.target)) { return; }

  console.log('New focus!: ', evt);
 
}, true);
```

Here you see if an element exits, and if so, see if the target is nested beneath it (or is the element itself). I set the `capture` to `true` because focus events do not bubble, so catch them in the capturing event phase. You can't miss an event this way, since events start at the highest DOM element and go down, then after reaching the target, bubble up if the event bubbles. That is, if the event is fired at all.

While working in an unfamiliar code base, there are times where someone uses event.stopPropagation() or .stopImmediatePropagation() to stop your expected event early. Use the developer tools to list out the event listeners and then inspect those. If that's too much work, just use your dev tools' search functionality on the source code, look for `stopPropagation` or `stopImmediatePropagation`.

## Which event listener is being called?

Sometimes events are handled by the target's ancestor. This is where you either repeat your actions to trigger the event or programmatically fire your own DOM events. You can catch which listeners are handling the events by setting break points at different event listeners on the page.

## Programmatically fire your own DOM events

Check the docs: https://developer.mozilla.org/en-US/docs/Web/API/Event/Event#Browser_compatibility

It's faster. You can write the code once and use it from the cached commands in your debugger console. You can do some cursory unit test your event listeners. You can combine this with other techniques to see if your event will ever reach your intended DOM target.

## What if some javascript is starting a popup window, and I want to debug the js when it's loading?

I've had situations where the normal, open popup window --> open dev tools --> set breakpoints --> refresh page, technique doesn't work. The javascript file gets refreshed and the break points don't belong to the it.

Here you may not be able to open your developer tools in time. You can try grabbing the URL of the popup window and open that in a new tab in the browser and then open developer tools. Then refresh with your devtools still open. I have also had success with saving the webpage of the popup window into a file, then load it from hard drive into my browser. I think Chrome has some functionality so that you don't have to do this anymore.

You can also try live editing the js, then reloading the page without triggering complete redownloads of the js file:

If you use Chrome you have live editing capabilities. As of 28/01/16 I'm still not aware of any live editing capabilities in FireFox.
   * Modify the js files you want with `debugger;` statements or set breakpoints
   * And then refresh without wiping out your changes. Try this in the debugger console: `window.location = window.location`

This SO post has more details: http://stackoverflow.com/questions/15897434/javascript-refresh-parent-page-without-entirely-reloading.

## Pretty much everything for debugging is already available in your browser

For more event monitoring, `monitorEvents` is supported in Chrome, maybe other webkit browsers too. It takes a second argument actually, allowing you to filter out which events you want to inspect: `monitorEvents($0, ['focus'])`will give you all focus/blur events on whatever element you currently have as $0. Read all of it here: https://developers.google.com/web/tools/chrome-devtools/console/command-line-reference#monitoreventsobject-events

I also just discovered `debug(myEventListener)` which will breakpoint the debugger whenever the passed in function is invoked.

As of 15/10/2016, for FireFox, it seems that the EventBug plugin gives you this functionality. Checkout this SO: http://stackoverflow.com/questions/11097234/using-firefox-how-can-i-monitor-all-javascript-events-that-are-fired

In modern browsers you can also list out all event listeners attached to all DOM elements. Check the docs.

## But my events are Angular!

Angular events are not native browser events. I haven't looked into this deeply but "event" handling is probably done by a function that buffers a bunch of callbacks and fires them off in a loop. This is an area I'm not good at :/. Still, it's hard to go wrong with `debug(myEventListener)` in Chrome and then looking at the call stack. Or using `debug;` in live coding or in rebuilding your webapp during development.