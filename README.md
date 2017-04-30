# `turbo_transition`

## Motivation

It's pretty cool that since, on any given Rails app, we tend to use Bootstrap (or some other responsive grid), [Rails unobtrusive Ajax helpers](http://guides.rubyonrails.org/working_with_javascript_in_rails.html#built-in-helpers), and [some touch icons/favicons](http://realfavicongenerator.net/) anyway, a user who bookmarks the site to their homescreen on a mobile device has a pretty decent "app" with no specific effort on our part.

The one thing that makes it _feel_ markedly different is, in my opinion, the abrupt transitions between pages. On iOS, for example, most navigation is dominated by the standard slide-in-from-right when navigating "in" to something, and slide-in-from-left when navigating back "out".

Other typical interactions bring up sheets or modals with the next "page" of content, or fade in the content and then fade it back out. iOS achieves this by keeping a stack of views in memory. On the stateless web, of course, we can't do that.

It would be nice, though, if we could achieve a similar _feel_ in our responsive sites, rather than jumping abruptly from page to page. In addition, we want a zero-extra-effort solution; otherwise, we might as well go down the road of building a properly done mobile-first experience.

## Turbolinks

Turbolinks is a zero-extra-effort solution for providing Ajax-like performance. Turbolinks works in part by keeping the document in place and only swapping out the `<body>`.

[From the Turbolinks docs:](https://github.com/turbolinks/turbolinks#full-list-of-events)

> ## Full List of Events
>
> Turbolinks emits events that allow you to track the navigation lifecycle and respond to page loading. Except where noted, Turbolinks fires events on the `document` object.
>
> * `turbolinks:click` fires when you click a Turbolinks-enabled link. The clicked element is the event target. Access the requested location with `event.data.url`. Cancel this event to let the click fall through to the browser as normal navigation.
>
> * `turbolinks:before-visit` fires before visiting a location, except when navigating by history. Access the requested location with `event.data.url`. Cancel this event to prevent navigation.
>
> * `turbolinks:visit` fires immediately after a visit starts.
>
> * `turbolinks:request-start` fires before Turbolinks issues a network request to fetch the page. Access the XMLHttpRequest object with `event.data.xhr`.
>
> * `turbolinks:request-end` fires after the network request completes. Access the XMLHttpRequest object with `event.data.xhr`.
>
> * `turbolinks:before-cache` fires before Turbolinks saves the current page to cache.
>
> * `turbolinks:before-render` fires before rendering the page. Access the new `<body>` element with `event.data.newBody`.
>
> * `turbolinks:render` fires after Turbolinks renders the page. This event fires twice during an application visit to a cached location: once after rendering the cached version, and again after rendering the fresh version.
>
> * `turbolinks:load` fires once after the initial page load, and again after every Turbolinks visit. Access visit timing metrics with the `event.data.timing` object.

## Hypothesis

Hypothesis: **_We can leverage Turbolinks events to create iOS-like animated transitions between pages without any extra code required._**

Here's how I would want it to work:

```html
<!-- app/views/events/index.html.erb -->

<%= link_to "Show", event, turbo_transition: "slideInRight" %>
```

```html
<!-- app/views/events/show.html.erb -->

<%= link_to "Back", events_path, turbo_transition: "slideInLeft" %>
```

Perhaps there could be an initializer to set a global default value for `turbo_transition`.

## Implementation

I imagine it could be done with [some CSS transitions](https://daneden.github.io/animate.css/) alone, and by adding/removing CSS classes while the events above are emitted; but perhaps additional JavaScript would be necessary to clone the original page's `<body>` and animate it while the next page's body is being rendered.

Additional thoughts:

 - It would be nice to allow users to define their own animations and invoke them.
 - If you Google "turbolinks transitions", you'll see that the idea is not novel, but perhaps using them in this particular way and in conjunction with `link_to` might be. And no prior implementation was developed very much, as far as I can tell; if there is a one that is useable/maintained, I'd love to just use it.

## Game Plan V1

 1. As soon as a click is initiated, capture an image of the viewport (using something like [this](https://github.com/niklasvh/html2canvas)?).
 1. Set the capture as the background of the `<html>` element.
 1. Move the `<body>` offscreen.
 1. Start animating the `<body>` in immediately, while the content is rendering.
 1. Hopefully the content finishes rendering before the animation completes. Even if not, the experience is no worse than the current state of affairs.
