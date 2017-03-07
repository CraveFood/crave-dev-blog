Here at CraveFood we have a mixed team working remotely and on site and very often we pair using hangouts to help each other, until one day… it happened.

My coworker was using a legacy machine and doing TDD on his computer was quite impossible, from one change to another we've to wait more than 30s and this could kill our workflow very easily by saying that writing tests slowed us down (and it's true).

When we started this project we picked the most used tools by that time: Karma, PhantomJS, Mocha, Webpack and Isparta.

We tried to use JSDOM but it just didn't work with react and we've spent a lot of time building our environment and we had to move on.

It was slow, but I got used to it, from one change to another on my machine it took 3s (well, for TDD this is really slow!), but I've argued with myself that I had to run karma, phantomjs and then transpiling our code using webpack and that takes time… hell yeah!

And maybe that's the problem, when we stay in the same place for so long we get blind, we just can't see that the world has changed (huuhuh). When I decided to learn Elixir in the last year, it was punch in my face, because for newcomers things are not that easy and those 3s weren't a problem for me anymore.

New year, new promises… the time had come to dive into the world of the front end tooling again, and my first guess was to search something related to slow tests with karma and webpack and boom:

I found this [article](https://medium.com/@TomazZaman/how-to-get-fast-unit-tests-with-out-webpack-793c408a076f#.r27618ai2) from Tomaz and he was saying to take off webpack, karma and phantom from the game. (WTF?!)

I ignored him, of course! hahahaha

I spent more time searching and then I read this article from Vitality having the same feelings. He proposed to compile webpack creating a hook with webpack-shell-plugin to call mocha after the compilation was completed.

I tried, it failed!

So I reached an article written by Ole Michelsen proposing a similar approach from Vitality. This time without webpack.

Mocking loaders from webpack using **require.extension**:

```javascript
const noop = () => {}; 
require.extensions[‘.css’] = noop; 
require.extensions[‘.ico’] = noop;
require.extensions[‘.png’] = noop; 
require.extensions[‘.svg’] = noop;
```

Almost there, My Dear Watson!

But life ain't easy. We were using webpack alias, css modules and other stuff. We need something more!!

Mocking .css files would break tests using classNames. How do we solve that?

**[mock-css-modules](https://www.npmjs.com/package/mock-css-modules)**

It is awesome because it returns to the original class name!!!

The second part was to solve the problems with webpack alias, so I found this awesome package called **[babel-plugin-wepack-alias](https://www.npmjs.com/package/babel-plugin-webpack-alias).**

But again, we hit another wall. If you are using **[inject-loader](inject-loader)** or **[proxyquire](https://github.com/thlorenz/proxyquire)**, this won't work because the mapped alias from babel-plugin-webpack will be a different path and proxyquire and inject-loader won't be able to locate them. So what do we do? Cry! (I'm Joking!)

Erik Ardal suggested in his **[article](https://eirikardal.no/mocking-babel-modules/)** the usage of sinon.stub for module mocking. It has some downsides.

```javascript
// a.js
export function hello() {}

// BAD
// b.js
import {hello} from './a';

const greet = hello();

export function wolrd() {
 return greet;
}

// GOOD
// b.js
import {hello} from ‘./a’;

export function wolrd() {
 const greet = hello();
 return greet;
}
```

We could try jest, because it mocks all modules for you, but it didn't come to my mind.

What about code coverage? It was pretty straight forward. Just drop **[babel-plugin-istanbul](https://github.com/istanbuljs/babel-plugin-istanbul)** and **[nyc](https://github.com/istanbuljs/nyc)** to your project and you are good to go!

And now, some numbers of our entire suite test running without watch mode:

**PhantomJS, Karma and Webpack:**
**real 1m17.745s**
user 1m19.767s
sys 0m7.583s

**JSDOM, Mocha, Babel:**
**real 0m23.180s**
user 0m17.785s
sys 0m1.800s

YAY, much faster!!!

So here is our project setup:

```javascript
// package.json
“test”: “./node_modules/.bin/_mocha”
“coverage”: “BABEL_ENV=test nyc ./node_modules/.bin/_mocha”

// test/mocha.opts
 — compilers js:babel-register
 — require jsdom-global/register
 — require test/setup.js
src/**/*.spec.js

// test/setup.js
var mockCssModules = require('mock-css-modules');
mockCssModules.register(['.sass', '.scss', '.css']);

const noop = () => {};

require.extensions['.ico'] = noop;
require.extensions['.png'] = noop;
require.extensions['.svg'] = noop;
require.extensions['.jpg'] = noop;

// .babelrc
{

  "env": {
    "test": {
      "presets": ["es2015", "stage-0", "react"],
      "plugins": [
        [ "babel-plugin-webpack-alias", { "config": "./test/webpack.config.test.js" } ],
        ["istanbul", {
          "exclude": [
            "**/*.spec.js"
          ],
          "useInlineSourceMaps": false
         }]
       ]
    }
  }
}

// test/webpack.config.test.js
var path = require('path');

var config = {
  resolve: {
    alias: {
      something: find('actions')
    }
  }
};

function find(alias) {
  return path.join(__dirname, '..', 'src', alias);
}

module.exports = config;
```

And that's all!! Many thanks to Tomaz, Ole and Erik who inspired me!

Thank you!
