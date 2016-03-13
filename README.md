JavaScript的不可变集合
====================================

[![Build Status](https://travis-ci.org/facebook/immutable-js.svg)](https://travis-ci.org/facebook/immutable-js)

[Immutable][] 数据一旦创建就不可能被改变, 这样可以让应用开发更加简单，这使得无脑复制、启用高效的内存和变化的检测技术逻辑变得简单. [Persistent][]数据呈现了一个不更新原数据，但是反而总是产出新的更新过的数据的可变的API .

Immutable.js 提供了很多牢固的不可变数据结构，包括:
`List`, `Stack`, `Map`, `OrderedMap`, `Set`, `OrderedSet` and `Record`.

通过Clojure和Scala的普及，这些数据结构是在现代的JavaScript虚拟机高效
通过结构性共享[hash maps tries] []和[vector tries] []
，最大限度地减少复制或缓存数据.

`Immutable` 也提供了一个懒的 `Seq`, 允许方法集合的高效链接，如：`map` 和 `filter`不需要创建中间交涉。创建一些带有`Range` 和 `Repeat` 的 `Seq`.

[Persistent]: http://en.wikipedia.org/wiki/Persistent_data_structure
[Immutable]: http://en.wikipedia.org/wiki/Immutable_object
[hash maps tries]: http://en.wikipedia.org/wiki/Hash_array_mapped_trie
[vector tries]: http://hypirion.com/musings/understanding-persistent-vector-pt-1


Getting started
---------------

用npm安装 `immutable` .

```shell
npm install immutable
```

然后可以在任何模块引入

```javascript
var Immutable = require('immutable');
var map1 = Immutable.Map({a:1, b:2, c:3});
var map2 = map1.set('b', 50);
map1.get('b'); // 2
map2.get('b'); // 50
```

### 浏览器端

从浏览器使用 `immutable` , 下载 [dist/immutable.min.js](https://github.com/facebook/immutable-js/blob/master/dist/immutable.min.js)
或用 CDN 如 [CDNJS](https://cdnjs.com/libraries/immutable)
或 [jsDelivr](http://www.jsdelivr.com/#!immutable.js).

然后，在你的页面里的 script 标签加入:

```html
<script src="immutable.min.js"></script>
<script>
    var map1 = Immutable.Map({a:1, b:2, c:3});
    var map2 = map1.set('b', 50);
    map1.get('b'); // 2
    map2.get('b'); // 50
</script>
```

或者使用 AMD 加载器(比如 [RequireJS](http://requirejs.org/)):

```javascript
require(['./immutable.min.js'], function (Immutable) {
    var map1 = Immutable.Map({a:1, b:2, c:3});
    var map2 = map1.set('b', 50);
    map1.get('b'); // 2
    map2.get('b'); // 50
});
```

如果你使用 [browserify](http://browserify.org/),  `immutable` npm 模块
也可以在浏览器端工作.

### TypeScript

使用这些不可变集合和序列就像你在你的本地使用 [TypeScript](http://typescriptlang.org)  本地集合一样。一样的会在IDE里有各种特性。

只要在文件的顶部添加带有相对路径的类型声明引用即可

```javascript
///<reference path='./node_modules/immutable/dist/immutable.d.ts'/>
import Immutable = require('immutable');
var map1: Immutable.Map<string, number>;
map1 = Immutable.Map({a:1, b:2, c:3});
var map2 = map1.set('b', 50);
map1.get('b'); // 2
map2.get('b'); // 50
```


不变性的场景
-------------------------

令应用开发者觉得困难的最多的情况是跟踪变化和维持状态。带着可变的数据去开发的时候会刺激你去思考数据如何在你的应用中流动。

在你的应用中到处订阅数据事件会导致很糟糕性能问题, and creates
opportunities for areas of your application to get out of sync with each other
due to easy to make programmer error. Since immutable data never changes,
subscribing to changes throughout the model is a dead-end and new data can only
ever be passed from above.

这个数据结构可以很好的兼容 [React][] 的结构，在使用 [Flux][] 的思想的应用中搭配尤其棒

当数据从上面穿过而不是被订阅，当有东西变化时，你只需要关注你的工作而不是去订阅他，你可以一样使用

不变集合应该像 *values* 一样被对待而不是 *objects*. While
objects represents some thing which could change over time, a value represents
the state of that thing at a particular instance of time. This principle is most
important to understanding the appropriate use of immutable data. In order to
treat Immutable.js collections as values, it's important to use the
`Immutable.is()` function or `.equals()` method to determine value equality
instead of the `===` operator which determines object reference identity.

```javascript
var map1 = Immutable.Map({a:1, b:2, c:3});
var map2 = map1.set('b', 2);
assert(map1.equals(map2) === true);
var map3 = map1.set('b', 50);
assert(map1.equals(map3) === false);
```

注意：有一个性能优化的点，就是当一个操作的结果是一个相同的集合时， `Immutable` 尝试去返回一个已经存在的集合，用 `===`  参考去确定是不是没有被改变。
 This can be extremely useful when used within memoization function
which would prefer to re-run the function if a deeper equality check could
potentially be more costly. The `===` equality check is also used internally by
`Immutable.is` and `.equals()` as a performance optimization.

如果一个对象是不可变的，他可以对另一个对象引用来被简单的拷贝，而不是复制整个对象。因为一个引用对于对象来说是很轻量的，结果在内存中并且潜在的促进了那些依赖副本的程序的运行速度（such as an undo-stack）.

```javascript
var map1 = Immutable.Map({a:1, b:2, c:3});
var clone = map1;
```

[React]: http://facebook.github.io/react/
[Flux]: http://facebook.github.io/flux/docs/overview.html


JavaScript-first API
--------------------

当时 `immutable` 是受到 Clojure, Scala, Haskell 和其他的函数式编程语言环境启发,目的是把那些强大的概念带到 JavaScript 里来，所以拥有一个面向对象的API，和 [ES6][] [Array][], [Map][], [Set][] 类似。

[ES6]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/New_in_JavaScript/ECMAScript_6_support_in_Mozilla
[Array]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array
[Map]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map
[Set]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set

对于不可变对象不同的是，改变集合的那些方法如 `push`, `set`, `unshift` 或 `splice`是返回一个新的不可变集合。那些返回新数组的方法像： `slice` 或 `concat` 也将返回的是一个新的不可变集合

```javascript
var list1 = Immutable.List.of(1, 2);
var list2 = list1.push(3, 4, 5);
var list3 = list2.unshift(0);
var list4 = list1.concat(list2, list3);
assert(list1.size === 2);
assert(list2.size === 5);
assert(list3.size === 6);
assert(list4.size === 13);
assert(list4.get(0) === 1);
```

几乎所有在 [Array][] 的方法都和 `Immutable.List` 的形式很类似， [Map][] 和 `Immutable.Map`， [Set][] 和 `Immutable.Set`，包括集合操作像`forEach()` 和 `map()`

```javascript
var alpha = Immutable.Map({a:1, b:2, c:3, d:4});
alpha.map((v, k) => k.toUpperCase()).join();
// 'A,B,C,D'
```

### 接受原生js对象

为了和你现有的js互相操作， `immutable` 在任何地方都接受纯js数组和对象，这些方法不会有性能问题

```javascript
var map1 = Immutable.Map({a:1, b:2, c:3, d:4});
var map2 = Immutable.Map({c:10, a:20, t:30});
var obj = {d:100, o:200, g:300};
var map3 = map1.merge(map2, obj);
// Map { a: 20, b: 2, c: 10, d: 100, t: 30, o: 200, g: 300 }
```

因为 `immutable` 可以对待任何可迭代的数组和对象。这样你就可以使用一些很复杂的对象方法，而不是使用以前那些稀稀拉拉的原生的API. 因为他是懒执行而不在中间层缓存，所以这些操作会非常高效。

```javascript
var myObject = {a:1,b:2,c:3};
Immutable.Seq(myObject).map(x => x * x).toObject();
// { a: 1, b: 4, c: 9 }
```

要记住的是，当用js对象去构造一个 Immutable Maps 时，这js对象的属性将一直是字符串类型，even if written in a quote-less shorthand，然而 Immutable Maps 接受任何类型的keys

```js
var obj = { 1: "one" };
Object.keys(obj); // [ "1" ]
obj["1"]; // "one"
obj[1];   // "one"

var map = Immutable.fromJS(obj);
map.get("1"); // "one"
map.get(1);   // undefined
```

Property access for JavaScript Objects first converts the key to a string, but
since Immutable Map keys can be of any type the argument to `get()` is
not altered.


### 转换回原生js对象

所有的可迭代的 `immutable` 都可以被浅转换成纯js数组和对象，利用 `toArray()` 和 `toObject()` 或者使用 `toJS()` 深转换
所有的 Immutable Iterables 也可以直接通过`JSON.stringify` 实现 `toJSON()`

```javascript
var deep = Immutable.Map({ a: 1, b: 2, c: Immutable.List.of(3, 4, 5) });
deep.toObject() // { a: 1, b: 2, c: List [ 3, 4, 5 ] }
deep.toArray() // [ 1, 2, List [ 3, 4, 5 ] ]
deep.toJS() // { a: 1, b: 2, c: [ 3, 4, 5 ] }
JSON.stringify(deep) // '{"a":1,"b":2,"c":[3,4,5]}'
```

### 拥抱 ES6

`Immutable` 充分利用了 JavaScript 中的 [ES6][] 的特性，最新的ECMAScript稳定版本中，包括了[Iterators][],[Arrow Functions][], [Classes][], 和 [Modules][]。他的 [Map][] 和 [Set][] 集合也被es6启发。这个库  "transpiled" es3，是为了去支持所有的现代浏览器.

所有的例子是用es6去呈现的，为了运行在所有的浏览器中，他需要被转换成es3

```js
// ES6
foo.map(x => x * x);
// ES3
foo.map(function (x) { return x * x; });
```

[Iterators]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/The_Iterator_protocol
[Arrow Functions]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions
[Classes]: http://wiki.ecmascript.org/doku.php?id=strawman:maximally_minimal_classes
[Modules]: http://www.2ality.com/2014/09/es6-modules-final.html


Nested Structures
-----------------

The collections in `immutable` are intended to be nested, allowing for deep
trees of data, similar to JSON.

```javascript
var nested = Immutable.fromJS({a:{b:{c:[3,4,5]}}});
// Map { a: Map { b: Map { c: List [ 3, 4, 5 ] } } }
```

A few power-tools allow for reading and operating on nested data. The
most useful are `mergeDeep`, `getIn`, `setIn`, and `updateIn`, found on `List`,
`Map` and `OrderedMap`.

```javascript
var nested2 = nested.mergeDeep({a:{b:{d:6}}});
// Map { a: Map { b: Map { c: List [ 3, 4, 5 ], d: 6 } } }
```

```javascript
nested2.getIn(['a', 'b', 'd']); // 6

var nested3 = nested2.updateIn(['a', 'b', 'd'], value => value + 1);
// Map { a: Map { b: Map { c: List [ 3, 4, 5 ], d: 7 } } }

var nested4 = nested3.updateIn(['a', 'b', 'c'], list => list.push(6));
// Map { a: Map { b: Map { c: List [ 3, 4, 5, 6 ], d: 7 } } }
```


Lazy Seq
--------

`Seq` describes a lazy operation, allowing them to efficiently chain
use of all the Iterable methods (such as `map` and `filter`).

**Seq is immutable** — Once a Seq is created, it cannot be
changed, appended to, rearranged or otherwise modified. Instead, any mutative
method called on a Seq will return a new Seq.

**Seq is lazy** — Seq does as little work as necessary to respond to any
method call.

For example, the following does not perform any work, because the resulting
Seq is never used:

    var oddSquares = Immutable.Seq.of(1,2,3,4,5,6,7,8)
      .filter(x => x % 2).map(x => x * x);

Once the Seq is used, it performs only the work necessary. In this
example, no intermediate arrays are ever created, filter is called three times,
and map is only called twice:

    console.log(oddSquares.get(1)); // 9

Any collection can be converted to a lazy Seq with `.toSeq()`.

    var seq = Immutable.Map({a:1, b:1, c:1}).toSeq();

Seq allow for the efficient chaining of sequence operations, especially when
converting to a different concrete type (such as to a JS object):

    seq.flip().map(key => key.toUpperCase()).flip().toObject();
    // Map { A: 1, B: 1, C: 1 }

As well as expressing logic that would otherwise seem memory-limited:

    Immutable.Range(1, Infinity)
      .skip(1000)
      .map(n => -n)
      .filter(n => n % 2 === 0)
      .take(2)
      .reduce((r, n) => r * n, 1);
    // 1006008

Note: An iterable is always iterated in the same order, however that order may
not always be well defined, as is the case for the `Map`.


Equality treats Collections as Data
-----------------------------------

`Immutable` provides equality which treats immutable data structures as pure
data, performing a deep equality check if necessary.

```javascript
var map1 = Immutable.Map({a:1, b:1, c:1});
var map2 = Immutable.Map({a:1, b:1, c:1});
assert(map1 !== map2); // two different instances
assert(Immutable.is(map1, map2)); // have equivalent values
assert(map1.equals(map2)); // alternatively use the equals method
```

`Immutable.is()` uses the same measure of equality as [Object.is][]
including if both are immutable and all keys and values are equal
using the same measure of equality.

[Object.is]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is


Batching Mutations
------------------

> If a tree falls in the woods, does it make a sound?
>
> If a pure function mutates some local data in order to produce an immutable
> return value, is that ok?
>
> — Rich Hickey, Clojure

Applying a mutation to create a new immutable object results in some overhead,
which can add up to a minor performance penalty. If you need to apply a series
of mutations locally before returning, `Immutable` gives you the ability to
create a temporary mutable (transient) copy of a collection and apply a batch of
mutations in a performant manner by using `withMutations`. In fact, this is
exactly how  `Immutable` applies complex mutations itself.

As an example, building `list2` results in the creation of 1, not 3, new
immutable Lists.

```javascript
var list1 = Immutable.List.of(1,2,3);
var list2 = list1.withMutations(function (list) {
  list.push(4).push(5).push(6);
});
assert(list1.size === 3);
assert(list2.size === 6);
```

Note: `immutable` also provides `asMutable` and `asImmutable`, but only
encourages their use when `withMutations` will not suffice. Use caution to not
return a mutable copy, which could result in undesired behavior.

*Important!*: Only a select few methods can be used in `withMutations` including
`set`, `push` and `pop`. These methods can be applied directly against a
persistent data-structure where other methods like `map`, `filter`, `sort`,
and `splice` will always return new immutable data-structures and never mutate
a mutable collection.


Documentation
-------------

[Read the docs](http://facebook.github.io/immutable-js/docs/) and eat your vegetables.

Docs are automatically generated from [Immutable.d.ts](https://github.com/facebook/immutable-js/blob/master/type-definitions/Immutable.d.ts).
Please contribute!

Also, don't miss the [Wiki](https://github.com/facebook/immutable-js/wiki) which
contains articles on specific topics. Can't find something? Open an [issue](https://github.com/facebook/immutable-js/issues).


Contribution
------------

Use [Github issues](https://github.com/facebook/immutable-js/issues) for requests.

We actively welcome pull requests, learn how to [contribute](./CONTRIBUTING.md).


Changelog
---------

Changes are tracked as [Github releases](https://github.com/facebook/immutable-js/releases).


Thanks
------

[Phil Bagwell](https://www.youtube.com/watch?v=K2NYwP90bNs), for his inspiration
and research in persistent data structures.

[Hugh Jackson](https://github.com/hughfdjackson/), for providing the npm package
name. If you're looking for his unsupported package, see [v1.4.1](https://www.npmjs.org/package/immutable/1.4.1).


License
-------

`Immutable` is [BSD-licensed](./LICENSE). We also provide an additional [patent grant](./PATENTS).
