# neon-page-behavior

_Make the most of Polymer's 'neon-animated-pages' effortlessly. NeonPageBehavior fires events allowing more control over a page's lifecycle, and allows your page element to use a different animation-configuration when transitioning to each different page._

## Page lifecycle

Elements having the `NeonPageBehavior` and being a child of a `<neon-animated-pages>` element can listen to 4 new events:

* **entry-animation-start**:
Called BEFORE the transition TO the element starts.
Useful to handle initialization before your element gets visible (start loading data, animation optimisation,...).

* **entry-animation-finish**:
Called AFTER the transition TO the element starts.
Useful to finish initialization of your element (allow user focus,...).

* **exit-animation-start**:
Called BEFORE the transition FROM the element starts.
Useful to deal with exit tasks (disallow user focus, animation optimisation,...).

* **exit-animation-finish**:
Called AFTER the transition FROM the element starts.
Useful to handle exit tasks when your element isn't visible anymore (reset scroller position,...).

### Example

_index.html_
```html
<neon-animated-pages class="flex" entry-animation="fade-in-animation" exit-animation="fade-out-animation">
	<my-neon-page title="Page 1" style="background-color: red;"></neon-page>
	<my-neon-page title="Page 2" style="background-color: blue;"></neon-page>
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
</polymer-element>
```

## Animation configurations

Elements having the `NeonPageBehavior` and being child of a `<neon-animated-pages>` can also declared different `animationConfig` properties that will be used automatically for transitioning to and from each different page.
These properties' names must respect the following naming convention: `animationConfig` + Capitalized value representing the page to transition from/to for the parent `<neon-animated-pages>` (see `selected` and `attrForSelected` properties in the `<neon-animated-pages>` documentation for more detail on this).

If your element also have the `NeonSharedElementAnimatableBehavior`, you can similarly declare different `sharedElements` properties for each different page to transition from/to. The naming convention is the following: `sharedElements` + Capitalized value representing the page to transition from/to.
You can also differentiate the `sharedElements` for the transition FROM a given page (entering this element) from the `sharedElements` for the transition TO a given page (exiting this element) by following this naming convention: `sharedElements` + Capitalized value representing the page to transition from/to + `Entry` or `Exit`.

### Example

_index.html_
```html
<neon-animated-pages class="flex" attr-for-selected="name">
	<my-neon-page name="Page1" style="background-color: red;"></neon-page>
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
</polymer-element>
```

## Demos

[Here](https://vguillou.github.io/webcomponents/neon-page-behavior/demo/index.html)

## License

[MIT License](http://opensource.org/licenses/MIT)