# neon-page-behavior

[![Build Status](https://travis-ci.org/vguillou/neon-page-behavior.svg?branch=master)](https://travis-ci.org/vguillou/neon-page-behavior)
[![GitHub version](https://badge.fury.io/gh/vguillou%2Fneon-page-behavior.svg)](https://badge.fury.io/gh/vguillou%2Fneon-page-behavior)

_Make the most of Polymer's [`<neon-animated-pages>`](https://github.com/PolymerElements/neon-animation#page-transitions) effortlessly. NeonPageBehavior fires events allowing more control over a page's lifecycle, allows your page element to use a different animation-configuration when transitioning to each different page, and makes it possible to display a page as an overlay on top of another._

Please refer to the <a href="https://vguillou.github.io/webcomponents/neon-page-behavior/">component page</a> for more informations, or directly take a look at the [demos](https://vguillou.github.io/webcomponents/neon-page-behavior/demo/index.html).

* [Why do you need NeonPageBehavior](#why)
* [The page lifecycle](#lifecycle)
* [Declaring different animation configurations](#animation)
  * [animationConfig](#animation_config)
  * [sharedElements](#shared_elements)
* [Setup Overlay page](#overlay)

<a name="why"></a>
## Why do you need NeonPageBehavior ?
[`<neon-animated-pages>`](https://github.com/PolymerElements/neon-animation#page-transitions) is great for building
single-page application with engaging page transitions. However it currently has a few shortcomings:

1. _Pages are not aware of their lifecycle_, and a good amount of boilerplate is necessary to introduce this notion:
A page including the `NeonPageBehavior` takes care of that for you,
and can __listen to events fired before and after both enter and exit transitions__.

2. _You can only setup declaratively one transition_, no matter how many pages your application is made up of:
Using `NeonPageBehavior`, you can __setup declaratively a different animation to transition
to each and every page of you application__.

3. _There is no built-in solution to make a page overlay another_:
__`NeonPageBehavior` offers a solution to declaratively setup overlay pages__,
and thus plug them into you routing solution without any extra code.


<a name="lifecycle"></a>
## The page lifecycle 

An element being extended with the `NeonPageBehavior` (and of course being a child of a
[`<neon-animated-pages>`](https://github.com/PolymerElements/neon-animation#page-transitions)
element) can listen to 4 new events:

- `entry-animation-start`

    __Called BEFORE the transition TO the page starts__.
    Useful to handle initialization before your page gets visible
    (start loading data, animation optimisation,...).
    _However it is not recommended to launch heavy processing at this
    time not to hinder the transition's animation._

- `entry-animation-finish`

     __Called AFTER the transition TO the page finished__.
     Useful to finish initializing your page (allow user focus,...).

- `exit-animation-start`

     __Called BEFORE the transition FROM the page starts__.
     Useful to deal with exit tasks (disallow user focus, animation optimisation,...).
     _However it is not recommended to launch heavy processing at this
     time not to hinder the transition's animation._

- `exit-animation-finish`

     __Called AFTER the transition FROM the page finished__.
     Useful to handle exit tasks when your page isn't visible anymore
     (reset scroller position,...).

The `detail` property of those dispatched events contains the following properties :
- `animationConfig`:
     The `animationConfig` of the page (ie `this`) for the transition.

- `sharedElements`:
     The `sharedElements` of the page (ie `this`) for the transition.

- `from`:
     The identifier of the original page of the transition (ie the previously selected page),
     as in `neon-animated-pages.selected`.

- `fromPage`:
     The reference to the original page of the transition (ie the previously selected page).

- `to`:
     The identifier of the destination page of the transition (ie the newly selected page),
     as in `neon-animated-pages.selected`.

- `toPage`:
     The reference to the destination page of the transition (ie the newly selected page),.

#### Example

_index.html_
```html
<neon-animated-pages class="flex" entry-animation="fade-in-animation" exit-animation="fade-out-animation">
  <my-neon-page title="Page 1" style="background-color: red;"></my-neon-page>
  <my-neon-page title="Page 2" style="background-color: blue;"></my-neon-page>
</neon-animated-pages>
```

_my-neon-page.html_
```js
<dom-module id="my-neon-page">
  <template>
    <div>{{title}}</div>
  </template>
  <script>
    Polymer({

      is: 'my-neon-page',

      behaviors: [
        Polymer.NeonAnimatableBehavior,
        Polymer.NeonPageBehavior
      ],

      properties: {
        title: {
          type: String
        }
      },

      listeners: {
        'entry-animation-start': 'onEntryStart',
        'entry-animation-finish': 'onEntryFinish',
        'exit-animation-start': 'onExitStart',
        'exit-animation-finish': 'onExitFinish'
      },

      onEntryStart: function(e) {
        console.log(this.title + ' entry animation starts');
      },
      onEntryFinish: function(e) {
        console.log(this.title + ' entry animation finished');
      },
      onExitStart: function(e) {
        console.log(this.title + ' exit animation starts');
      },
      onExitFinish: function(e) {
        console.log(this.title + ' exit animation finished');
      }

      ...
```



<a name="animation"></a>
## Declaring different animation configurations

Chances are your SPA's will contain more than a couple of pages. 

Let's consider a simple SPA with 4 pages: `page-list` displaying a list of items, `page-detail` showing more information about a selected item, and finally `page-settings-1` and `page-settings-2` that are two general preference pages that can be access from all other pages.

It's very likely that you may want to implement a nice transition between `page-list` and `page-detail`, such as a hero animation, and something different for transitioning to the `page-settings-1` and `page-settings-2`.

Elements having the `NeonPageBehavior` (and of course being child of a [`<neon-animated-pages>`](https://github.com/PolymerElements/neon-animation#page-transitions)) can let you do just that by supporting different `animationConfig` properties for every single page transition, in addition to the global one.

#### Example

_index.html_
```html
<neon-animated-pages attr-for-selected="name">
  <page-list name="list"></page-list>
  <page-detail name="detail"></page-detail>
  <page-settings-1 name="settings1"></page-settings-1>
  <page-settings-2 name="settings2"></page-settings-2>
</neon-animated-pages>
```

_page-list.html_
```js
// Specific animation config used when transitioning to/from `page-detail`
animationConfigDetail: {
  type: Object,
  value: function() {
    return {
      'entry': [{
        name: 'hero-animation',
        id: 'hero',
        toPage: this
      },
      {
        name: 'fade-in-animation',
        node: this
      }],
      'exit': [{
        name: 'hero-animation',
        id: 'hero',
        fromPage: this
      },
      {
        name: 'fade-out-animation',
        node: this
      }]
    };
  }
},

// Shared elements when going to `page-detail`
sharedElementsDetail: {
  type: Object,
  value: function() {
    return {
      'hero': this.$.myHero
    };
  }
},

// Global animation config that will be used for any other page
// (`page-settings-1` and `page-settings-2` in our example)
animationConfig: {
  type: Object,
  value: function() {
    return {
      'entry': {
        name: 'slide-from-bottom-animation',
        node: this
      },
      'exit': {
        name: 'slide-down-animation',
        node: this
      }
    };
  }
}
```

_page-detail.html_
```js
// Specific animation config used when transitioning to/from `page-list`
animationConfigList: {
  type: Object,
  value: function() {
    return {
      'entry': [{
        name: 'hero-animation',
        id: 'hero',
        toPage: this
      },
      {
        name: 'fade-in-animation',
        node: this
      }],
      'exit': [{
        name: 'hero-animation',
        id: 'hero',
        fromPage: this
      },
      {
        name: 'fade-out-animation',
        node: this
      }]
    };
  }
},

// Shared elements when going to `page-list`
sharedElementsList: {
  type: Object,
  value: function() {
    return {
      'hero': this.$.myHero
    };
  }
},

// Global animation config that will be used for any other page
// (`page-settings-1` and `page-settings-2` in our example)
animationConfig: {
  type: Object,
  value: function() {
    return {
      'entry': {
        name: 'slide-from-bottom-animation',
        node: this
      },
      'exit': {
        name: 'slide-down-animation',
        node: this
      }
    };
  }
}
```

_page-settings-1.html_ and _page-settings-2.html_
```js
// Always the same transition animation for every page
animationConfig: {
  type: Object,
  value: function() {
    return {
      'entry': {
        name: 'fade-in-animation',
        node: this
      },
      'exit': {
        name: 'fade-out-animation',
        node: this
      }
    };
  }
}
```



<a name="animation_config"></a>
### animationConfig

All `animationConfig` properties must respect the following naming convention:

__`animationConfig` + value representing the page to transition from/to__ for the parent `<neon-animated-pages>`
(see `selected` and `attrForSelected` properties in the
[`<neon-animated-pages>`](https://github.com/PolymerElements/neon-animation#page-transitions)
documentation for more detail on this), all normalized to become a valid javascript variable name.
(ie if `pageValue`='home-alone', the `animationConfigHomeAlone` property will be used if it is defined, and `animationConfig` if not).

<a name="shared_elements"></a>
### sharedElements

If your element also have the [`NeonSharedElementAnimatableBehavior`](https://elements.polymer-project.org/elements/neon-animation?active=Polymer.NeonSharedElementAnimatableBehavior), you can similarly
declare different `sharedElements` properties for each different page to transition
from/to. The naming convention is the following:

__`sharedElements` + value representing the page to transition from/to__, all normalized to become a valid javascript variable name.
(ie if `pageValue`='home-alone', the `sharedElementsHomeAlone` property will be used if it is defined, and `sharedElements` if not).

__You can also differentiate the `sharedElements` to use for the transition FROM a given page
and the `sharedElements` to use for the transition TO a given page__ by following this naming convention:

__`sharedElements` + value representing the page to transition from/to + `Entry` or `Exit`__, all normalized to become a valid javascript variable name.
(ie if `pageValue`='home-alone' and entering this page from the page 'home-alone', the `sharedElementsHomeAloneEntry` property will be used if it is defined, `sharedElementsHomeAlone` otherwise and `sharedElements` if none of the 2 aforementioned properties are defined).


### Usefull to know:
- Under the hood, this behavior swaps the `animationConfig` and `sharedElements` properties of your page with the page-specific ones (for example `animationConfigMyPage` and `sharedElementsMyPage`) for the duration of the transition.
- A direct consequence of this implementation is that if you need to setup dynamic transitions, for example by using the clicked element of a list as the target node of an animation, you must modify the page-specific `animationConfig` and `sharedElements` (for example `animationConfigMyPage` and `sharedElementsMyPage`) and not the global `animationConfig` and `sharedElements` directly.



<a name="overlay"></a>
## Setup an Overlay page

__`NeonPageBehavior` offers a solution to declaratively setup overlay pages__, simply by adding the `overlay-page` attribute to your page(s).
This will allow the page to be displayed over the previously selected one (which will be given the `background-page` attribute).
No styling (other than setting the `z-index` to appear on top) is done, so this remains entirely up to the developer.

#### Example

_index.html_
```html
<neon-animated-pages attr-for-selected="name">
  <normal-page name="normalPage"></normal-page>
  <overlay-page name="overlayPage" overlay-page></overlay-page>
</neon-animated-pages>
```

### Usefull to know:
- It is not possible to display nested overlay pages (this is a pretty bad UX practice).
- For overlay pages to be displayed properly, __all__ children (pages) of the
[`<neon-animated-pages>`](https://github.com/PolymerElements/neon-animation#page-transitions)
must be including the `NeonPageBehavior`.
- An overlay page is styled to appear on top of the others by setting its CSS `z-index` property (to `1001`). You must ensure no element in a background page has a higher `z-index`.
- If a page transitions to a `background-page` state (ie is displayed under an overlay page), the hypothetical `sharedElements` of this transition will be hidden automatically, until the page stops being a `background-page`. You may set the `backgroundPageShowsSharedElements` attribute if you want to prevent this (you probably do not, as the `sharedElements` will then awkwardly "pop" back after the transition).



## Demos

[You can find them here.](https://vguillou.github.io/webcomponents/neon-page-behavior/demo/index.html)

## License

[MIT License](https://github.com/vguillou/neon-page-behavior/blob/master/LICENSE.md)
