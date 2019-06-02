# operator
1.8kb drop-in "PJAX" solution for fluid, smooth transitions between pages. Zero stress.

## Features
1. Advanced routing via [matchit](https://github.com/lukeed/matchit)
2. Client-side redirects
3. Async-by-default, easy data loading between routes
4. Pages are cached after initial visit
5. Pre-fetch select pages, as needed

## Install
```bash
npm i operator --save
```

# Usage
Basically zero config by default, just specify a root DOM node to attach to.
```javascript
import operator from 'operator'

operator('#root')
```

## Defining routes
To define custom handlers for a given route, pass an object with a `path`
property and `handler` method.
```javascript
operator('#root', [
  {
    path: '/',
    handler (state) {
      console.log(state)
    }
  }
])
```

Routes handlers can also return `Promise`s, and they support params, optional
params, and wildcards.
```javascript
operator('#root', [
  {
    path: '/',
    handler (state) {
      console.log(state)
    }
  },
  {
    path: '/products',
    handler (state) {
      return getProducts() // Promise
    }
  },
  {
    path: '/products/:category/:slug?',
    handler ({ params }) {
      const reqs = [ getProductCategory(params.category) ]
      if (params.slug) reqs.push(getProductBySlug(params.slug))
      return Promise.all(reqs)
    }
  }
])
```

### Route Caching
Routes are cached by default, so on subsequent visits, no data will be loaded.
To follow links to pages via AJAX, but fetch fresh content on each navigation
action, set `cache` to `false`:
```javascript
operator('#root', [
  {
    'path': '/',
    cache: false
  }
])
```

### Ignoring Routes
Sometimes you need to navigate to a page without AJAX, perhaps to load some sort
of `iframe` content. To do so, set `ignore` to `true`:
```javascript
operator('#root', [
  {
    'path': '/',
    ignore: true
  }
])
```

## Lifecycle
Any function passed to the route config will be called on every route change,
kind of like *middleware*.
```javascript
const app = operator('#root', [
  state => console.log(state)
])
```

Operator also emits some helpful events.
```javascript
app.on('navigate', state => {}) // on valid link click
app.on('before', state => {}) // before render
app.on('after', state => {}) // after render
```

### History state
Operator does not manage `History` or page title, for maximum flexibility to the
user. Most people should probably just use this snippet:
```javascript
app.on('after', ({ title, pathname }) => {
  document.title = title
  window.history.pushState({}, '', pathname)
})
```

# API
### go(path)
```javascript
app.go('/about')
```

### load(path)
Use this for prefetching pages.
```javascript
app.load('/about')
```

### state (getter)
```javascript
app.state // => { title, pathname, location, params, hash, search, handler }
```

# Recipes
### Redirects
```javascript
app.on('before', ({ pathname }) => {
  if (/redirect/.test(pathname)) {
    app.go('/') // redirect
  }
})
```

### Transition animation
```javascript
import wait from 'w2t'

operator('#root', [
  state => {
    return wait(600, [
      const root = document.documentElement.classList
      return new Promise(res => {
        root.add('is-transitioning')
        setTimeout(() => {
          root.remove('is-transitioning')
          res()
        }, 600)
      })
    ])
  }
])
```

# Changelog
### v1.2.0
Slight update to the API, will require brief migration to new syntax for most
users.
- deprecated Array format for route configs in favor of more flexible Object
  syntax
- add `ignore` and `cache` options

## License
MIT License © [Eric Bailey](https://estrattonbailey.com)
