---
layout: post
title:  We can transpile it for you wholesale
tagline: Start writing the future of JavaScript today
date:   2016-03-13 20:18:03 -0700
categories: Javascript ES6/ES7
comments: true
---
I have been brushing up my ES6 of late; and I wanted to migrate some current projects over to the new work flow.  Node.js has native support for a [growing list](https://nodejs.org/en/docs/es6/) of new language features, though others are still in progress.  Some transpiler magic is required to make everything work like it should.

Syntactic sugar features like arrow functions and string literals work right out of the box in node 5.3.x.  The new module syntax on the other hand, does not.  This post demonstrates how I was able to implement and consume ES6 code modules using 'import' and 'export'.

First, let's create a new project.  Install some dependencies while we're there.  Install babel-cli globally if you can.  And make sure to install the 'babel-cli' package from npm, not the predecessor 'babel'.
{% highlight bash %}
$ mkdir es6ify
$ cd es6ify
$ npm install -g babel-cli
$ npm install babel-preset-es2015
{% endhighlight %}

Next we create a .babelrc file.  Presets defined here are enable babel plugins.  "es2015" tells babel how to parse the source for the ES6 specification.

{% highlight json %}
{
    "presets": [
        "es2015"
    ]
}
{% endhighlight %}

What are we going to do with our new syntax powers anyway?  Well, ES6 brings with it the [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) object.  But there is no native support for the standard set [operations](https://en.wikipedia.org/wiki/Set_(mathematics)#Basic_operations).  Fortunately they're quite simple to implement.  So let's write some shiny new ecmascript files.

{% highlight javascript %}
/* file: set-operations.js */
export const union = (a, b) => new Set([...a, ...b])
export const intersection = (a, b) => [...a].filter((x) => b.has(x))
export const difference = (a, b) => [...a].filter((x) => !b.has(x))

/*
 * setString is not standard. But ES6 makes string manipulation
 * easy.  So let's write some handy template literals.
 */
export const setString = (s) => `set{ ${[...s].join(', ')} }`
{% endhighlight %}

{% highlight javascript %}
/* file: index.js */
import { union, intersection, difference, setToString } from 'set-operations'

// we'll need 2 Sets
let s1 = new Set([1, 2, 3, 4, 5])
let s2 = new Set([2, 4, 6, 8, 10])

// and a tagged template literal for logging
const log = (strs) => {
    console.log([...strs].join(' '))
}

// put it all together now
log(['s1:', setString(s1)])
log(['s2:', setString(s2)])
log(['union(s1, s2) //=>', setString(union(s1, s2))])
log(['intersection(s1, s2) //=>', setString(intersection(s1, s2))])
log(['difference(s1, s2) //=>', setString(difference(s1, s2))])
{% endhighlight %}

Next we'll fire up a terminal and try running what happens with what we have done so far.

{% highlight bash %}
$ node index
/User/me/projects/es6ify/index.js:1
(function (exports, require, module, __filename, __dirname) { import { union, intersection, difference, setString } from './lib/operations'
                                                              ^^^^^^

SyntaxError: Unexpected token import
    at exports.runInThisContext (vm.js:53:16)
    at Module._compile (module.js:387:25)
    at Object.Module._extensions..js (module.js:422:10)
    at Module.load (module.js:357:32)
    at Function.Module._load (module.js:314:12)
    at Function.Module.runMain (module.js:447:10)
    at startup (node.js:140:18)
    at node.js:1001:3
{% endhighlight %}

"SyntaxError: Unexpected token import"!?!  That can't be right.  It turns out that we need to transpile our source ahead of time.  There are a few ways to get this done.  One way is to run our code with 'babel-node', an es6 friendly wrapper for node.  See it in action below.

{% highlight bash %}
$ babel-node index
s1: set{ 1, 2, 3, 4, 5 }
s2: set{ 2, 4, 6, 8, 10 }
union of s1 and s2: set{ 1, 2, 3, 4, 5, 6, 8, 10 }
intersection of s1 and s2: set{ 2, 4 }
difference of s1 and s2: set{ 1, 3, 5 }
{% endhighlight %}

That works fine, but there is another just-in-time transpiling mode.  Simply require the 'babel-register' package before including any ES6 syntax.  After that, we can use our ecma-futuristic modules with import and export syntax.  Here is a new bootstrap file to set it all up.

{% highlight javascript %}
/* file: bootstrap.js */
require('babel-register')
require('./index')
{% endhighlight %}

Now we can run our ES6 code directly through node.

{% highlight bash %}
$ node bootstrap
s1: set{ 1, 2, 3, 4, 5 }
s2: set{ 2, 4, 6, 8, 10 }
union of s1 and s2: set{ 1, 2, 3, 4, 5, 6, 8, 10 }
intersection of s1 and s2: set{ 2, 4 }
difference of s1 and s2: set{ 1, 3, 5 }
{% endhighlight %}

Finally, we can have babel write transpiled files out to disk and run them in ordinary vanilla javascript node environments.  This is useful when you want to package the code for deployment environments where you might not want to transpile on the fly.  We can create a 'dist' folder in our project.  And then tell babel to write the output files there.

{% highlight bash %}
$ mkdir dist
$ babel *.js -d dist/.
bootstrap.js -> dist/bootstrap.js
index.js -> dist/index.js
set-operations.js -> dist/set-operations.js
$ node dist/index
s1: set{ 1, 2, 3, 4, 5 }
s2: set{ 2, 4, 6, 8, 10 }
union(s1, s2) //=> set{ 1, 2, 3, 4, 5, 6, 8, 10 }
intersection(s1, s2) //=> set{ 2, 4 }
difference(s1, s2) //=> set{ 1, 3, 5 }
{% endhighlight %}

Just look at how far we've come.  We are living in the future, today.  At this point, it should be easy enough to automate or customize this process to suit your needs.  So I'll wrap up this post and let you get to it.  Your feedback is appreciated.  Please share your thoughts and ideas in the comments below.  Thanks for reading!
