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

Browser support: Chrome and Opera implement "the Blink version". Firefox were working on it and you could try it if you enabled the right preference. But then all the browser vendors [had a meeting](https://www.w3.org/wiki/Webapps/WebComponentsApril2015Meeting) where they sat down and presented the issues they had found which hadn't surfaced in the initial version that Blink had implemented, and agreed on working on a "V1" minimally viable Shadow DOM spec, so several things that you might have already heard or read regarding Shadow DOM might have changed again. Conclusion: this spec is really unstable.

#### HTML imports

### Browser support, or "where/when/how can I use this?"

"Nothing is a standard until there's several browsers committed to it".

* Custom Elements polyfill http://webcomponents.org/polyfills/custom-elements/

### Real-life use cases of Web Components in production

### How do Web Components compare to JS frameworks? How well do they interoperate (or not)?

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
* [Minutes](http://www.w3.org/2015/07/21-webapps-minutes.html) from the Face to Face meeting 21 July 2015--many contentious bits were discussed with various browser vendors present in this meeting.

### Firefox implementation

### Developer Tools

* Docs: https://developer.mozilla.org/docs/Tools
* Wiki: https://wiki.mozilla.org/DevTools
* Feedback: https://ffdevtools.uservoice.com/ or http://mzl.la/devtools
