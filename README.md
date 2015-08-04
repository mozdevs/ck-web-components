# Web Components

> Hopefully current as of August 2015 (but don't 100% trust it, these specs are changing pretty much every week)

## Introduction

* The Web today is much more than documents. Web 2.0 kickstarted a new era in complex apps and mash-ups that we hadn't thought of before, yet we're still building complex UIs with just rudimentary browser support and aggregating code that comes from many different vendors and origins. This can create a number of issues: side effects, CSS bleeding, bloated code and really bad mark-up.
* Web Components are a series of new browser features to help developers build reusable and modular front-end code that circumvents those issues.

## Key Points

Instead of creating entire new JavaScript frameworks, Web Components propose to solve the encapsulation issue by extending existing elements or building new elements on top of them, and keep on using the DOM and other browser features for interoperability with existing code as it's framework agnostic.

1. ???

## Presentation Setup and Materials

For __any presentation__:

- [ ] Firefox (Stable, Developer Edition or Nightly--all are fine, but newest releases are better as support in DevTools will be improved e.g. inspecting CSS pseudos or element event handlers) installed on the host computer.


## Demoing / Script

### Terrible Mark-Up

```html
<div class="widget calendar ui theme-winter">
	<div class="component-wrapper">
    	<div class="inner-content">
        	(ad nauseam)
        </div>
	</div>
</div>
```

```javascript
$("#datepicker").datepicker($.datepicker.regional["fr"]);
$("#locale").change(function() {
	$("#datepicker").datepicker("option", $.datepicker.regional[ $(this).val() ] );
});
```

### What if the browser helped us write better code? What if we could teach new elements to the browser?

```html
<x-calendar></x-calendar>
```

```javascript
var calendar = document.createElement('x-calendar');

// or...
var calendar = document.querySelector('x-calendar');
```

```javascript
calendar.nextMonth();
calendar.setLocale('fr');
```

This is shorter, cleaner, and easier to understand and maintain.

### How do we get there?

By using Web Components! But as mentioned before, Web Components is not just one new big monolithic API, but a series of different features and APIs addressing different aspects:

* Custom Elements
* HTML Templates
* Shadow DOM
* HTML imports

Let's go through each of the features and see how it works and what does it address.

#### Custom Elements

[Custom Elements](http://w3c.github.io/webcomponents/spec/custom/) let you teach new elements to the browser via the `registerElement` method. For example:

```javascript
document.registerElement('web-bell', WebBellPrototype);
```

Here `web-bell` is the tag name (just like `a`, `h1`, `table` etc are tag names) and `WebBellPrototype` defines the behaviour of instances of this custom element.

All custom elements tag names must include a hyphen. This is so that authors don't attempt to register elements that might already exist in the browser, either today or in the future. You can also use this as a way to create namespaced elements (e.g. `mozilla-calendar`).

A custom element prototype extends at least `HTMLElement.prototype`--this is what makes it possible to insert custom element instances into the DOM tree. It might also implement certain life cycle callbacks, and its own methods, getters and setters. Following on the example:

```javascript
var WebBellPrototype = Object.create(HTMLElement.prototype);

WebBellPrototype.createdCallback = function() {
	this.innerHTML = '🔔';
};
```

This is a very simple custom element whose only purpose is to render an emoji bell. So when the `createdCallback` function is called (sometime right after creating the instance), we set the contents of the element to said emoji bell.

There are [four types](http://w3c.github.io/webcomponents/spec/custom/#types-of-callbacks) of callbacks:

* `created`
* `attached`
* `detached`
* `attributeChanged`

Only `created` will be executed once.

Once the element has been registered, we can create instances as we would with any other standard element:

```javascript
// Programmatically
var bell = document.createElement('web-bell');
```

```html
// Declaratively
<web-bell></web-bell>
```

And we can also style them as if they were any other element:

```css
web-bell {
	border: 3px solid green;
}
```

You can also extend elements other than the `HTMLElement`. For example, you can extend buttons by extending from the `HTMLButtonElement` property:

```javascript
var DingButtonProto = Object.create(HTMLButtonElement.prototype);

DingButtonProto.createdCallback = function() {
	this.innerHTML = 'ding!';
};

document.registerElement('ding-button', {
	prototype: DingButtonProto,
	extends: 'button'
});
```

These elements have to be instantiated slightly differently:

```javascript
var ding = document.createElement(
	'button', 'ding-button'
);
```

Or

```html
<button is="ding-button"></button>
```

This is amazing for **accessibility** and **progressive enhancement**: by extending existing elements we already have access to a lot of built-in browser behaviour and we don't need to (badly) reimplement it ourselves, which is the source for a lot of accessibility issues. Also, if the browser doesn't support the `is=""` feature, users should at least get basic functionality as the browser would just ignore the `is=` attribute.

But apparently implementing this type of extensibility is complicated to implement in browsers and so it is not clear if this will be supported in the future.

Custom elements are supported in: Firefox, Chrome and Opera.

#### HTML Templates

HTML Templates are inert HTML chunks and they are not live "until you say so". The browser just essentially ignores them until you tell it otherwise. It is only then that it will parse that code.

The way to define a template is by using the `<template>` element. For example, suppose we have a somewhat complicated piece of mark-up to create table rows, and we don't want to create each node manually with `document.createElement`:

```html
<template id="row-template">
	<tr>
	    <td><input .../></td>
        <td><button .../></td>
	</tr>
</template>
```

Then we could create instances of rows as easily as this:

```javascript
var rowTemplate = document.getElementById('row-template');
var table = document.getElementById('form-table');
table.appendChild(rowTemplate.content.cloneNode());
```

HTML Templates are very simple. There is no two way binding, or string interpolation. They just "do what it says on the tin".

Support is pretty good: Firefox, Chrome, Opera and Safari and [in development](https://status.modern.ie/templateelement) in Edge.

#### Shadow DOM

The <a href="http://w3c.github.io/webcomponents/spec/shadow/">Shadow DOM</a> lets you have multiple DOM trees inside a hierarchy and have them interact with each other, with the main goal being having better composition.

It is also quite complicated to understand, so it has sometimes been informally defined as "now you see it, now you don’t", as it lets you replace the *normal* HTML tree in the normal hierarchy (the *light DOM*) with another HTML tree which is not visible from the outside (the *shadow DOM*).

To do this, the browser creates a boundary around your element inside which you can place mark-up that *may* stay hidden to external elements, and you also have an option to reset and isolate CSS styles.

This is *superuseful* for things such as players or calendars as there won't be inherited styles that you have to reset, etc, but it is still executed in the same document context; Shadow DOMs are not iframes and so they are not inherently more secure, JavaScript wise.

A simple Shadow DOM example:

```javascript
node.innerHTML = 'This is the light DOM';
var shadow = node.createShadowRoot({ mode: 'closed' });
shadowRoot.innerHTML = 'The Shadow DOM has taken over';
```

If we inspect this with DevTools, we will see that there are two "elements" in the content of the node. The "light DOM" that we initially set, and the "shadow DOM".

When there is a shadow DOM present, the browser will display that instead of the light DOM. But scripts trying to access the contents of the node will only see the default light DOM. Furthermore, because we defined the shadow root as `closed` at creation time, if a script tries to access the `node.shadowRoot` property they will just get `null`--showing the boundary in action.

Browser support: Chrome and Opera implement "the Blink version". Firefox were working on it and you could try it if you enabled the right preference. But then all the browser vendors [had a meeting](https://www.w3.org/wiki/Webapps/WebComponentsApril2015Meeting) where they sat down and presented the issues they had found which hadn't surfaced in the initial version that Blink had implemented, and agreed on working on a "V1" minimally viable Shadow DOM spec, so several things that you might have already heard or read regarding Shadow DOM might have changed again.

Conclusion: this specification is really unstable. We, as browser vendors, encourage you to play with it and help us find edge and use cases, but you should be aware that things are going to change any time soon.

#### HTML imports

HTML imports allow you to include HTML documents from an HTML document. This lets you load external content into your document in a declarative way, and so some people call them *`require()` for the web*.

Suppose you had this line in your document HTML:

```html
<link rel="import" href="my-component.html">
```

And then in `my-component.html`:

```html
<script src="my-component.js"></script>
<link rel="stylesheet" href="my-component.css">
```

So `my-component.js` and `my-component.css` get parsed and are made available to the document that imported them via `my-component.html`. You could also have more imports inside `my-component.html`, and so on.

This can cause a situation in which you need to wait for lots of network requests to finish loading before you can use a given import, so folks from Polymer made a utility called [Vulcanize](https://github.com/Polymer/vulcanize) that follows through all the import links in an import and generates just two files: HTML (with inlined JavaScript) and CSS.

Support for HTML imports isn't too good: only Chrome and Opera support it. Firefox had an unfinished implementation but it will be removed, as Mozilla felt that the issues imports create far outweigh the advantages they provide, and also they want to see if it's possible to accomplish the same effect using ES6 modules--or maybe even if they are actually required at all. With the latest developments in the Fetch API and Service Workers we have several ways to get content into the browser and we'd prefer to make sure these are robust.

### Browser support recap, or "where/when/how can I use this?"

* Chrome, Opera: everything
* Firefox: Behind a flag, everything except HTML imports.
* Safari: Templates.
* Edge: they're going to implement Templates and Shadow DOM

The thing you need to remember here is: "nothing is a standard until there's several browsers committed to it". So far the API you can use most safely is HTML templates, but there's only a few things you can do with it. This doesn't sound super exciting, does it?

### Polyfills

The good news is that because this is *JavaScriptLandia*, **there are polyfills** that we can use to get a feeling of what a Web Components-powered future would be, and play and experiment with it. You might realise you have a particular use case that the people writing the specs didn't realise was possible, or the way you want your components to interact with other pieces of code might not be ideal, so you should give feedback to browser vendors / spec authors. It is also a fantastic way to get involved with the Web!

* [webcomponents.js](https://github.com/WebComponents/webcomponentsjs) is the biggest. It polyfills custom elements, HTML imports, Shadow DOM and also <tt>WeakMap</tt> and Mutation Observers.
* [webcomponents-lite.js](https://github.com/webcomponents/webcomponents-lite) is almost like the above, but it doesn't polyfill Shadow DOM.

A note of warning, though: **polyfills are not free*. They come at a cost (bandwidth and processing) which is not trivial--specially when on mobile. Also, the Shadow DOM polyfill is a huge beast, and you might run into issues with Shadow DOM scoped selectors. That's why you can choose to not to use the Shadow DOM features via polyfills. Additionally there might be potential inconsistencies with the HTML imports polyfill: the way import requests block or not and their timing might be different between a native implementation and a polyfilled one. You can get weird bugs.

### Should you use a web components framework? What about <a href="https://www.polymer-project.org/">Polymer</a>? <a href="http://x-tags.org/">X-Tag</a>? <a href="http://bosonic.github.io/">Bosonic</a>?

These are *syntactic sugar to make vanilla web components less sour*. They are built on top of the Web Components pillars, or on the polyfills, but they are *not* shipped with the browser.

Their advantage versus using raw Web Components code or vanilla JavaScript to implement them is that if something changes in the native browser layer, you just need to update the frameworks to a newer version--not *your code*. But they're not conceptually that different from building "vanilla" Web Components, and they're not that different between them either.

For example, here's how we would define a custom element with X-Tag:

```javascript
xtag.register('web-bell', {
  extends: 'div',
  lifecycle: {
	created: function() {
	  this.innerHTML = '🔔';
	}
  }
});
```

And here's how you would do the same using Polymer:

```javascript
Polymer('web-bell', {
  extends: 'div',
  created: function() {
	this.innerHTML = '🔔'
  }
});
```

So yes--they *are* nice. The only issue is that in order to use them, you also need to load the framework they were built on. So in the future, when Web Components are standard, a vanilla-based component will have less dependencies than a framework-based component.

### How do Web Components compare to JS frameworks? How well do they interoperate (or not)?

### Real-life use cases of Web Components in production

## Demoing: Things that are Broken

- [ ] Firefox: some features do not work well if `dom.webcomponents.enabled` is enabled in `about:config`.

## What's next for...

### Web Components



### DevTools

* Shadow DOM inspection
* ...

## Reference Materials

### Specifications and discussions

* [Custom Elements](http://w3c.github.io/webcomponents/spec/custom/) specification
* [Custom Elements documentation](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Custom_Elements)
* [HTML Templates documentation](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template).
* [Shadow DOM](http://w3c.github.io/webcomponents/spec/shadow/) specification.
* [The State of Web Components](https://hacks.mozilla.org/2015/06/the-state-of-web-components/) - a blog post discussing the state of Web Components, why we are where we are, and Mozilla's position on each API.
* [Minutes](http://www.w3.org/2015/07/21-webapps-minutes.html) from the Face to Face meeting 21 July 2015--many contentious bits were discussed with various browser vendors present in this meeting.
* [Update on standardizing Shadow DOM and Custom Elements](https://annevankesteren.nl/2015/07/shadow-dom-custom-elements-update) - a post describing the pain points on Custom Elements that are delaying its implementation across browsers.
* [Are we componentized yet?](http://jonrimmer.github.io/are-we-componentized-yet/) - tracking support for each API/feature on each browser.

### Firefox implementation

### Developer Tools

* Docs: https://developer.mozilla.org/docs/Tools
* Wiki: https://wiki.mozilla.org/DevTools
* Feedback: https://ffdevtools.uservoice.com/ or http://mzl.la/devtools
