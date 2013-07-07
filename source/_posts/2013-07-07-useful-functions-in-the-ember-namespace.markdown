---
layout: post
title: "Useful Functions in the Ember Namespace"
date: 2013-07-07 10:28
comments: true
categories:
- EmberJS
---

There are a lot of really excellent functions baked right into the Ember. Here are some really useful ones that may come in handy and save you a few keystrokes. Many of the examples below are taken from the [Ember API docs](http://emberjs.com/api/) and a few are taken from the [Discourse source code](https://github.com/discourse/discourse).

## Ember.assert(desc, test)

Define an assertion that will throw an exception if the condition is not met. *Ember build tools will remove any calls to `Ember.assert()` when doing a production build.*

```javascript
Ember.assert("should pass", 1 === 1) //=> undefined
Ember.assert("should fail", 1 === 2) //=> Assertion failed: should fail

// You can force a failed assertion by not passing the boolean argument
Ember.assert("force fail")           //=> Assertion failed: force fail
```

[http://emberjs.com/api/#method_assert](http://emberjs.com/api/#method_assert)

## Ember.computed.SOMETHING

You can easily create computed properties with the computed property helper functions:

<small style="float: right;">[http://emberjs.com/api/#method_computed_and](http://emberjs.com/api/#method_computed_and)</small>
#### and(dependentKey,)

Performs the logical *and* on the dependentKeys provided and returns a boolean if all resolve to true.

```javascript
showMessageInput: Ember.computed.and('is_custom_flag', 'selected')
```
<small style="float: right;">[http://emberjs.com/api/#method_computed_any](http://emberjs.com/api/#method_computed_any)</small>
#### any(dependentKey,)

Returns the first truthy value of a given list of properties.

```javascript
isAuthorized: Ember.computed.any('user_id', 'auth_token')
```

<small style="float: right;">[http://emberjs.com/api/#method_computed_bool](http://emberjs.com/api/#method_computed_bool)</small>
#### bool(dependentKey)

Convert to boolean the original value for property.

```javascript
isSorted: Ember.computed.bool('sortProperties')
```

#### empty(dependentKey)

Negate the original value for property.

```javascript
sendTestEmailDisabled: Ember.computed.empty('testEmailAddress')
```

<small style="float: right;">[http://emberjs.com/api/#method_computed_equal](http://emberjs.com/api/#method_computed_equal)</small>
#### equal(dependentKey, value)

Computed property which returns true if the original value for property is equal to the given value.

```javascript
markdown: Ember.computed.equal('format', 'markdown'),
plainText: Ember.computed.equal('format', 'plain'),
html: Ember.computed.equal('format', 'html'),
css: Ember.computed.equal('format', 'css')
```

<small style="float: right;">[http://emberjs.com/api/#method_computed_gt](http://emberjs.com/api/#method_computed_gt)</small>
#### gt(dependentKey, value)

Computed property which returns true if the original value for property is greater than the given value.

```javascript
showContinueButton: Ember.computed.gt('minimumSongsSelected', 'songCount')
```

<small style="float: right;">[http://emberjs.com/api/#method_computed_gte](http://emberjs.com/api/#method_computed_gte)</small>
#### gte(dependentKey, value)

Computed property which returns true if the original value for property is greater than or equal to the given value.

```javascript
isOldEnough: Ember.computed.gte('minimumAge', 'age')
```


<small style="float: right;">[http://emberjs.com/api/#method_computed_lt](http://emberjs.com/api/#method_computed_lt)</small>
#### lt(dependentKey, value)

Computed property which returns true if the original value for property is less than the given value.

```javascript
needMoreSongs: Ember.computed.lt('minimumSongsSelected', 'songCount')
```

<small style="float: right;">[http://emberjs.com/api/#method_computed_lte](http://emberjs.com/api/#method_computed_lte)</small>
#### lte(dependentKey, value)

Computed property which returns true if the original value for property is less than or equal to the given value.

```javascript
showContinueButton: Ember.computed.lte('maximumFavoriteGenres', 'chosenGenreCount')
```

<small style="float: right;">[http://emberjs.com/api/#method_computed_map](http://emberjs.com/api/#method_computed_map)</small>
#### map(dependentKey,)

Computed property which maps values of all passed properties in to an array.

<a class="jsbin-embed" href="http://jsbin.com/ixapom/1/embed?javascript">JS Bin</a><script src="http://static.jsbin.com/js/embed.js"></script>

<small style="float: right;">[http://emberjs.com/api/#method_computed_match](http://emberjs.com/api/#method_computed_match)</small>
#### match(dependentKey, regExp)

Computed property which match the original value for property against a given RegExp.

<a class="jsbin-embed" href="http://jsbin.com/esovay/3/embed?javascript">JS Bin</a><script src="http://static.jsbin.com/js/embed.js"></script>

<small style="float: right;">[http://emberjs.com/api/#method_computed_none](http://emberjs.com/api/#method_computed_none)</small>
#### computed.none(dependentKey)

Computed property which returns true if original value for property is null or undefined.

```javascript
showInstructions: Ember.computed.none('email')
```

<small style="float: right;">[http://emberjs.com/api/#method_computed_not](http://emberjs.com/api/#method_computed_not)</small>
#### computed.not(dependentKey)

```javascript
submitDisabled: Ember.computed.not('submitEnabled')
```

<small style="float: right;">[http://emberjs.com/api/#method_computed_notEmpty](http://emberjs.com/api/#method_computed_notEmpty)</small>
#### computed.notEmpty(dependentKey)

Computed property which returns true if original value for property is not empty.

```javascript
visible: Ember.computed.notEmpty('controller.buffer')
```

<small style="float: right;">[http://emberjs.com/api/#method_computed_oneWay](http://emberjs.com/api/#method_computed_oneWay)</small>
#### computed.oneWay

Computed property which creates an one way computed property to the original value for property. Where `computed.alias` aliases `get` and `set`, and allows for bidirectional data flow, `computed.oneWay` only provides an aliased `get`. The `set` will not mutate the upstream property, rather causes the current property to become the value set. This causes the downstream property to permentantly diverge from the upstream property.

```javascript
User = Ember.Object.extend({ 
  firstName: null, 
  lastName: null, 
  nickName: Ember.computed.oneWay('firstName') 
}); 

user = User.create({ firstName: 'Teddy', lastName: 'Zeenny' }); 
user.get('nickName');              //=> 'Teddy' 
user.set('nickName', 'TeddyBear'); //=> 'TeddyBear' 
user.get('firstName');             //=> 'Teddy'
```

<small style="float: right;">[http://emberjs.com/api/#method_computed_or](http://emberjs.com/api/#method_computed_or)</small>
#### computed.or

Computed property which peforms a logical `or` on the values of all the original values for properties.

```javascript
showManagerTools: Ember.computed.and('is_admin', 'is_manager')
```

## Ember.debug(message)

Display a debug notice. Ember build tools will remove any calls to Ember.debug() when doing a production build.

```javascript
Ember.debug("I'm a debug notice!");
```

[http://emberjs.com/api/#method_debug](http://emberjs.com/api/#method_debug)

## Ember.destroy(obj)

Tears down the meta on an object so that it can be garbage collected. Multiple calls will have no effect.

```javascript
Ember.destroy(session)
```

[http://emberjs.com/api/#method_destroy](http://emberjs.com/api/#method_destroy)

## Ember.isArray(obj)

Returns true if the passed object is an array or Array-like.

```javascript
Ember.isArray();                                            // false
Ember.isArray([]);                                          // true
Ember.isArray( Ember.ArrayProxy.create({ content: [] }) );  // true
```

[http://emberjs.com/api/#method_isArray](http://emberjs.com/api/#method_isArray)

## Ember.isEmpty(obj)

Verifies that a value is null or an empty string, empty array, or empty function.

```javascript
Ember.isEmpty();                // true
Ember.isEmpty(null);            // true
Ember.isEmpty(undefined);       // true
Ember.isEmpty('');              // true
Ember.isEmpty([]);              // true
Ember.isEmpty('Adam Hawkins');  // false
Ember.isEmpty([0,1,2]);         // false
```

[http://emberjs.com/api/#method_isEmpty](http://emberjs.com/api/#method_isEmpty)

## Ember.isEqual(a, b)

Compares two objects, returning true if they are logically equal. This is a deeper comparison than a simple triple equal. For sets it will compare the internal objects. For any other object that implements isEqual() it will respect that method.

```javascript
Ember.isEqual('hello', 'hello');  // true
Ember.isEqual(1, 2);              // false
Ember.isEqual([4,2], [4,2]);      // false
```

[http://emberjs.com/api/#method_isEqual](http://emberjs.com/api/#method_isEqual)

## Ember.isNone(obj)

Returns true if the passed value is null or undefined. This avoids errors from JSLint complaining about use of ==, which can be technically confusing.

```javascript
Ember.isNone();              // true
Ember.isNone(null);          // true
Ember.isNone(undefined);     // true
Ember.isNone('');            // false
Ember.isNone([]);            // false
Ember.isNone(function(){});  // false
```

[http://emberjs.com/api/#method_isNone](http://emberjs.com/api/#method_isNone)

## Ember.makeArray(obj)

Forces the passed object to be part of an array. If the object is already an array or array-like, returns the object. Otherwise adds the object to an array. If obj is null or undefined, returns an empty array.

```javascript
Ember.makeArray();                           // []
Ember.makeArray(null);                       // []
Ember.makeArray(undefined);                  // []
Ember.makeArray('lindsay');                  // ['lindsay']
Ember.makeArray([1,2,42]);                   // [1,2,42]

var controller = Ember.ArrayProxy.create({ content: [] });
Ember.makeArray(controller) === controller;  // true
```

[http://emberjs.com/api/#method_makeArray](http://emberjs.com/api/#method_makeArray)

## Ember.tryInvoke(obj, methodName, args)

Checks to see if the methodName exists on the obj, and if it does, invokes it with the arguments passed.

```javascript
Ember.trySet('model', 'deliverWelcomeEmail', 'cavneb@gmail.com')
```

[http://emberjs.com/api/#method_tryInvoke](http://emberjs.com/api/#method_tryInvoke)

## Ember.trySet(obj, methodName, args)

Error-tolerant form of Ember.set. Will not blow up if any part of the chain is undefined, null, or destroyed.

This is primarily used when syncing bindings, which may try to update after an object has been destroyed.

```javascript
Ember.trySet('model', 'firstName', 'Eric')
```

[http://emberjs.com/api/#method_trySet](http://emberjs.com/api/#method_trySet)
