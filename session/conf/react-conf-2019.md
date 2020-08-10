## Building The New Facebook With React and Relay

* build modern design system, support themeing
* improved accessibility, component library out of the box
* faster page loads
* seamless interactions

redesign is not enough -> need to rebuild -> SPA -> React GraphQL Relay

hidden topic: CSS -> built a CSS-in-JS library of Facebook's own

priorities when building this library:
* readability and maintainability
* themeability

it is inspired by React Native and other frameworks

use css variables to build themes instead of using Context API

img tags -> inline svg icons, take svg files from designer, and generate svg icon components from them. change theme and replace icons without extra downloads.

px -> rem to improve accessibility for different screen sizes. use library to transform px -> rem. code in px, send browser rem.

type checks: find typo in styles, highlight unused styles, avoid cross-browser pitfalls, since now styles are now integrated into javascript.

runtime checks: built a tool to highlight components lack of accessibility on pages.

composability -> library break styles object into atomic css classes, and compose them.

### faster page loads

decrease time to visually complete
traditional way:

time to paint(show mockup maybe?)

time to first meaningful paint(show the real contents?)

unused resources: data-driven code-splitting by Relay

improve loading experience: nested React.Suspense

## Using hooks and codegen to bring the benefits of GraphQL to REST APIs

operational UI: generate documentation from markdown files with code blocks. can also edit the code blocks on pages.

give description with qualification, eg. "REST is bad, because...", or "GraphQL is good, because...".

REST is bad, because:
* it is not a formal specification
* leads to a lot of guessing
* meaningless discussions -> users don't really care about the status code
* no contractual agreements

GraphQL is good, because: it has everything opposite to above things that REST has.

