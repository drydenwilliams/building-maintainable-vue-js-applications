# Building maintainable VueJS applications

When building an application in a new framework it's sometimes difficult to get a real world example; escpiecally on a larger scale or real project. 
As we've been busy build an exciting energy reporting app in [VueJS](https://vuejs.org/) that shows breakdowns from your utilities smart meters. 
I thought it would be nice to share somethings we have found out along the way. 

## <a name="table-of-contents"></a>Table of contents

* [Architecture](#architecture)
* [Template Syntax](#template-syntax)
* [Styling](#styling)
* [Vuex](#vuex)
* [Unit Testing](#unit-testing)

## <a id="architecture"></a>Architecture

Within our app we try and keep things as "Vuey" as possible, however along the way one of the biggest changes has been the app architecture. We want to easily navigate between components and different pages or views. Readibility for us is key, both in code quality and architecture. Taken from React, I love the idea of [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0). 

```
src
  assets
   logo.png
  components <= Presentational Components
  views      <= Container Components
  router
    index.js
    routes.js
  utils
    dom.js
  store
    index.js
    mutation-types.js
  App.vue
  main.js
```

Our Presentational and Container Components follow the same structure:

```
components/
  Hello
    test/
      Hello.spec.js
    Hello.html
    Hello.scss
    Hello.vue
    
views/
  Dashboard
    test/
      Dashboard.spec.js
    Dashboard.html
    Dashboard.scss
    Dashboard.vue
```

We found that this structure makes it really easy to traverse our application. Also many of our components have large data visualisation in which makes the overall file sizes sometimes pretty large. We split out our `html` and `scss` into separate files and import them into the main `.vue` component file. 

**Note:** This does require a little tweaking in the webpack config

Here's a little look of the `.vue` component that imports our template and styles:

MyComponent.vue
```js
// Import the html template
<template src="./Hello.html"></template>
// Import scss styles
<style scoped src="./Hello.scss"></style>

<script>
import FooBar from '../FooBar/FooBar'

export default {
    // Component name
    name: 'hello',
    
    // When component uses other components
    components: {
      'foo-bar': FooBar
    },

    // Component props are alphabetised
    props: {
      bar: {},
      foo: {},
      fooBar: {},
    },

    // Component Data
    data () {
      msg: 'Hello World'
    },

    computed: {},

    // Methods
    watch: {},
    methods: {},

    // Component Lifecycle hooks
    created () {},
    ready () {}
};
</script>
```

**Note:** the structure of the component instance properties and methods. We try and keep them in a flowing structure.

## <a id="template-syntax"></a>Template Syntax

HTML attributes should come in this particular order for easier reading of code. This is taken from [Code Guide by Mark Otto](http://codeguide.co/#html-attribute-order)

* `class`
* `id`, `name`
* `data-*`
* `src`, `for`, `type`, `href`, `value`
* `title`, `alt`
* `role`, `aria-*`

Classes make for great reusable components, so they come first. Ids are more specific and should be used sparingly (e.g., for in-page bookmarks), so they come second.

```html
<a class="..." id="..." data-toggle="modal" href="#">
  Example link
</a>
```

We don't have a real preference to using shorthand or full-syntax on Vue-specific attributes but if/when you do just make sure you pop them at the front. When you are conditionally rendering an element it's great to see that first. 

```html
<a v-if="showExampleLink" class="..." id="..." data-toggle="modal" href="#">
  Example link
</a>
```

Sometimes your element might get quite big so don't me scared to drop the attributes on multiple lines:

```html
<a 
  v-if="showExampleLink"
  v-bind:class="{ 'class-a': isA, 'class-b': isB }"
  id="..." 
  data-toggle="modal" 
  href="#"
>
  Example link
</a>
```

## <a id="styling"></a>Styling

Just a short word on styling. We import a main style sheet that follows [The 7 in 1 pattern](https://sass-guidelin.es/#the-7-1-pattern) in to our `App.vue`. It's pretty hand to pattern to keep everything in order. All components import what they need and everything else is inherited globally. We reuse what we can as much as we can. Here's a look at a little `Alert.vue` component:

```css
@import 'colors';
@import 'spacing';

.alert {
    padding: $u-verical;
    text-align: left;
    color: $c-white;
    border-radius: 5px; 
}

.alert--danger {
    background: $col7;
}

.alert--warning {
    background: $col5;
}

.alert--info {
    background: $col4;
}
```

We try and keep things as 'CSS-y' as we can. Nothing super fancy makes it easy to read which is key.


## <a id="vuex"></a>Vuex - Maintaining State

Knowing when to use something like Vuex or any state management can be difficult. I like this answer by [@gaearon on Github](https://github.com/reactjs/redux/issues/1287#issuecomment-175351978):

> Use React for ephemeral state that doesn't matter to the app globally and doesn't mutate in complex ways. For example, a toggle in some UI element, a form input state. Use Redux for state that matters globally or is mutated in complex ways. For example, cached users, or a post draft. Sometimes you'll want to move from Redux state to React state (when storing something in Redux gets awkward) or the other way around (when more components need to have access to some state that used to be local). The rule of thumb is: do whatever is less awkward.

We have recently implimented social login and here's an example of what we would want to Vuex store in our global app state:

```javascript 
import * as types from '../mutation-types'
import * as actions from '../actions/user'

// Initial State
const state = {
  token: null,
  userName: null,
  isAuthenticated: false,
  isAuthenticating: false,
}

const getters = {
  isAuthenticated: (state) => typeof state.token === 'string' && state.isAuthenticated === true
}

const mutations = {
  // Social Login
  [types.OAUTH_LOGIN_REQUEST] (state, payload) {
    state.isAuthenticating = true
  },
  // Social Login Success
  [types.OAUTH_LOGIN_SUCCESS] (state, payload) {
    state.token = payload.token
    state.userName = payload.userName
    state.isAuthenticated = true
    state.isAuthenticating = false
  }
}

export default {
  actions,
  getters,
  mutations,
  state
}
```

We haven't got anything about the login form or the UI in the module. It simply passes the infomation througout our app. Different components can 

## <a id="unit-testing"></a>Unit Testing

I recently started popping all of my tests in to one file. I like the clean output and for us we don't really test our templates too much. Our designs change quite a bit so need to be flexible for this change. We test the login in both unit tests and end-to-end tests. Here's just a silly little example:

CheeseList.spec.js

```javascript
import Vue from 'vue'
import CheeseList from '../CheeseList/CheeseList'

describe('CheeseList', () => {
  /*
   * Template
   *
   */
  describe('Template', () => {
    it('should render a CheeseList component', () => {
      const vm = new Vue(CheeseList).$mount()
      expect(vm.$el).toBeTruthy()
    })
  })
  
  /*
   * Actions
   *
   */
  describe('Actions', () => {
    it('cheesesInStock', (done) => {
      const cheesesPayload = {
        'devon-blue': 10,
        stilton: 2,
        wensleydale: 12
      }
      // Using Jasmine
      spyOn(Vue, 'http').and.returnValue(new Promise((resolve, reject) => {
        return resolve(new Response(JSON.stringify(cheesesPayload), {
          status: 200
        }))
      }))
      testAction(cheeseActions.cheesesInStock, null, {}, [
        { type: 'CHEESE_STOCK_REQUEST', payload: {} },
        { type: 'CHEESE_STOCK_SUCCESS', payload: cheesesPayload }
      ], done)

      expect(Vue.http).toHaveBeenCalled()
    })
  })

  /*
   * Mutations
   *
   */
  describe('Mutations', () => {
    it('CHEESE_STOCK_SUCCESS', () => {
      const state = {
        request: false,
        success: false,
        failure: false,
        cheeses: null
      }
      // apply mutation
      CHEESE_STOCK_SUCCESS(state, {
        'devon-blue': 10,
        stilton: 2,
        wensleydale: 12
      })
      // assert result
      expect(state.request).toBe(false)
      expect(state.success).toBe(true)
      expect(state.cheeses.stilton).toBe(2)
    })
  })
})


```
