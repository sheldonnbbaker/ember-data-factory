Ember Data Factory
===============

Factory library for Ember Data.  Allows you to define and create factories instead of using fixtures.  It is best used with the ember-testing package.

Why use it?
-------------
Aside from the advantages of using factories instead of fixtures, by using Ember Data Factory in your tests, you will be able to create tests that are independent of your adapter.  You can write tests using factories and run them with fixture adapter, and later switch to another adapter (such as RestAdapter, LocalStorageAdapter) to test integration with a specific backend.

Defining factories
--------------------
You should provide a name and default attributes to every factory definition.

```javascript
Factory.define('post', {
  title: 'Post Title',
  body: 'This is the content'
});
```
This will define a factory for the `App.Post` model.

Using factories
------------------

Given the factory definition:
```javascript
Factory.define('post', {
  title: 'Post Title',
  body: 'Post body'
});
```
There are three ember-testing helpers you can use: `attr`, `build`, and `create`

`attr` helper returns an object containing the attributes.

```javascript
var postAttributes = attr('post'); // { title: 'Post Title', body: 'Post body' }
```

`build` helper creates a model instance but does not commit.  This helper is async and therefore a promise.  It can be chained with any other ember-testing helper.

```javascript
build('post').then(function(post) {
  post instanceof App.Post; // true
  post.get('isNew'); // true
  post.get('title'); // Post Title
});
```

`create` helper creates a model same as `build`, but also commits it.  This helper is also async and therefore is a promise and can be chained to any other async helper.

```javascript
create('post').then(function(post) {
  post instanceof App.Post; // true
  post.get('isNew'); // false
  post.get('title'); // Post Title
});
```

You can pass custom properties to any of these helpers.  Passed properties will overwrite the default ones.

```javascript
attr('post', { title: 'My Post'} ); // { title: 'My Post', body: 'Post body' }
```

Factories and Related Records
------------------------------------

If the model belongs to a parent model, you can define the parent attributes in the factory definition (or creation).

```javascript
Factory.define('post', {
  title: 'My Post'
});

Factory.define('comment', {
  title: 'My Comment'
  post: {}
});

create('comment').then(function(comment) {
  comment.get('post.title'); // 'My Post'
});

```

You can also override the attributes for a specific relation:

```javascript
Factory.define('comment', {
  title: 'My Comment',
  post: {
    title: "My Comment's Post"
  }
});

create('comment', { post: { body: 'Custom Body' } }).then(function(comment) {
  comment.get('title'); // "My Comment"
  comment.get('post.title'); // "My Comment's Post'
  comment.get('post.body'); // "Custom Body"
});
```

Ember-testing Example
---------------------------

```javascript

Factory.define('author', {
  name: 'Teddy'
});

Factory.define('post', {
  title: 'My Post',
  author: {}
});

test("Editing a post", function() {
   create('post')
  .visit('/posts')
  .click('.post-title')
  .click('.edit')
  .fillIn('.txt-title', 'New Title')
  .click('.submit')
  .then(function() {
    equal(find('.post-title').text(), 'New Title');
    equal(find('.author-name').text(), 'Teddy');
  });
});
```