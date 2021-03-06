![Alt text](https://github.com/hij1nx/SugarSkull/raw/master/img/sugarskull.png)

## What?

SugarSkull is a client side URL router. It's the smallest amount of glue needed for building dynamic single page applications. Not a jquery plugin, no dependencies.

## Why?

Storing some information about the state of an application within the URL allows the URL of the application to be emailed, bookmarked or copied and pasted. When the URL is visited it restores the state of the application. A client side router will also notify the browser about changes to the page, so even if the page does not reload, the back/forward buttons will give the illusion of navigation.

The HTML5 history API isn't quite a replacement for using the location hash. The HTML5 history API requires that a URL resolves to real assets on the server. And is designed around the requirement that all pages *should* load without Javascript. SugarSkull targets script-rich applications who's audience is 'well-known'.

SugarSkull enhances ***backbone.js***! It covers more use cases and provides a more expressive way to define routes and associate logic with them. It's a lightweight alternative to ***sammy.js***, narrowly focused on routing.

Are single page apps a problem for SEO? Yes and No. SugarSkull is meant for script-heavy web-apps, you can use it for web-sites, but learn how google and other search engines crawl and index pages before you decide on anything.

## How?

SugarSkull monitors the URL. When the URL changes, and it is a match to one defined in your router table, the functions that are associated with that route are executed. You could almost think of the URL as an event emitter.

More specifically the way this works is that we divide the url into two parts. First the server-side (everything 
before the '#'), and then the client-side (everything after the '#'). The second part is the HashRoute.
A hash route looks like this...<br/><br/>
<img src="https://github.com/hij1nx/SugarSkull/raw/master/img/hashRoute.png" width="598" height="113" alt="HashRoute" -->

## Cross Browser
Needs testing.

## Usage

First, the router constructor accepts an object literal that will serve as the routing table. Optionally, it can also accept a second object literal that contains functions. The second option is useful when the functions to be called get defined or loaded after the router gets defined.

### A trivial demonstration

```javascript
    var router = Router({

      '/dog': bark,
      '/cat': meow

    });
```

In the above code, the object literal contains a set of key/value pairs. The keys represent each potential part of the URL. The values contain instructions about what to do when there is an actual match. `bark` and `meow` are two functions that you have defined in your code.

### More complex URLs
```javascript
    var router = Router({

      '/dog': {
        '/angry': {
          on: growl
        },
        on: bark
      }

    });
```

Above is a case where the URL's are more complex. Routes can have many events and properties, `on`, `before`, `after`, etc. 

```javascript
    var router = Router({

      '/dog': {
        '/(\\w+)': {
          on: function(urlPathPart) {}
        },
        on: bark
      }

    });
```

In the above code, you'll notice that you can also use regular expressions inside the URLs. The capture groups from the regular expressions are then sent to the functions as parameters, one after the other (a, b, c, etc).

### Providing Callbacks
```javascript
    var router = Router({

      '/dog': {
        '/angry': {
          on: [growl, freakout]
        }
        on: bark
      },

      '/cat': {
        '/squish': {
          on: freakout
        }
        on: meow
      }

    });
```

Above we have a case where both `/dog/angry` and `cat/squsih` will execute `freakout`. Hence the `on` property will support an array which can execute many functions when there is a URL match.

### Special events

```javascript
    var router = Router({

      '/dog': {
        '/angry': {
          on: 'growl',
          once: 'zap'
        }
        on: bark
      },

      '/cat': {
        '/saton': {
          on: 'freakout'
        }
        on: meow
      }

    });
```

In some cases, you may want to fire a function once. For instance a signin or advertisement is a good use case. In addition to the `on` property there is a `once` property for this purpose.

### More special events

```javascript
    var router = Router({

      '/dog': {
        on: bark
      },

      '/cat': {
        on: meow
        leave: function() {}
      }

      before: function() {},
      leave: function() {},
      notfound: function() {}

    });
```

It is common to need a particular function to fire every time a route is matched, no mater what route it is. In this case `before` can be defined at the top level of the router definition. Similarly, `leave` will be fired when leaving all routs, and `notfound` will be fired when none of the routes can be matched against the user's request.

### Providing some state.

```javascript
    var router = Router({

      '/dog': {
        on: bark,
        state: { needy: true, fetch: 'possibly' }
      },

      '/cat': {
        '/hungry': {
          state: { needy: true, frantic: true }
        },
        on: meow,
        state: { needy: false, fetch: 'unlikely' }
      }

    });
```

It is possible to attach state to any segment of the router, so in our case above if `/dog` is reached, the current state will be set to `{ needy: true, fetch: 'possibly' }`. Each nested section will merge into and overwrite the current state. So in the case where the router matches `/cat/hungry`, the state will become `{ needy: true, fetch: 'unlikely', frantic: true }`.

### Alternate ways to associate functions with routes.

```javascript
    (function() {

      return {

        Main: function() {

          var router = Router({

            '/dog': {
              on: ['bark', 'eat'], // eat and bark.
              '/fat': {
                on: ['eat'] // eat a second time!
              }
            },

            '/cat': {
              on: ['meow', 'eat']
            }

          }, this);
        },

        bark: function() {
          // woof!
        },

        meow: function() {
          // mrrrow!
        },
    
        eat: function() {
          // yum!
        }

      };

    })().Main();
```

Above, a host-object is provided to the router, this provides a way to organize methods which may defined or loaded after the router is configured.

### Using SugarSkull with backbone.
SugarSkull can be a replacement for backbone controllers. 

API
===

### Methods

#### Constructor Methods<br/><br/>

#### `Router(config [, hostObject])` - Returns a new instance of the router.<br/>
@param {Object} config - An object literal representing the router configuration, aka: the routing table.<br/>
@param {Object} hostObject - an object that contains function declarations for later use.<br/>

#### Instance methods<br/><br/>

#### `init()` - Initialize the router, start listening for changes to the URL.<br/><br/>

#### `getState()` - Returns the state object that is relative to the current route.<br/><br/>

#### `getRoute([index])` - Returns the entire route or just a section of it.<br/>
@param {Numner} index - The hash value is divided by forward slashes, each section then has an index, if this is provided, only that section of the route will be returned.<br/>

#### `setRoute(route)` - Set the current route.<br/>
@param {String} route - Supply a route value, such as `home/stats`.<br/>
  
#### `setRoute(start, length)` - Remove from the current route.<br/>
@param {Number} start - The position at which to start removing items.<br/>
@param {Number} length - The number of items to remove from the route.<br/>

#### `setRoute(index, value)` - Set the current route.<br/>
@param {Number} index - The hash value is divided by forward slashes, each section then has an index.<br/>
@param {String} value - The new value to assign the the position indicated by the first parameter.<br/>

`getRetired()` - Returns an array that shows which routes have been retired.

### Events

#### Available on all routes<br/><br/>

#### `on` - A function or array of functions to execute when the route is matched.<br/>
#### `leave` - A function or array of functions to execute when leaving a particular route.<br/>
#### `once` - A function or array of functions to execute only once for a particular route.<br/><br/>

#### Available only at the top level of the router configuration<br/><br/>

#### `before` - A function or array of functions to execute before any route is matched.<br/>
#### `leave` - A function or array of functions to execute when leaving any route.<br/>

# Credits

Author - hij1nx<br/>
Contributors - indexzero, jdalton

# Version
0.2.5

# Licence

(The MIT License)

Copyright (c) 2010 hij1nx <http://www.twitter.com/hij1nx>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
