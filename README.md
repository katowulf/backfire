Backfire
========
Backfire is an officially supported [Backbone](http://backbonejs.org) binding for
[Firebase](http://www.firebase.com/?utm_medium=web&utm_source=backfire).
The bindings let you use special model and collection types that will
automatically synchronize with Firebase, and also allow you to use regular
`Backbone.Sync` based synchronization methods.

Live Demo: <a target="_blank" href="http://firebase.github.io/backfire/examples/todos/">Real-time TODO app</a>.

Getting Started
---------------
Include both firebase.js and backbone-firebase.js in your application.
They're both served off of Firebase's CDN, which you are welcome to use!

### As a Script Include
``` html
<script src="https://cdn.firebase.com/v0/firebase.js"></script>
<script src="https://cdn.firebase.com/libs/backfire/0.3.0/backbone-firebase.min.js"></script>
```

You will now have access to the `Backbone.Firebase`,
`Backbone.Firebase.Collection`, and `Backbone.Firebase.Model` objects.

### Bower Install
``` bash
bower install backfire
```

### Node Install
``` js
var BackboneFirebase = require('./backbone-firebase.js');
```

Pretty much just like web installs; use `BackboneFirebase.Collection`, and `BackboneFirebase.Model`.

Backbone.Firebase
-----------------
The bindings also override `Backbone.sync` to use Firebase. You may consider
this option if you want to maintain an explicit seperation between _local_ and
_remote_ data, and want to use regular Backbone models and collections.

This adapter works very similarly to the
[localStorage adapter](http://documentcloud.github.com/backbone/docs/backbone-localstorage.html)
used in the canonical Todos example.

Please see [todos-sync.js](https://github.com/firebase/backfire/blob/gh-pages/examples/todos/todos-sync.js)
for an example of how to use this feature.

### firebase
You simply provide a `firebase` property in your collection, and that set of
objects will be persisted at that location.

``` js
var TodoList = Backbone.Collection.extend({
  model: Todo,
  firebase: new Backbone.Firebase("https://<your-namespace>.firebaseio.com")
});
```

You can also do this with a model.

``` js
var MyTodo = Backbone.Model.extend({
  firebase: new Backbone.Firebase("https://<your-namespace>.firebaseio.com/myTodo")
});
```

### fetch()
In a collection with the `firebase` property defined, calling `fetch` will
retrieve data from Firebase update the collection with its contents.

``` js
TodoList.fetch();
```

### sync()
In a collection with the `firebase` property defined, calling `sync` will
set the contents of the local collection to the specifeid Firebase location.

``` js
TodoList.sync();
```

### save()
In a model with the `firebase` property defined, calling `save` will set the
contents of the model to the specified Firebase location.

``` js
MyTodo.save();
```

### destroy()
In a model with the `firebase` property defined, calling `destroy` will remove
the contents at the specified Firebase location.

``` js
MyTodo.destroy();
```

Backbone.Firebase.Collection
----------------------------
This is a special collection object that will automatically synchronize its
contents with Firebase. You may extend this object, and must provide a Firebase
URL or a Firebase reference as the `firebase` property.

Each model in the collection will be treated as a `Backbone.Firebase.Model`
(see below).

Please see [todos.js](https://github.com/firebase/backfire/blob/gh-pages/examples/todos/todos.js)
for an example of how to use this special collection object.

```js
var TodoList = Backbone.Firebase.Collection.extend({
  model: Todo,
  firebase: "https://<your-firebase>.firebaseio.com"
});
```

You may also apply a `limit` or some other [query](https://www.firebase.com/docs/queries.html)
on a reference and pass it in:

```js
var Messages = Backbone.Firebase.Collection.extend({
  firebase: new Firebase("https://<your-firebase>.firebaseio.com").limit(10)
});
```
Any models added to the collection will be synchronized to the provided
Firebase. Any other clients using the Backbone binding will also receive
`add`, `remove` and `changed` events on the collection as appropriate.

**BE AWARE!** You do not need to call any functions that will affect _remote_
data. If you call `fetch` or `sync` on the collection,
**the library will ignore it silently**.

``` js
Messages.fetch(); // DOES NOTHING.
Messages.sync();  // DOES NOTHING.
```

You should add and remove your models to the collection as you normally would,
(via `add` and `remove`) and _remote_ data will be instantly updated.
Subsequently, the same events will fire on all your other clients immediately.

### add(model)
Add a new model to the collection. This model will be synchronized to Firebase,
triggering an `add` event both locally and on all other clients.

``` js
Messages.add({subject: "Hello", time: new Date().getTime()});
```

### remove(model)
Removes a model from the collection. This model will also be removed from
Firebase, triggering a `remove` event both locally and on all other clients.

``` js
Messages.remove(someModel);
```

### create(value)
Cretes and adds a new model to the collection. The newly created model is
returned, along with a `id` property (uniquely generated by Firebase).

``` js
var model = Messages.create({bar: "foo"});
Messages.get(model.id);
```

Backbone.Firebase.Model
-----------------------
This is a special model object that will automatically synchronize its
contents with Firebase. You may extend this object, and must provide a Firebase
URL or a Firebase reference as the `firebase` property.

```js
var MyTodo = Backbone.Firebase.Model.extend({
  firebase: "https://<your-firebase>.firebaseio.com/mytodo"
});
```

You may apply limits as with `Backbone.Firebase.Collection`.

**BE AWARE!** You do not need to call any functions that will affect _remote_
data. If you call `save`, `sync` or `fetch` on the model,
**the library will ignore it silently**.

``` js
MyTodo.save();  // DOES NOTHING.
MyTodo.sync();  // DOES NOTHING.
MyTodo.fetch(); // DOES NOTHING.
```

You should modify your model as you normally would, (via `set` and `destroy`)
and _remote_ data will be instantly updated.

### set(value)
Sets the contents of the model and updates it in Firebase.

``` js
MyTodo.set({foo: "bar"}); // Model is instantly updated in Firebase (and other clients).
```

### destroy()
Removes the model locally, and from Firebase.

``` js
MyTodo.destroy(); // Model is instantly removed from Firebase (and other clients).
```
