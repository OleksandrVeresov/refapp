# Operator reference application

The [operator reference appliation](https://github.com/LibertyGlobal/refapp) provides a basic GUI that allows for playback of QAM and IP video and startup of various types of applications in the [RDK](https://wiki.rdkcentral.com/display/RDK/RDK+Central+Wiki) environment. It is a JavaScript application that runs in the [Spark](http://www.sparkui.org/) Application Engine with [lgire](https://code.rdkcentral.com/r/plugins/gitiles/components/generic/js-plugins/) as thin graphic abstraction layer on top.  For control of QAM, IP and applications the refapp uses the api offered by the RDK components sessionmanager, aamp and optimus.

The refapp serves as example, starting point for how operators, integrators, community members can glue the various pieces of RDK together into a user experience. It works standalone without need for any Comcast or Liberty Global backend.

The refapp is also useful for RDK integration testing as it exercises the various rdk components and requires them to work well together (vs standalone component testing). It is used in RDK Continuous Integration testing.

There is an "[rdk-generic-hybrid-refapp](https://code.rdkcentral.com/r/plugins/gitiles/components/generic/rdk-oe/meta-cmf-video-restricted/+/master/recipes-core/images/rdk-generic-hybrid-refapp-image.bb)" yocto image available that contains the refapp javascript application and the various native RDK components that it makes use of such as [spark](https://github.com/pxscene/pxCore/tree/master/examples/pxScene2d), [rmfstreamer](https://code.rdkcentral.com/r/plugins/gitiles/components/generic/rdk-oe/meta-cmf-video-restricted/+/master/recipes-extended/rmfstreamer/rmfstreamer_git.bbappend) with [sessionmanager](https://code.rdkcentral.com/r/plugins/gitiles/components/generic/sessionmgr), IP players ([aamp](https://github.com/rdkcmf/rdk-aamp) and [Liberty IPplayer](https://code.rdkcentral.com/r/plugins/gitiles/components/generic/websocket-ipplayer2)), [WPE browser](https://github.com/WebPlatformForEmbedded/WPEWebKit), [westeros](https://github.com/rdkcmf/westeros) and [optimus](https://github.com/pxscene/pxCore/blob/b8e0aa0e26e83d78cdb44cd55bd1e42e268171e9/examples/pxScene2d/src/rcvrcore/optimus.js) application manager.

Wiki page on refapp with more information is available on https://wiki.rdkcentral.com/display/RDK/Operator+Reference+Application
(you need to be registered RDK member to access it)

## Table of contents

- [Getting started](#getting-started)
- [Refapp breakdown](#refapp-breakdown)
	- [Components](#components)
	    - [Apps](#apps)
		- [Channel Bar](#channel-bar)
		- [Detail Page](#detail-page)
		- [Main Menu](#main-menu)
		- [MastHead](#masthead)
		- [OnDemand](#ondemand)
		- [Numeric input popup](#num-input)
		- [Warning popup](#warning)
	- [Domain models](#domain-models)
	    - [Channel](#channel)
		- [OnDemand](#ondemand-domain)
		- [Player](#player-domain)
		- [System](#system)
	- [Navigation](#navigation)
	- [Renderer](#renderer)
	    - [List](#list)
	- [Shared](#shared)
	    - [Animation](#animation)
		- [View Model](#view-model)
		- [Dynamic List](#dynamic-list)
- [Contributing](#contributing)

## <a name="getting-started"></a>Getting started
see wiki on RDK 
https://wiki.rdkcentral.com/display/RDK/Operator+Reference+Application
(you need to be registered RDK member to access it)
There you can also download prebuild image for Raspberry pi 

## <a name="refapp-breakdown"></a>Refapp breakdown


### <a name="components"></a>Components

#### <a name="apps"></a>Apps
Apps components is a collection of different applications and services that are integrated in our application.

#### <a name="channel-bar"></a>Channel Bar
Channel bar is a component which provides you possibility to navigate among channels while you are watching some event and player is top view.
It also shows channel name and logo, start and end time of currently playing event on each channel.

#### <a name="detail-page"></a>Detail Page
Detail page component is used for showing information about movies in [OnDemand](#ondemand).

#### <a name="main-menu"></a>Main Menu
Main Menu component is used for navigating among main 'big' components  like [Apps](#apps), [OnDemand](#ondemand).

#### <a name="masthead"></a>MastHead
MastHead component is header which provides specific information about the top view. It also shows current time and
the name of the event which is currently playing.

#### <a name="ondemand"></a>OnDemand
OnDemand component is a collection of movies. It has Recommended section which shows you assets based on what you have seen recently,
and section with all movies.

#### <a name="num-input"></a>Numeric Input popup
Numeric input component is used for faster navigation in [Channel Bar](#channel-bar).
RCU Manager handles key presses and provides key codes which are mapped to numbers in controller.

#### <a name="warning"></a>Warning popup
Warning popup is a component which can be used for different purposes from showing errors to
showing some information about system and other things

### <a name="domain-models"></a>Domain models

#### <a name="channel"></a>Channel

Channel domain model is used for retrieving channels with logos and info from cache.

Example:

```javascript
const { go, paths } = require('navigation');
const channelDomainModel = require('domainModels/channel');
channelDomainModel.setCurrentChannel(channelDomainModel.getCurrentChannel())
	 .then(() => go(paths.channelBar))
	 .catch(err => console.error(err));
```

#### <a name="ondemand-domain"></a>OnDemand
OnDemand domain model is used for retrieving movies from cache.

Example:

```javascript
const { getMovies, getRecommendedMovies } = require('ondemand');
   Promise.all([
       getMovies(),
       getRecommendedMovies(),
   ]).then(([moviesData, recommendedData]) => {
	//do what you need to
   }).catch(err => console.error(err));
```
####  <a name="player-domain"></a>Player
Player domain model is used for retrieving entities to play.
Example:

```javascript
const player = require('domainModels/player');
const entity = player.getCurrentPlayableEntity();
// then prepare entity for showing
```

#### <a name="system"></a>System
System domain model is used to manage events execution without random delay when multiple events
were emitted simultaneously. Also it has internal clock to count current time for different purposes.
There are `setSystemInterval()` and `clearSystemInterval()` for setting updates each N seconds and clear them.
`GetNow()` is used for retrieving current time.

### <a name="navigation"></a>Navigation
Navigation module is used for showing specific components in proper order.
Navigator can work both with views and popups

Example:

```javascript
const { init: initNavigator, go, paths } = require('navigation');
initNavigator();
go(paths.mainMenu);
```

In this example we init navigator by importing array with all possible routes.
Then we navigate to main menu by calling `go()` method, which is calling methond `find()`.
This method is traversing array with routes until it finds the path and then calls method `show()` for each
component it met on the way.

Navigator also handles the 'back' behavior. It adds components to the stack, so you can return to previous views.

### <a name="renderer"></a> Renderer
Renderer is a module for creating new or updating exisiting nodes.
Node in essence is a square to which we add styles.
Possible styles are stored in StyleProperties array, which is used for creating StylesExecutionList,
so each style has it's own position in queue for applying.

There are 2 ways of **creating nodes**:
1.Creating only one node:

```javascript
const { createOrUpdateNode } = require('renderer');
const styles = {
	width: 100,
	height: 100,
	backgroundColor: '#000000',
};
let node = createOrUpdateNode('id-1', styles, 'parent of the node');
```

2.Creating the whole layout:

```javascript
const { renderLayout } = require('renderer');
const layout = {
	id: 'id-1',
	styles: {},
	children: [
		{
		   id: 'id-2',
		   styles: {},
		},
		{
		   id: 'id-3',
		   styles: {},
		}
	]
};
let viewNode = renderLayout(layout, 'parent (root node by default)');
```

You can also **remove nodes** by specific id for optimization if you don't need it anymore:

```javascript
const { removeNode } = require('renderer');
removeNode('id-1');
```
#### <a name="list"></a>List
Layout helper which takes an array of elements, creates a wrapper and applies top or left position for passed elements, depending on the initial direction. List can include other list or any static layout structures as its elements. Also you can set margins between all elements by defining elemMargin props.
```javascript
const list = require('renderer/list');
const { listDirections } = require('shared/constants');

const staticVerticalList = list({
    direction: listDirections.COLUMN,
    elemMargin: 30,
    id: 'wrapper-node-id',
    styles: {
        // any styles for wrapper node
    }
    elements: [
        {
            id: 'child-id-0',
            styles: {}
        },
        list({
            direction: listDirections.ROW,
            elements: [/* some other layouts */],
        })
    ],
});
```

### <a name="shared"></a> Shared
#### <a name="animation"></a>Animation
Example:

```javascript
const Animation = require('shared/animation');
//animate node
Animation.animate('node instance', {
    timingFunction: Animation.timingFunctions.LINEAR,
    duration: 0.15,
    xDistance: -240,
});
//cancel animation
Animation.cancel('node instance');
```

#### <a name="view-model"></a>ViewModel
View Model is used for creating models for different views to contain static and dynamic properties.
You can use method `bindWatcher()` to create a handler which triggers when your value has been changed. `updateView()` method tells watchers to check their values if they was changed.
Example:

```javascript
const ViewModel = require('shared/ViewModel');
const model = new ViewModel({
    //static data
});
model.set('key of the dynamic data', 'dynamic data');
model.bindWatcher('key of the dynamic data', (oldValue, newValue) => {
    //make your updates here
});
model.updateView();
```

#### <a name="dynamic-list"></a>Dynamic List
Dynamic List is a helper which is used for creating lists (wrapper node) that can be dynamically extended and updated.
Dynamic List can be used in two cases:
1. Make a not movable vertical(column) or horizontal(row) list of items, which can dynamically change its capacity by setting new data.
2. Make a movable vertical(column) or horizontal(row) list of items. In this case dynamic list deletes all previous items and fully renders new items.

DynamicList includes five methods for manage state:
1. `generateList()` method generates items from data and set movable or not movable mode of this list;
2. If you make movable list, you can use `move()` method for moving list items. This method automatically generates new items before rendering if necessary. But previous items are never deleted from list;
3. `updateItem()` method updates existing item by passing index and new layout;
4. `restore()` method deletes current items and reset all initial props of dynamic list.
5. Getter `items` return actual map of items with structure like `{ index => { styles: { actual item styles } } }`

It's also necessary to create a function that will be called each time you create item. This function should accept two arguments: `itemData` - data for item and `prevItem` - previous items layout. Function should return layout object.

**Notes:** It is necessary to set `root` prop to the DynamicList instance before calling `generateList()` method.
```javascript
const DynamicList = require('shared/DynamicList');
const { listDirections } = require('shared/constants');

// function-generator
function listItemGenerator(itemData, prevItem) {
    // return layout object without ids
    return {
        styles: {},
        children: [
            {
                styles: {},
                children: [/*...*/]
            },
            // ...
        ]
    };
}

const dynamicList = new DynamicList({
        root: { id: 'nodeId', node: 'parent or root node instance'},
        id: 'dynamicList-node-id',
        styles: {
            // any styles for wrapper node
        }
        listItemGenerator,
        direction: listDirections.ROW
    });

dynamicList
    .generateList({ elementsData: [/* array of item data */], movable: true })
    .move({ size = -150, duration = 0.2, timingFunction = timingFunctions.LINEAR })
    .updateItem(0, {
        styles: { /* any styles for current item */}
        children: [/* layouts of the children nodes */]
    }
    restore();

console.log(dynamicList.items) // `{ 0 => { styles: {...} } }`
```
## <a name="contributing"></a>Contributing
- [Bug reports](https://github.com/LibertyGlobal/refapp/issues)