[restful-react](https://github.com/contiamo/restful-react): use hooks to fetch data. can [generate types](https://github.com/contiamo/restful-react#code-generation) from OpenAPI or Swagger files.

## Data Fetching With Suspense In Relay

building app with Relay and Suspense

what problem are we solving? -> build a great loading experience

some render patterns:

1. fetch on render: component mounts -> do data fetching, render spinner -> data loaded -> render more components -> a lot of spinners...
2. fetch then render: fetch all the data and render whole things -> Relay can aggregate the dependencies of multiple components and build a single query to fetch all the data
3. render as you fetch

1 & 2 are two extremes, what we really want in practices, is incrementally render ui when data becomes really. That's what React Suspense is about: allow showing some of a UI without waiting for all of it to be ready.

How to achieve this type of incrementally loading with Relay and Suspense?

1. fetching simple data

```jsx
function Composer(props) {
  const data = useQuery(graphql`
    query ComposerQuery {
      me {
        photo {
          uri
        }
      }
    }
  `, variables);

  return (
    <div>
      <img src={data.me.photo.uri} alt={...} />
    </div>
  )
}
```
```jsx
function Home() {
  return (
    <ErrorBoundary fallback={<ErrorMessage />}>
      <Suspense fallback={<Placeholder />}>
        <Composer />
      </Suspense>
    </ErrorBoundary>
  )
}
// ErrorBoundary -> user defined component using error hook/life cycle method
```
loading sequence:

renders                | render Home | render fallback | render Home |                    | show image

requests download code |             | fetch data      |             | load profile image |

2. multiple fetches
```jsx
function Home() {
  return (
    <ErrorBoundary fallback={<ErrorMessage />}>
      <Suspense fallback={<Placeholder />}>
        <Composer />
      </Suspense>
    + <Suspense>
    +   <NewsFeed fallback={<Placeholder />}/>
    + </Suspense>
    </ErrorBoundary>
  )
}
```

loading sequence:(skip, too complex to draw with text)

key point is that React is able to know queries in sibling components, and can send them both during the parent component's rendering. However, the images might show up disordered.

3. Suspense ordering: waiting for images
```jsx
function Home() {
  return (
    <ErrorBoundary fallback={<ErrorMessage />}>
      + <SuspenseList revealOrder="forward">
          <Suspense fallback={<Placeholder />}>
            <Composer />
          </Suspense>
          <Suspense>
            <NewsFeed fallback={<Placeholder />}/>
          </Suspense>
      + </SuspenseList>
    </ErrorBoundary>
  )
}
```
4. page transitions
```jsx
const reference = preloadQuery(query, variables);
const data = usePreloadedQuery(query, reference); // used in component

const NewsFeedEntryPoint = { // component & data it uses described together
  root: JSResource('NewsFeed.react'),

  getPreloadProps(params) {
    return {
      queryReference: query(
        require('NewsFeedQuery.graphql'),
        { id: params.id },
      )
    };
  },
}

const { queryReference } = prepareEntryPoint(
  NewsFeedEntryPoint,
  routeParams,
);
```
differences between implementing `useQuery` and `preloadQuery`(skip)

benefits over `useQuery` style: simpler to implement, more predictable, more efficient(parallelize code/data loading)

best practices:

* render as fetch:
  * start loading resources before rendering
  * suspense if resources aren't ready when render
* use Suspense at key points for staged loading
* use SuspenseList for in-order resolution of lists/blocks of elements
* use Suspense for code, data, assets, etc.

some new features not covered coming to OSS soon(skip)

## [Building a Custom React Renderer](https://www.youtube.com/watch?v=CGpMlWVcHok&list=PLPxbbTqCLbGHPxZpw4xj_Wwg8-fdNxJRh&index=7)

How to use React Reconciler to build a custom renderer for your host environment, say browser?

This talk give some practices of building your own version of react-dom in a hundred of code.

react-reconciler has two modes: mutation mode & persistent mode. react-dom uses the former mode cuz that is how DOM API works, and the latter mode is used when your target is an environment where the API treats tree of view as a list of immutable objects.

```js
import ReactReconciler from 'react-reconciler';

const reconciler = ReactReconciler({
  // config for how to talk to host env
  supportsMutation: true,

  // tell react how to create node with attributes, event handlers, etc. in your host env
  createInstance(
    type,
    props,
    rootContainerInstance,
    hostContext,
    internalInstanceHandle,
  ) {
    const el = document.createElement(type);
    if (props.className) el.className = props.className;
    if (props.onClick) el.addEventListener('click', props.onClick);
    return el;
  },
  createTextInstance() {},

  // tell react how to attach nodes
  appendChildToContainer(){},
  appendChild() {},
  appendInitialChild() {},

  // tell react how to detach nodes
  removeChildFromContainer() {},
  removeChild() {},
  insertInContainerBefore() {},
  insertBefore() {},

  // tell react how to update view
  prepareUpdate(
    instance,
    type,
    oldProps,
    newProps,
    rootContainerInstance,
    currentHostContext,
  ) {
    let updatePayload = {};
    // compare props
    return updatePayload;
  },
  commitUpdate(
    instance,
    updatePayload,
    type,
    oldProps,
    newProps,
    finishedWork,
  ) {
    // update node properties based on updatePayload
  },

  // other things...
});

const ReactDOM = {
  render(whatToRender, div) {
    const container = reconciler.createContainer(div, false, false);
    reconciler.updateContainer(whatToRender, container, null, null);
  },
};

export default ReactDOM;
```

things did not mention: form controls, controlled and un-controlled components, refs, event normalization cross browsers, svg, event bubbling, portals, etc.

## [React Developer Tooling](https://www.youtube.com/watch?v=Mjrfb1r3XEM&list=PLPxbbTqCLbGHPxZpw4xj_Wwg8-fdNxJRh&index=17)

What is fast refresh? make code changes, auto update component sometimes even without resetting its previous state. It's different from hot reloading. -> already supported in React Native, has an issue tracking status for react-dom.

How is Fast Refresh different?
* first class feature
* built for all renderers
* supports function components and hooks
* recovers after errors

How does Fast Refresh work?
* when to 'update' vs 'remount'
* computes a component 'signature'(say, a string describes what hooks are used and the used sequence)
* remount when signature changes

Limitations?
* resets class component state all the time
* mixed react and non-react exports gets remount all the time
* memorization like using memo may be called more frequently than it should be

rewrites React DevTools: react-devtools-tutorial.now.sh

[codemods](https://github.com/reactjs/react-codemod) helps update React API in existed projects.

## [Building (And Re-Building) the Airbnb Design System](https://www.youtube.com/watch?v=fHQ1WSx41CA&list=PLPxbbTqCLbGHPxZpw4xj_Wwg8-fdNxJRh&index=13)

what is a design system?

evolution of their design system

-> facing issues: fragmentation complexity performance
-> want a design system that helps to resolve, which may contain two parts: components & styles
-> why css-in-js: css tightly coupled to components, can use tree-shaking to only ship critical css, themeing, prevent style overrides
-> design language system in 2016
-> design language system for 2020

modular architecture: a system of independent components that can be composed together -> not quite helpful, cuz add a props is so easy

base & variants pattern: create base component which is not tied with design system, can be easily extended, and implement variants which does not allow overrides.

some trade-offs of these patterns(skip)

constrained system
* the system is static
* the system is small
* have limited design support
* not the only enforcers of consistency

flexible system
* the system is evolving
* the system is large
* a strong design system
* no enforcement of consistency

## Let's Program Like It's 1999

illustrate below process through stories at Facebook across three problems: rendering views, managing state, and fetching data.

loops: abstraction -> clear syntax API -> more refined mental models -> better abstractions...

building newsfeed with memcache led to the dreaded N+1 problem -> build a tool called Preparable -> async functions & GraphQL & Relay

LAMP = Linux + Apache web server(support CGI) + MySQL server + PHP = Free & Open Web stack

## Is React Translated yet?

interesting talk about how React 's documentation got translated to different languages.

a schedule tool from UNIX: https://www.npmjs.com/package/cron


## Automatic Visualizations of the Frontend

a talk about tools(9+) that automatic visualizations

observable notebook -> reactive js notebook
