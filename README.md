# redux-most

[Most.js](https://github.com/cujojs/most) based middleware for Redux.

Handle async actions with monadic streams & reactive programming.

### Instructions

Install `redux-most` and its peer dependencies, `redux` & `most`.

```
npm i -S redux-most redux most
```

### Background

`redux-most` is based on [redux-observable](https://github.com/redux-observable/redux-observable).
It uses the same pattern/concept of "[epics](https://github.com/redux-observable/redux-observable/blob/master/docs/basics/Epics.md)"
without requiring RxJS as a peer dependency. Although `redux-observable` does
provide capability for using other stream libraries via adapters, `redux-most`
allows you to bypass needing to install both RxJS 5 and most. I prefer most for
working with observables and would rather have minimal dependencies. So, I wrote
this middleware primarily for my own use.

Please, see `redux-observable`'s [documentation](https://github.com/redux-observable/redux-observable/blob/master/README.md)
for details on usage.

### Why most over RxJS?

RxJS 5 is great. It's quite a bit faster than RxJS 4, and Rx, in general, is a
very useful tool which happens to exist across many different languages.
Learning it is definitely a good idea. However, most is significantly smaller,
less complicated, and faster than RxJS 5. I prefer its more minimal API and
focus on performance for my own JS projects using reactive streams.

### Why integrate most/RxJS with redux instead of recreating it with streams?

It's true that it's quite easy to implement the core ideas of Redux with
observables using the `scan` operator. (See my [inferno-most-counter-demo](https://github.com/joshburgess/inferno-most-counter-demo)
for an example.) However, the Redux DevTools provide what is arguably the nicest
developer tooling experience currently available in the JavaScript ecosystem.
Therefore, it is huge to be able to maintain it as an asset while still reaping
the benefits of reactive programming with streams. Purists, those who are very
experienced with working with observables, and those working on smaller apps may
not care as much about taking advantage of that tooling as using an elegant
streams-only based solution, and that's fine. The important thing is having a
choice.

### Why `redux-most` or `redux-observable` over `redux-saga`?

`redux-saga` is great. It's a sophisticated approach to handling asynchronous
actions with Redux and can handle very complicated tasks with ease. However,
due to generators being pull-based, it is much more imperative in nature. I
simply prefer the more declarative style of push-based streams & reactive
programming.

### Differences between `redux-most` & `redux-observable`

I chose not to extend the Observable/Stream type with a custom ActionsObservable
type. So, when working with `redux-most`, you will be working with normal most
streams without any special extension methods. However, I have offered something
similar to `redux-observable`'s `ofType` operator in `redux-most` with the
`select` helper function.

Like `ofType`, `select` is a convenience utility for filtering
actions by a specific type. However, `ofType` can optionally take multiple
action types to filter on, whereas `select` only takes a single type. I am not
yet convinced of a great use case for filtering on multiple types. If you have
one, please open an issue and describe it to me.

Additionally, to better align with the most API, `select` can be used in either
a fluent style or a more functional style.


To use the fluent style, just use most's `thru` operator to pass the stream
through to `select` as the 2nd argument.

```js
action$.thru(select(ActionTypes.SEARCHED_USERS_DEBOUNCED))
```

Otherwise, simply directly pass the stream as the 2nd argument.

```js
select(ActionTypes.SEARCHED_USERS_DEBOUNCED, action$)
```

# API Reference

* [createEpicMiddleware](#createepicmiddleware-rootepic)
* [combineEpics](#combineepics-epics)
* [EpicMiddleware](#epicmiddleware)
* [select](#select-actiontype-stream)

<hr>

# `createEpicMiddleware (rootEpic)`

`createEpicMiddleware(rootEpic)` is used to create an instance of the actual redux-most middleware. You provide a single, root Epic.

#### Arguments

1. *`rootEpic: Epic`*: The root Epic

#### Returns

(*`MiddlewareAPI`*): An instance of the redux-most middleware.

#### Example

### redux/configureStore.js

```js
import { createStore, applyMiddleware, compose } from 'redux'
import { createEpicMiddleware } from 'redux-most'
import { rootEpic, rootReducer } from './modules/root'

const epicMiddleware = createEpicMiddleware(rootEpic)

export default function configureStore() {
  const store = createStore(
    rootReducer,
	applyMiddleware(epicMiddleware)
  )

  return store
}
```

<hr>

## `combineEpics (...epics)`

`combineEpics()`, as the name suggests, allows you to take multiple epics and combine them into a single one.

#### Arguments

1. *`...epics: Epic[]`*: The [epics](../basics/Epics.md) to combine.

#### Returns

(*`Epic`*): An Epic that merges the output of every Epic provided and passes along the `ActionsObservable` and redux store as arguments.

#### Example

#### `epics/index.js`

```js
import { combineEpics } from 'redux-most'
import pingEpic from './ping'
import fetchUserEpic from './fetchUser'

export default combineEpics(
  pingEpic,
  fetchUserEpic
)
```

<hr>

## EpicMiddleware

An instance of the redux-most middleware.

To create it, pass your root Epic to [`createEpicMiddleware`](#createepicmiddleware-rootepic).

### EpicMiddleware Methods

- [`replaceEpic (nextEpic)`](#replaceEpic)

<hr>

### <a id='replaceEpic'></a>[`replaceEpic(nextEpic)`](#replaceEpic)

Replaces the epic currently used by the middleware.

It is an advanced API. You might need this if your app implements code splitting and you want to load some of the epics dynamically or you're using hot reloading.

#### Arguments

1. `epic` (*Epic*) The next epic for the middleware to use.

<hr>

## select (actionType, stream)

A helper function for filtering the stream of actions by ActionType

#### Arguments

1. `actionType` The type of action you want to filter by.

2. `stream` The stream of actions you are filtering. Ex: `actions$`

The `select` operator is curried, allowing you to use a `fluent` or `functional`
style.

#### Example: fluent style

```js
import * as ActionTypes from '../ActionTypes'
import { clearSearchResults } from '../actions'
import { select } from 'redux-most'

const clear = action$ =>
  action$.thru(select(ActionTypes.SEARCHED_USERS_DEBOUNCED))
    .filter(action => !action.payload.query)
    .map(clearSearchResults)

export default clear
```

#### Example: functional style

```js
import * as ActionTypes from '../ActionTypes'
import { clearSearchResults } from '../actions'
import { select } from 'redux-most'

const clear = action$ => {
  const search$ = select(ActionTypes.SEARCHED_USERS_DEBOUNCED, action$)
  const emptySearch$ = filter(action => !action.payload.query, search$)
  return map(clearSearchResults, emptySearch$)
}

export default clear
```
