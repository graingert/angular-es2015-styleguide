
# Angular ES2015 Style Guide

## This is still a Work In Progress - Please Collaborate

## Endorsements
The idea for this styleguide is inspired from [john papa's angular style guide](https://github.com/johnpapa/angular-styleguide) which is focused on angular version 1.x, written with ES5.
Some sections are currently taken from john papa's styleguide and are subject to change in the future (as this stylguide grows).
This styleguide is also inspired from [@angularclass's boilerplates](http://github.com/angularclass) for angular & es2015.

## Purpose
*Opinionated Angular.js written with ES2015 and module loadeing style guide for teams by [@orizens](//twitter.com/orizens)*

If you are looking for an opinionated style guide for syntax, conventions, and structuring Angular.js applications v 1.x with ES2015 and module loading, then this is it. These styles are based on my development experience with [Angular.js](//angularjs.org) and the [Echoes Player](http://echotu.be), [Open Source Project Development (currently the es2015 branch)](https://github.com/orizens/echoes/tree/es2015).

The purpose of this style guide is to provide guidance on building Angular applications with ES2015 and module loading by showing the conventions I use and in order to prepare angular v 1.x code to angular v 2.

## Inspirations
[NG6-Starter](https://github.com/AngularClass/NG6-starter)
[Angular, Gulp, Browserify](https://github.com/jakemmarsh/angularjs-gulp-browserify-boilerplate)

## See the Styles in a Sample App
This guide is based on my [open source Echoes Player application](http://echotu.be) that follows these styles and patterns. You can clone/fork the code at [echoes repository](https://github.com/orizens/echoes).

##Translations
TBD

## Table of Contents

1. [Single Responsibility](#single-responsibility)
1. [Modules](#modules)
1. [Components](#components)
1. [Services](#services)

## Single Responsibility

### Rule of 1

- Define 1  class per file.

The following example defines the several classes in the same file.

```javascript
// now-playlist.controllers.js
/* avoid */
export class NowPlaylistCtrl {}
export class NowPlaylistFilterCtrl {}
```

The same classes are now separated into their own files.

```javascript
/* recommended */

// now-playlist.ctrl.js
export default class NowPlaylistCtrl {}
```

```javascript
/* recommended */

// now-playlist-filter.ctrljs
export default class NowPlaylistFilterCtrl {}
```

**[Back to top](#table-of-contents)**

## Modules

Use [Proposed Loader Specification](https://whatwg.github.io/loader/) (Former ES6/ES2015)

**Why?**: it assists in bundling the app and promotes the seperation of concerns. In Addition, Angular 2 is also based on   Loader's standards.

```javascript
import NowPlaylist from './now-playlist';
import { NowPlaylistComponent } from './now-playlist.component';
```

### Module Loaders Tools

* [System.js](https://github.com/systemjs/systemjs)
* [Browserify](http://browserify.org/)
* [Webpack](https://webpack.github.io/)
* [Typescript CLI](http://www.typescriptlang.org/)
  * [tsd is a type definition package manager](http://definitelytyped.org/tsd/)

### Module Folder Structure

- each module directory should be named with a dash seperator (kebab notation).

**Why?**: it follows the web-component notation of seperating a tag name with a dash. It is easier to read and follow in the code editor.

```
// name of directory for <now-playlist></now-playlist> component
- now-playlist
```

### Module files

  - each module should contain the following:
  1. index.js - it should contain:
  1. module-name.component.js - a component (directive) file defintion with class as a controller
  1. an html template file
  1. a spec file

#### index.js - module entry point

  should contain:
  * the module defintion
  * components/directives angular wrappers
  * its dependencies
  * config phase & function
  * angular's entities wrappers - services, factories, additional components/directives, other..

**Why?**: this is the file where we can hook vanilla js files into angular. This is the main point to see what this module is composed of.

```javascript
import angular from 'angular';
import AngularSortableView from 'angular-sortable-view/src/angular-sortable-view.js';
import { NowPlaylistComponent } from './now-playlist.component';

export default angular.module('now-playlist', [
      'angular-sortable-view'
    ])
    .config(config)
    .directive(NowPlaylistComponent.controllerAs, () => NowPlaylistComponent)
    // with angular 1.5, the component definition is as follows:
    .component(NowPlaylistComponent.controllerAs, NowPlaylistComponent)
;
// optional
/* @ngInject */
function config () {

}
```
#### module-name.component.js - component defintion with controller class

  this file should contain:
  1. the component/directive **definition** as a literal object, with export.
  2. the **"controller"** property should be defined as a **class**.
  3. template should be imported from external file or inlined with template string es2015 syntax.

**Why?**: It's easy to understand the bigger picture of this component: what are the inputs and outputs retrieved from scope. Everything is in one place and easier to reference. Moreover, this syntax is similar to angular 2 component definion - having the component configuration above the "controller" class.

```javascript
import template from './now-playlist.tpl.html';

export let NowPlaylistComponent = {
        template,
        // if using inline template then use template strings (es2015):
        template: `
          <section class="now-playlist">
              ....
          </section>
        `,
        controllerAs: 'nowPlaylist',
        // or "bindings" to follow ng1.5 "component" factory
        scope: {
            videos: '=',
            filter: '=',
            nowPlaying: '=',
            onSelect: '&',
            onRemove: '&',
            onSort: '&'
        },
        bindToController: true,
        replace: true,
        restrict: 'E',
        controller:
/* @ngInject */
class NowPlaylistCtrl {
    /* @ngInject */
    constructor () {
        // injected with this.videos, this.onRemove, this.onSelect
        this.showPlaylistSaver = false;
    }

    removeVideo($event, video, $index) {
        this.onRemove && this.onRemove({ $event, video, $index });
    }

    selectVideo (video, $index) {
        this.onSelect && this.onSelect({ video, $index });
    }

    sortVideo($item, $indexTo) {
        this.onSort && this.onSort({ $item, $indexTo });
    }
}
}
```

## Components
* Use ES2015 class for controller
* Use **Object.assign** to expose injected services to a class methods (make it public)

**Why?** - ```Object.assign``` is a nice one liner usage for overloading services on "this" context, making it available to all methods in a service (i.e., **playVideo** method).

```javascript
/* @ngInject */
export default class YoutubeVideosCtrl {
	/* @ngInject */
	constructor (YoutubePlayerSettings, YoutubeSearch, YoutubeVideoInfo) {
		Object.assign(this, { YoutubePlayerSettings, YoutubeVideoInfo });
		this.videos = YoutubePlayerSettings.items;

		YoutubeSearch.resetPageToken();
		if (!this.videos.length) {
			YoutubeSearch.search();
		}
	}

	playVideo (video) {
		this.YoutubePlayerSettings.queueVideo(video);
		this.YoutubePlayerSettings.playVideoId(video);
	}

	playPlaylist (playlist) {
		return this.YoutubeVideoInfo.getPlaylist(playlist.id).then(this.YoutubePlayerSettings.playPlaylist);
	}
}
```

## Services
It's a best practice to write all logics in services.

**Why?**: logics can be reused in multiple files. logics can be tested easily whne in a service object.

### angular.service
Use **angular.service** api with a class for a service.

**Why?**: Services in angular 2 are classes. it'll be easier to migrate the code.

### angular.factory
use **angular.service** instead.

### angular.provider
export a function as provider as in angular with ES5.

## Application Structure

### Overall Guidelines
[-] - a folder
an * - a file

```
[-] src
  [-] components
  [-] core
    [-] components
    [-] services
  [-] css
  * app.js
  * index.html
```
#### src/components
This directory includes **smart components**. It consumes the **app.core** services and usually doesn't expose any api in attributes.
It's like an app inside a smart phone. It consumes the app's services (ask to consume it) and knows how to do its stuff.

Usage of such smart component is as follows:

```html
<now-playing></now-playing>
```

Example definition in **index.js** can be:

```javascript
import angular from 'angular';
import AppCore from '../core';
import nowPlaying from './now-playing.component.js';
import nowPlaylist from './now-playlist';
import nowPlaylistFilter from './now-playlist-filter';
import playlistSaver from './playlist-saver';
import YoutubePlayer from '../youtube-player';

export default angular.module('now-playing', [
      AppCore.name,
      nowPlaylist.name,
      nowPlaylistFilter.name,
      playlistSaver.name,
      YoutubePlayer.name
    ])
    .config(config)
    .directive('nowPlaying', nowPlaying)
;
/* @ngInject */
function config () {

}
```

#### src/core/components
This directory includes system wide **dumb components**. A Dumb Component gets data and fire events. It is communicating only through events.
This is example:
```javascript
<dropdown items="vm.presets" on-select="vm.handlePresetSelect(item, index)"></dropdown>
```

## Testing
Unit testing helps maintain clean code, as such I included some of my recommendations for unit testing foundations with links for more information.

### Write Tests with Stories

- Write a set of tests for every story. Start with an empty test and fill them in as you write the code for the story.

**Why?**: Writing the test descriptions helps clearly define what your story will do, will not do, and how you can measure success.

```javascript
it('should have a collection of media items', function() {
    // TODO
});

it('should find fetch metadata for a youtube media', function() {
    // TODO
});

```

### Testing Library

- Use [Jasmine](http://jasmine.github.io/) or [Mocha](http://mochajs.org) for unit testing.

**Why?**: Both Jasmine and Mocha are widely used in the Angular community. Both are stable, well maintained, and provide robust testing features.

Note: When using Mocha, also consider choosing an assert library such as [Chai](http://chaijs.com). I prefer Mocha.

### Test Runner

- Use [Karma](http://karma-runner.github.io) as a test runner.

**Why?**: Karma is easy to configure to run once or automatically when you change your code.

**Why?**: Karma hooks into your Continuous Integration process easily on its own or through Grunt or Gulp.

**[Back to top](#table-of-contents)**

### Organizing Tests

- Place unit test files (specs) side-by-side within the component's code.
- Place mocks in a **tests/mocks** folder

**Why?**: Unit tests have a direct correlation to a specific component and file in source code.

**Why?**: Mock files (json) should be agnostic to the component which is using them. Multiple components specs might use the same jsom mocks.

**[Back to top](#table-of-contents)**

## Comments
TBD

## ES Lint

### Use an Options File
- Use [eslint.org](http://eslint.org/) to deifne es2015 support
- Use **.eslintrc** file for linting and support es2015 features

**Why?**: Provides a first alert prior to committing any code to source control.

**Why?**: Provides consistency across your team.

```javascript
{
    arrowFunctions: true,
    classes: true,
    modules: true,
    restParams: true,
    spread: true,
    defaultParams: true
}
```
More To Come...

**[Back to top](#table-of-contents)**

## Routing
TBD

## Contributing

1. Open an issue for discussion
2. Create a pull request to suggest additions or changes

## License

Share your thoughts with an issue or pull request

### Copyright

Copyright (c) 2015-2016 [Oren Farhi](http://orizens.com)

**[Back to top](#table-of-contents)**
