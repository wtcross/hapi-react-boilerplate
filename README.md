# What is it?
A boilerplate project for a [Hapi](http://hapijs.com)-powered web/api server serving a [Webpack](http://webpack.js.org)-built React/Redux app.

## What it includes
* Project hierarchy and structure
* React/Webpack build process
* React, Redux, ReactRouter, ReduxActions
* Zero-config testing suite for server and client
* Auto-generated SwaggerUI (when using Hapi route conventions)
* Sensible Webpack config
* Base HapiJS manifest

## What it doesn't include
* Database/ORM choices/libraries/operations - intentionally left out

## Gotchas
* Node still doesn't handle `import` well, so you'll need to use `require` on the server side and `import` on the client side.

# Dev
* `yarn install`
* `yarn dev` - wait a sec for webpack to build, then go to `localhost:8000`

# Usage as a project template:
* Run from project root: `rm -rf .git && rm -rf ./server/tasks && rm -rf ./client/src/store/actions/tasksAdd && rm -rf ./client/src/store/reducers/tasks && rm client/src/store/index.js && mv ./client/src/store/index_blank.js ./client/src/store/index.js`
* remove `{ plugin: './server/tasks' }` line from `manifest.js`
* Gut `client/src/RouteHome`

# JSON API/File server

### Plugins
Create a plugin for each logical grouping of features for the application and register them in the server manifest. A plugin lives at the top level of the `/server` directory and has an `index.js` file that exports a registration function. The rest of the folder structure is up to the requirements of the app, but it's recommended to do something similar to this:

```
/server
  /tasks
    /db - database operations for tasks
      tasksSchemas.js
      tasksDB.js
      tasksDB.test.js
      index.js - optional convenience exports
    /handlers -
      tasksHandlers.js
      index.js - optional convenience exports
    /responses
      taskResponses.js - Joi schemas for route JSON responses
    /routes
      taskRoutes.js
      taskRoutes.test.js
    index.js
```

# React App

### Components
The React/Redux app has only two main top-level directories in `src`: `components` and `store`. For components, No hierarchical distinction is made to distinguish "containers", presentational components, connected components, route components, etc. Instead, reflect this with the name of the component itself.

* One top-level `Root` component should wrap all necessary `Provider`s, routers, and initializer components
* A high level `App` component may be used to render a layout or other components that organize Routes. This would be rendered inside of the providers/initializers of the `Root` component's `render` method.
* For components loaded directly from React Router, prefix their name with `Route`. A component that represents a single Task routed view, for instance, would be named `RouteTask`.
* For components that are mapped over for display in lists, prefix with `Item`
* For components that are used to initialize behavior or do business-logicy things on App load, prefix with `Initializer` and insert into the main `App` component.

### Actions and Reducers
Actions, Reducers, and any other pure function transformations should live in their own directories under `/src/store`. Reducers should be namespaced and organized by what portion of the state they are modifying.

#### Directory-based Components and Actions
Each Component and Action has a directory named after it, containing an `index.js` file for all of the relevant code and a `test.js` containing tests. No name or namespace distinction is made for actions vs async actions.

## Redux
### Defining action types
The [`store/actionTypes.js`](./client/src/store/actionTypes.js) file takes an array of strings and exports them as an object with a matching key. This is simpler and quicker to modify than exporting a new constant for every action type. Just `import types from 'store/actionTypes'` and access your action name from the `types` object.

### redux-actions
The [redux-actions](https://github.com/acdlite/redux-actions) library is used to reduce redundancy while creating actions and action handlers.

#### Creating actions
When defining actions in their corresponding index files, always export a `createAction` function with a null `payloadCreator` function (so that the default is used.)

```js
export default createAction(types.TASKS_ADD)
```
same as
```js
const tasksAdd = task => ({
  type: types.TASKS_ADD,
  payload: task
})
export default tasksAdd
```

#### Using actions
Use the `handleActions` function from `redux-actions` in each reducer to assign a handler function to an action type:

```js
export default handleActions({
  [types.TASKS_ADD]: addTask, // defined elsewhere
  [types.TASKS_SET]: (state, action) => {...state, tasks: action.payload},
}, initialState)
```

### Connected Components
* Use a named function `mapStateToProps`, defined immediately before the `export`, to...map state to props.
* Use the second `connect` argument (`mapDispatchToProps`) with a destructured object with your imported action names. This allows you to call `tasksAdd(whatever)` instead of having to pass it to dispatch like `this.props.dispatch(tasksAdd(whatever))`, and it's easier to reason about.

# TODO
* [ ] Example Smoke tests for React Components
* [ ] Example tests for API routes
* [ ] Docs on generating Swagger Docs
* [ ] HMR
* [ ] Basic Auth Plugin example
* [x] Documentation for React/Redux practices
