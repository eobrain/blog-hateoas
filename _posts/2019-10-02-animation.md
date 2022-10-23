---
layout: post
title:  Adding Swipe Animation
date:   2019-10-02
categories: Programming
---

In a follow-up to the last article which described [Adding swipe actions to a web site][1],
I'll show how to add simple animation to give some user feedback that the swipe
has happened.

First of all, we'll modify the JavaScript to add a single line as follows:

```js
mc.on('swipe', (ev) => {
  const dir = (ev.deltaX < 0) ? 'next' : 'prev'
  const a = document.getElementById(dir)
  if (a) {

    // Added line for animation:
    document.body.classList.add('animate-' + dir)

    window.location = a.href
  }
})
```

So, just before loading in the new page this code adds a CSS class
`animate-next` or `animate-prev` to the body element of the page (i.e. to the
entire page). This will trigger an animation that will run while waiting for the
new page to load.

To actually cause the animation we add the following CSS:

```css
  @keyframes slideLeft {
    to {
      transform: translateX(100%);
      opacity: 0;
    }
  }
  @keyframes slideRight {
    to {
      transform: translateX(-100%);
      opacity: 0;
    }
  }
  .animate-prev {
    animation: 0.2s ease-out 0s 1 slideLeft;
  }
  .animate-next {
    animation: 0.2s ease-out 0s 1 slideRight;
  }
```

Let's walk through this:

1. Each `@keyframes` includes a single `to` (100%) key-frame which is the state
   to animate to. Because there is no `from` (0%) key-frame the animation will
   default to using the current state as the starting state. It
   * translates 100% to the left or right (which for the body will translate it offscreen)
   * fades from opaque to transparent.
2. Each class selector has an `animation` that uses the corresponding key-frame,
   specifying that it should animate
   * over the given time
   * using `ease-out` (start fast then slow down)
   * with no delay
   * playing just once
   * the given `#keyframes` name

Note we are careful to only use [High Performance Animations][2] that can be
efficiently implemented by a phone's GPU, namely translation, scaling, rotation,
or opacity change. We could obviously use these four types of animations to
further enhance the page-swiping animation to make it more effective, while
still keeping it efficient.


[1]: {% post_url 2019-09-30-swipe %}
[2]: https://www.html5rocks.com/en/tutorials/speed/high-performance-animations/