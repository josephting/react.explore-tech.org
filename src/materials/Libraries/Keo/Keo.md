---
path: '/materials/Keo'
type: 'GitHub'
img: './screenshot.png'
material:
  title: 'Keo'
  url: 'http://keo-app.herokuapp.com/'
  github_url: 'https://github.com/Wildhoney/Keo'
  subscribers_count: '5'
  stargazers_count: '224'
  tags: ['']
  subtitle: 'Plain functions for a more functional Deku approach to creating stateless React components, with functional goodies such as compose, memoize, etc... for free.'
  clone_url: 'https://github.com/Wildhoney/Keo.git'
  ssh_url: 'git@github.com:Wildhoney/Keo.git'
  pushed_at: '2017-08-08T08:11:54Z'
  updated_at: '2019-02-01T15:33:07Z'
  author:
    name: 'Wildhoney'
    avatar: 'https://avatars3.githubusercontent.com/u/1528477?v=4'
    github_url: 'https://github.com/Wildhoney'
  latestRelease:
    tag_name: null
    name: null
    url: null
    created_at: null
---
<img src='media/logo.png' alt='Keo' width='250' />

> <sub><sup>*['Keo'](https://vi.wikipedia.org/wiki/Keo) is the Vietnamese translation for glue.*</sup></sub><br />
> Plain functions for a more functional [Deku](https://github.com/dekujs/deku) approach to creating stateless React components, with functional goodies such as compose, memoize, etc... for free.

![Travis](http://img.shields.io/travis/Wildhoney/Keo.svg?style=flat-square)
&nbsp;
![npm](http://img.shields.io/npm/v/keo.svg?style=flat-square)
&nbsp;
![License MIT](http://img.shields.io/badge/License-MIT-lightgrey.svg?style=flat-square)

* **npm:** `npm install keo --save`

<img src='media/screenshot.png' />

---

## Table of Contents

* [Advantages](#advantages)
* [Getting Started](#getting-started)
* [Destructuring](#destructuring)
* [Shadow DOM](docs/SHADOW_DOM.md)
* [Nonstandard Properties](#nonstandard-properties)
* [Testing Smart Components](#testing-smart-components)

At the core of Keo's philosophies is the notion that you **shouldn't** have to deal with the `this` keyword &mdash; and while in ES2015 the `this` keyword has become easier to manage, it seems wholly unnecessary in a React component. As such, Keo takes a more [Deku](https://github.com/dekujs/deku) approach in that items such as `props`, `context`, `nextProps`, etc... are passed in to [*some*](#lifecycle-functions) React [lifecycle functions](https://facebook.github.io/react/docs/component-specs.html).

Since `v4.x`, Keo has taken on a more fundamental interpretation of React where components are **expected** to be passed immutable properties &mdash; and `state` is entirely inaccessible, as is `setState` to prevent components from holding their own state. As such, you are **required** to use Redux with Keo to pass properties down through your components.

> **Note:** Prior to `v4.x` Keo had a different API which was more tolerant &mdash; please use `npm i keo@3.0.2` &mdash; [See associated README](LEGACY.md)

## Advantages

* Steer away from `class` sugaring, inheritance, and `super` calls;
* Create referentially transparent, pure functions without `this`;
* Gain `memoize`, `compose`, et cetera... for gratis with previous;
* Use `export` to export plain functions for simpler unit-testing;
* Simple composing of functions for [*mixin* support](https://github.com/dekujs/deku/issues/174);
* Avoid functions being littered with React specific method calls;
* Integrated `shouldComponentUpdate` performing immutable equality checks from `propTypes`;
* An assumption that [immutable properties](http://www.sitepoint.com/immutability-javascript/) are used for performance gains;
* Use `render` composition to enable [Shadow DOM](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/) support in React;

## Getting Started

Use [Redux](https://github.com/reactjs/redux) to pass down properties through your components, and an immutable solution &mdash; such as [`seamless-immutable`](https://github.com/rtfeldman/seamless-immutable) or Facebook's [`Immutable`](https://facebook.github.io/immutable-js/) &mdash; even [`Object.freeze`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze) can in many cases be perfectly acceptable for getting started.

Once you're setup with Redux, and your project is passing down immutable properties, within your first component you can import `stitch` from Keo. In the following example we'll assume the immutable property `name` is being passed down to your component:

```javascript
import React from 'react';
import { stitch } from 'keo';

const render = ({ props }) => {
    return <h1>{props.name}</h1>
};

export stitch({ render });
```

In the above example the component will re-render **every time** properties are updated in your Redux state &mdash; even when the `name` property hasn't been changed. React provides the [`PureRenderMixin` mixin](https://facebook.github.io/react/docs/pure-render-mixin.html) for these instances, and Keo provides a similar solution based on `propTypes`.

Taking advantage of the `shouldComponentUpdate` improvement means you **must** define your `propTypes` &mdash; Keo favours this approach over checking `props` directly to encourage strictness in component definitions. It's also important to remember that you should enumerate props that are passed to your child components &mdash; see React's documentation [Advanced Performance](https://facebook.github.io/react/docs/advanced-performance.html).

```javascript
import React, { PropTypes } from 'react';
import { stitch } from 'keo';

const propTypes = {
    name: PropTypes.string.isRequired
};

const render = ({ props }) => {
    return <h1>{props.name}</h1>
};

export stitch({ propTypes, render });
```

With the above component definition **only** when the `name` property has changed will the component re-render &mdash; in many cases this provides a [huge performance gain](https://facebook.github.io/react/docs/advanced-performance.html). It's important to benchmark your React applications using tools such as [`react-addons-perf`](https://facebook.github.io/react/docs/perf.html) &mdash; and in particular the `printWasted` function which will demonstrate the benefit of using `shouldComponentUpdate`.

## Destructuring

In keeping with one of Keo's philosophies that the `this` keyword should be avoided &ndash; Keo provides a way to destructure required arguments from within your components:

```javascript
const componentDidMount = ({ props }) => {
    dispatch(fetch(`/user/${props.user.id}`));
};
```

Properties which can be destructured are as follows:

* `props` which are passed down via Redux;
* `dispatch` which is an alias for `props.dispatch`;
* `context` allowing access to such modules as `router`;

Properties which are typically available in React components, but are unavailable in Keo components:

* `state` and `setState` as stateless components are forbidden to maintain local state;
* `refs` use `event.target` on events instead;
* `forceUpdate` as components are only updated via `props`;

## Lifecycle Functions

The entire gamut of [React's lifecycle methods](https://facebook.github.io/react/docs/component-specs.html) pass in their own associated arguments &mdash; for example the `render` method will take `props`, `context` and `dispatch`, whereas other functions such as `componentWillUpdate` would also take an additional `nextProps` argument.

## Nonstandard Properties

Below are a handful of additional nonstandard properties which can be destructured in **all** lifecycle methods.

* [`id`](#id) &mdash; for managing local state in the Redux tree structure;
* [`args`](#args) &mdash; accessing **all** arguments for passing to other functions;

### `id`

For managing [pseudo-local state](https://github.com/reactjs/redux/issues/159) in a single tree state you can use the `id` property &mdash; which is a unique [`Symbol`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol) representing the current component. When dispatching actions you should pass the `id` as the payload, and then pass the `id` back as part of the result &mdash; with that information it's simple to determine when a component *should* be updated.

```javascript
const render = ({ id }) => {
    return <a onClick={dispatch(setValueFor(id, 'United Kingdom'))}></a>;
};
```

You may also prevent other components from updating by using the `shouldComponentUpdate` function to determine when the action applies to the current component. It's worth noting that a custom `shouldComponentUpdate` will simply be composed with the Keo default `shouldComponentUpdate` which inspects the `propTypes` for a significant performance enhancement.

```javascript
const shouldComponentUpdate = ({ id, props }) => {
    return props.select.id === id;
};
```

**Note:** Will also check `propTypes` if they have been defined on the component.

### `args`

In [Haskell](https://www.haskell.org/) you have `all@` for accessing **all** of the arguments in a function, even after listing the arguments individually &mdash; with JavaScript you have the nonstandard `arguments` however with Keo `args` can be destructured to provide access to **all** of the arguments passed in, allowing you to forward these arguments to other functions.

```javascript
const greetingIn = (language, { props }) => {
    switch (language) {
        case 'en': return `Hello ${props.name}`;
        case 'de': return `Guten Tag ${props.name}`;
    }
};

const render = ({ props, context, args }) => {
    const greeting = greetingIn('en', args);
    // ...
    return <h1>${greeting}!</h1>
};
```

Which then allows you to destructure the arguments in the `greetingIn` function as though it's a typical lifecycle React method.

## Testing Smart Components

Whenever you pass the `mapStateToProps` argument to Keo's `stitch` function you create a [smart component](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0) &mdash; due to the wrapping that `react-redux` applies to these components they can be troublesome to test. As such they should ideally be exported as both a smart component for your application **and** as a dumb component for unit testing.

However Keo provides a convenient `unwrap` function to resolve smart components to dumb components for testing purposes &mdash; leaving your application to handle the smart components.

**Component:**

```javascript
import { stitch } from 'keo';

const render = ({ props }) => {
    return <h1>Hi {props.name}</h1>;
};

export default stitch({ render }, state => state);
```

**Unit Test:**

```javascript
import test from 'ava';
import { unwrap } from 'keo';
import Greet from './component';

test('We can unwrap the smart component for testing purposes', t => {

    const UnwrappedGreet = unwrap(Greet);
    const component = <UnwrappedGreet name='Philomena' />;
    
    // ...
    
    t.pass();
    
});
```
