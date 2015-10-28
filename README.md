# neon-page-behavior

_Make the most of Polymer's [`<neon-animated-pages>`](https://github.com/PolymerElements/neon-animation#page-transitions) effortlessly. NeonPageBehavior fires events allowing more control over a page's lifecycle, and allows your page element to use a different animation-configuration when transitioning to each different page._

Please refer to the <a href="https://vguillou.github.io/webcomponents/neon-page-behavior/">component page</a> for more informations.

* [Page lifecycle](#lifecycle)
* [Declaring different animation configurations](#animation)
  * [animationConfig](#animation_config)
  * [sharedElements](#shared_elements)

<a name="lifecycle"></a>
## Page lifecycle

Elements having the `NeonPageBehavior` and being a child of a [`<neon-animated-pages>`](https://github.com/PolymerElements/neon-animation#page-transitions) element can listen to 4 new events:

* **`entry-animation-start`**:
Called BEFORE the transition TO the element starts.
Useful to handle initialization before your element gets visible (start loading data, animation optimisation,...).

* **`entry-animation-finish`**:
Called AFTER the transition TO the element finished.
Useful to finish initialization of your element (allow user focus,...).

* **`exit-animation-start`**:
Called BEFORE the transition FROM the element starts.
Useful to deal with exit tasks (disallow user focus, animation optimisation,...).

* **`exit-animation-finish`**:
Called AFTER the transition FROM the element finished.
Useful to handle exit tasks when your element isn't visible anymore (reset scroller position,...).

The `detail` of the dispatched events contains the following properties :

* **`animationConfig`**:
The `animationConfig` of the target page for the transition.

* **`sharedElements`**:
The `sharedElements` of the target page for the transition.

* **`from`**:
The identifier of the original page of the transition, as in `neon-animated-pages.selected`.

* **`fromPage`**:
The reference to the original page of the transition.

* **`to`**:
The identifier of the destination page of the transition, as in `neon-animated-pages.selected`.

* **`toPage`**:
The reference to the destination page of the transition.


### Example

_index.html_
```html
<neon-animated-pages class="flex" entry-animation="fade-in-animation" exit-animation="fade-out-animation">
	<my-neon-page title="Page 1" style="background-color: red;"></my-neon-page>
	<my-neon-page title="Page 2" style="background-color: blue;"></my-neon-page>
</neon-animated-pages>
```

_my-neon-page.html_
```html
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
		});
	</script>
</dom-module>
```

<a name="animation"></a>
## Declaring different animation configurations

<a name="animation_config"></a>
### animationConfig
Elements having the `NeonPageBehavior` and being child of a [`<neon-animated-pages>`](https://github.com/PolymerElements/neon-animation#page-transitions) can also declared different `animationConfig` properties that will be used automatically for transitioning to and from each different page.
These properties' names must respect the following naming convention:

`animationConfig` + Capitalized value representing the page to transition from/to for the parent `<neon-animated-pages>` (see `selected` and `attrForSelected` properties in the [`<neon-animated-pages>`](https://github.com/PolymerElements/neon-animation#page-transitions) documentation for more detail on this), all normalized to become a valid javascript variable name.

<a name="shared_elements"></a>
### sharedElements
If your element also have the [`NeonSharedElementAnimatableBehavior`](https://elements.polymer-project.org/elements/neon-animation?active=Polymer.NeonSharedElementAnimatableBehavior), you can similarly declare different `sharedElements` properties for each different page to transition from/to. The naming convention is the following:

`sharedElements` + Capitalized value representing the page to transition from/to, all normalized to become a valid javascript variable name.

You can also differentiate the `sharedElements` for the transition FROM a given page (entering this element) from the `sharedElements` for the transition TO a given page (exiting this element) by following this naming convention:

`sharedElements` + Capitalized value representing the page to transition from/to + `Entry` or `Exit`, all normalized to become a valid javascript variable name.

### Example

_index.html_
```html
<neon-animated-pages class="flex" attr-for-selected="name">
	<my-neon-page name="Page1" style="background-color: red;"></my-neon-page>
	<my-page-2 name="Page2" style="background-color: blue;"></my-page-2>
	<my-page-3 name="Page3" style="background-color: green;"></my-page-3>
	<my-page-4 name="Page4" style="background-color: yellow;"></my-page-4>
</neon-animated-pages>
```

_my-neon-page.html_
```html
<dom-module id="my-neon-page">
  <template>
    <div id="myHero1"><span>1</span></div>
    <div id="myHero2"><span>2</span></div>
  </template>
  <script>
    Polymer({

      is: 'my-neon-page',

      behaviors: [
        Polymer.NeonSharedElementAnimatableBehavior,
        Polymer.NeonPageBehavior
      ],

      properties: {
        // Default animation config : slide up when entering, left and right when exiting
        animationConfig: {
          type: Object,
          value: function() {
            return {
              'entry': [{
                name: 'transform-animation',
                transformFrom: 'translateY(100%)',
                node: this
              }],
              'exit': [{
                name: 'slide-left-animation',
                node: this.$.myHero1
              },
              {
                name: 'slide-right-animation',
                node: this.$.myHero2
              }]
            };
          }
        },

        // Specific animation config for Page2: Entering from Page2, myHero1 is a hero and myHero2 scales. The other way around when exiting to Page2
        animationConfigPage2: {
          type: Object,
          value: function() {
            return {
              'entry': [{
                name: 'hero-animation',
                id: 'hero',
                toPage: this
              },
              {
                name: 'scale-up-animation',
                node: this.$.myHero2
              }],
              'exit': [{
                name: 'hero-animation',
                id: 'hero',
                fromPage: this
              },
              {
                name: 'scale-down-animation',
                node: this.$.myHero1
              }]
            };
          }
        },

        // Shared elements when entering this page from Page2: myHero1 is the hero
        sharedElementsPage2Entry: {
          type: Object,
          value: function() {
            return {
              'hero': this.$.myHero1
            };
          }
        },

        // Shared elements when exiting this page to Page2: myHero2 is the hero
        sharedElementsPage2Exit: {
          type: Object,
          value: function() {
            return {
              'hero': this.$.myHero2
            };
          }
        }
      }
    });
  </script>
</dom-module>
```

## Demos

[Here](https://vguillou.github.io/webcomponents/neon-page-behavior/demo/index.html)

## TODOs

1. Let the dev configure externally which "animationConfig" object should be used for the transitions to each page, instead of directly fetching the "animationConfig[NextPage]" or "animationConfig[PreviousPage]" property of the neon-page
2. Improve documentation
3. ...
4. Profit

## License

[MIT License](http://opensource.org/licenses/MIT)
