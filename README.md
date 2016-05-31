## Catbee Web Components

DocumentRenderer implementaion based on [Web Components](https://developer.mozilla.org/en-US/docs/Web/Web_Components), spiced by [Appstate](https://github.com/catbee/appstate) and [Baobab](https://github.com/Yomguithereal/baobab) for state management. You can use it with any template engine (jade, handlebars, dust) and don't care about browser rendering because it's use [morphdom](https://github.com/patrick-steele-idem/morphdom) for patial DOM updates insteadof full-page rerendering.

### Getting Started

Web components is one of implementation Catbee view layer. It's should be register as Catbee service and provide root document component. Below you can get code example, that can be used for integration in browser.

``` javascript
const catbee = require('catbee');
const components = require('catbee-web-components');
const cat = catbee.create();

// Describe first component
const component = {
  constructor: class Document {
    template () {
      return 'Hello world';
    }
  }
}

components.register(cat.locator, component);

cat.startWhenReady();
```

Similarly, it can be integrated on server.

``` javascript
const catbee = require('catbee');
const components = require('catbee-web-components');
const express = require('express');

const cat = catbee.create();
const app = express();

// Describe first component
const component = {
  constructor: class Document {
    template () {
      return 'Hello world';
    }
  }
}

components.register(cat.locator, component);

app.use(cat.getMiddleware());
app.listen(3000);
```

### Instalation

``` 
npm install catbee-web-components --save
```

### Registration

``` javascript
const components = require('catbee-web-components');
components.register(locator, root);
```

### Component Specification

``` javascript
class Component {
  constructor (locator) {
    // Every component get locator as argument in constructor, 
    // it's can be used for resolve services (config, uhr, api, etc...)
    this._config = locator.resolve('config');
  }
  
  // Template is function that called when you render component to HTML string
  // It's accept ctx as first argument, and should return string.
  // You can use any template language that can be compiled to function.
  // For example, you can use Handlebars require hook on server, 
  // and webpack loader for client side bundle, and use next code:
  // 
  // constructor () {
  //   this.template = require('./template.hbs');
  // }
  template (ctx) {
    return `
      <!DOCTYPE html>
      <html lang="en">
      <head></head>
      <body>
        <a class="link">Hello ${ctx.world}</a>
        <cat-main></cat-main>
        <script src="/build.js"></script>
      </body>
      </html>
    `
  }
  
  render () {
    // This method create ctx for template function
    // You should return Object or Promise resolved by Object
    // Also here you can make any DOM manipulation, because at this step 
    // all nodes already in DOM.
    return { 
      world: 'Earth'
    }
  }
  
  bind () {
    // This method should return DOM event bindings map as Promise or Object
    return {
      click: {
        'a.link': (e) => console.log(e.currentTarget)
      }
    }
  }
  
  unbind () {
    // This method can be used as dispose component hook.
    // It's called before DOM is will be disposed and event listeners detached.
  }
}

module.exports = {
  // This is component descriptor.
  // It's provide information about component context usage
  constrcutor: Component,
  children: [
    // Here should be described all chidren components (cat-*)
    // Document and head, it's special names used by library for better dev expirience.
    {
      name: "head",
      component: require('./headDescriptor') // You should head descriptor, same object as exports here
    },
    {
      name: "main",
      component: require('./mainDescriptor'),
      watcher: {
        value: ['path', 'to', 'value']
      }, // Bind to component part of data tree. Use this.$context.getWatcherData() to get it.
      props: {
        size: 'big'
      } // Pass to component some static props. Will be avaliable as this.$context.props
    }
  ]
}
```

### Shared context
Catbee sets as the property $context for every instance of each signal action and component.

- this.$context.isBrowser – true if code is being executed in the browser.
- this.$context.isServer – true if code is being executed on the server.
- this.$context.userAgent – the current user agent string of the environment.
- this.$context.cookie – the current cookie wrapper object.
- this.$context.location – the current URI object that constains the current location.
- this.$context.referrer – the current URI object that contains the current referrer.
- this.$context.locator – the Service Locator of the application.
- this.$context.notFound() – hands over request handling to the next express/connect middleware. If used while rendering the document or head component, this action will be accomplished using HTTP headers and status codes on the server, else via an inline `<script>` tag.

### Component context
Every component's $context is extended with the following properties & methods:

- this.$context.getWatcherData() - return Promise resolved by state tree projection data.
- this.$context.signal(actions, args) - run [appstate](https://github.com/catbee/appstate) signal with actions array and args object.
- this.$context.state - the current application state reference.
- this.$context.element – the current DOM element that represents the current component.
- this.$context.attributes – the set of attributes which component's DOM element has at the moment.
- this.$context.getComponentById('id') – gets another component object by ID of its element.
- this.$context.getComponentByElement(domElement) – gets another component's object by its DOM element.
- this.$context.createComponent('tagName', attributesObject) – creates a new component's instance and returns a promise of its DOM element.
- this.$context.collectGarbage() – collects all components which have been created using the createComponent('tagName', attributesObject) method and are not attached to the DOM at the moment.

