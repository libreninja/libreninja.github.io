---
layout: post
title:  We can transpile it for you wholesale
tagline: Start writing the future of JavaScript today
date:   2016-03-13 20:18:03 -0700
categories: Javascript ES6/ES7
comments: true
---
I have been slowly migrating myself and some of my projects to ES6 of late. Node.js has native support for a [growing list](https://nodejs.org/en/docs/es6/) of new language features, though others are still in progress. Some transpiler magic is required to make everything work like it should.

Things like arrow functions and string literals work right out of the box in node 5.3.x. The new module syntax on the other hand, did not. This post demonstrates how I was able to execute ES6 code modules using 'import' and 'export'.

First, let's create a new project. After that we'll install some dependencies. It makes things easier if babel-cli is installed globally. And make sure you install babel-cli, not babel.
{% highlight bash %}
$ mkdir es6ify
$ cd es6ify
$ npm install -g babel-cli
$ npm install babel-preset-es2015
{% endhighlight %}

Next we create a .babelrc file. The presets are used to tell babel which spec implementation to target.

{% highlight json %}
{
    "presets": [
        "es2015"
    ]
}
{% endhighlight %}


ES6 brings with it the [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) object. But there is no native support for the standard set [operations](https://en.wikipedia.org/wiki/Set_(mathematics)#Basic_operations). That's OK. Don't fret. They're quite simple to implement. For those of you following along, let's go ahead and create some new files.

{% highlight javascript %}
/* file: set-operations.js */
export const union = (a, b) => new Set([...a, ...b])
export const intersection = (a, b) => [...a].reduce((x) => b.has(x))
export const difference = (a, b) => [...a].reduce((x) => !b.has(x))

/*
 * toString isn't really standard. But ES6 makes working with
 * easy. So let's use some template literals while we're here.
 */
export const setString = (s) => `set{ ${[...s].join(', ')} }`
{% endhighlight %}

{% highlight javascript %}
/* file: index.js */
import { union, intersection, difference, setToString } from 'setops'

const log = (strs) => {
    console.log([...strs].join(' '))
}

let s1 = new Set([1, 2, 3, 4, 5])
let s2 = new Set([2, 4, 6, 8, 10])
log(['union(s1, s2) //=>', setString(union(s1, s2))])
log(['intersection(s1, s2) //=>', setString(intersection(s1, s2))])
log(['difference(s1, s2) //=>', setString(difference(s1, s2))])
{% endhighlight %}

Next, fire up a terminal and try running what we have so far.
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

"SyntaxError: Unexpected token import"? That doesn't look right... Because it isn't. We need to transpile our source before node tries to execute it. There are a couple of ways to do that. One way, is to run with 'babel-node', an es6 friendly wrapper for node. Let's try it for ourselves.

{% highlight bash %}
$ babel-node index
s1: set{ 1, 2, 3, 4, 5 }
s2: set{ 2, 4, 6, 8, 10 }
union of s1 and s2: set{ 1, 2, 3, 4, 5, 6, 8, 10 }
intersection of s1 and s2: set{ 2, 4 }
difference of s1 and s2: set{ 1, 3, 5 }
{% endhighlight %}

That works but there is an alternative. We can require 'babel-register' before any source written in ES6 (Don't worry if you've never heard of 'babel-register'. We installed when we installed 'babel-cli'). After node imports 'babel-register', we can start requiring ES6 files.

{% highlight javascript %}
/* file: init.js */
require('babel-register')
require('index')
{% endhighlight %}

Now we can run our ES6 code directly through node.

{% highlight bash %}
$ node init
s1: set{ 1, 2, 3, 4, 5 }
s2: set{ 2, 4, 6, 8, 10 }
union of s1 and s2: set{ 1, 2, 3, 4, 5, 6, 8, 10 }
intersection of s1 and s2: set{ 2, 4 }
difference of s1 and s2: set{ 1, 3, 5 }
{% endhighlight %}

Look how far we've come. From this point, it should be easy enough to automate or customize the process to suit your needs. So I'll wrap up this post and let you get to it. Thanks for reading! Your feedback is appreciated. Please share your thoughts and ideas about this post in the comments below.
