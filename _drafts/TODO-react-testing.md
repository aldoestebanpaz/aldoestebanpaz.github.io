# Testing on React

There is a broad spectrum of React component testing techniques. They range from ...
- a "smoke test" just verifying that a component renders without throwing,
- to "shallow rendering" and testing some of the output,
- to full rendering and testing component lifecycle and state changes.

Different projects choose different testing tradeoffs based on how often components change, and how much logic they contain.

If you haven’t decided on a testing strategy yet, it is recommend to start with creating basic "smoke tests" for your components.

## Concepts

### React elements

A React Element is what gets returned from components. It's an object that virtually describes the DOM nodes that a component represents.
- With a function component, this is the object that the function returns.
- With a class component, this is the object that the component's 'render' method returns.

**React elements are not what we see in the browser, they are just objects**.

### Unit testing

Unit testing is done against a single "unit" of code (generally, it means a single class or function). Note the word "unit" could mean different thinks to different people, so in some case you may define "unit" as more than a single class or function (e.g. a component along with a helper class).

#### Testing in React

#### Mocks and type of mocks

A mock is a class that looks like the real class, but we can control what it does, what its methods return, and we can ask it questions about what methods were called during a test.

Mocking allows us to make sure that we are only testing a single unit of code at a time.

Although most people just use the generic term mock, there are actually several types of objects that do various things related to mocking:

- Dummy: Dummies are just objects that fill a place. They generally don't do much interesting, they're just used in place of a real object. We could have a dummy with a method that requires a parameter that's an object, but it doesn't care what that object is.
- Stub: A stub is an object that has controllable behavior. If we call a certain method on a stub we can decide in our test what value that method call will return.
- Spy: A spy is an object that keeps track of which of its methods were called, and how many times they were called, and what parameters were used for each call.
- True mock: These are more complex objects that verify that they were used in exactly a specific way (For example, they can check that only a specific method was called, and that is was called only once, and it had some very specific parameters, and they're able to do this to themselves). They a bit more difficult to work with and are usually overkill for what most unit tests need. This type of object are usually used when testing components that use HTTP.

Most of the time, Dummy, Stub and Spy are the types of objects that we use when we need a mock, and many times the boundaries between these three can be a little blurred (We might use objects that have the behavior of both stubs and spies for example).

#### Isolated testing

This is a basic unit test, the type type of tests that we think of when we think of a unit test.

In an isolated test we simply exercise a single unit of code, it could be a react component, or another a helper file like a service with functions that make network requests or a utils file.

For non-react code (I mean, helper files like a service with network requests or utils functions) we construct that class or function by hand, and we give it its construction parameters ourselves.

Isolated tests are state-based tests, for example, we may need to check that the state of the system under test (sut) had changed.

#### Interaction testing

It is a particular kind of isolated testing where we check that methods were called in the way that we anticipate they should be called.

Interaction tests are tests for checking that some invocations, that doesn't change the state of the object under test, are made. For example, we may need to check that the method of a dependency (like a service) was called with the correct parameter.

#### React Integration testing

In React, Integration tests are an special type of unit tests. These tests uses the component to make sure that the renderization is working as expected.

There are two types of integration tests supported in React:
- Shallow integration tests: where we only test a single component.
- Deep integration tests: used when we want to test a component that has child components and we would like to check that both the parent component and the child component are working together.

##### Smoke tests

This test only mounts a component and makes sure that it didn't throw during rendering. Tests like this provide a lot of value with very little effort so they are great as a starting point.

```js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

it('renders without crashing', () => {
  const div = document.createElement('div');
  ReactDOM.render(<App />, div);
});
```

##### Shallow rendering, a bad practice

Different to shallow testing in Angular, in the world of React development when we say "shallow tests" we implicitly refers to "shallow rendering", a tool provided either by the React's '[Shallow Renderer](https://reactjs.org/docs/shallow-renderer.html)' and by the '[shallow](http://airbnb.io/enzyme/docs/api/shallow.html)' tool from enzyme. This tool renders a component "one level deep", this means that child components remain un-expanded in the rendered output instead of being rendered themselves.

Although it provides the isolation, it has several significant disadvantages:
- It doesn't render actual DOM nodes (jsdom is not used here).
- It doesn't run lifecycle methods (because we just have the React elements to deal with).
- Facebook don't actively use the shallow renderer any more and as a result, it is not being actively developed, although given the volume of existing code that relies on it in the wild, maybe it will not disappear any time soon.

What shallow rendering does is taking the result of a given component's 'render' method (this result is the React element of the component) and giving us another JavaScript object with some utilities for traversing it. This means it doesn't run lifecycle methods, it doesn't allow you to actually interact with DOM elements (because **nothing's actually rendered**), and it doesn't actually attempt to get the react elements (in other words, it doesn't call the 'render' method) of child components.

The important thing to note here is that, **in shallow rendering nothing's actually rendered**.

As an alternative to shallow rendering, we can unit test react components using real renderization with a library like the React Testing Library (but enzyme could be used as well) and Jest mocking.

See [Why I Never Use Shallow Rendering](https://kentcdodds.com/blog/why-i-never-use-shallow-rendering) and [Migrating from shallow rendering react components to explicit component mocks](https://www.youtube.com/watch?v=LHUdxkThTM0&list=PLV5CVI1eNcJgCrPH_e6d57KRUTiDZgs0u) (video) for details.

#### Shallow rendering and snapshots testing

As said before, testing using shallow rendering is not recommended, so don't even try to use it along with other type of tests like Snapshot testing. Even so **Snapshot testing should be rarely used** and the following is a quote of Kent C. Dodds that explains the reason:

```
The whole snapshot is nothing but implementation details (it's full of component and prop names that change all the time on refactors). It'll fail any time you touch the component and the git diff for the snapshot will look almost identical to the one for your changes to the component.

This will make people careless about changes to the snapshot updates because they change all the time. So it's basically worthless (almost worse than no tests because it makes you think you're covered when you're not and you won't write proper tests because they're in place).

I do think that snapshots can be useful though...
```

See [Effective Snapshot Testing](https://kentcdodds.com/blog/effective-snapshot-testing) for details.

#### Static rendering (the enzyme's render function)

TODO

http://airbnb.io/enzyme/docs/api/render.html

### Functional testing (aka. Integration testing)

NOTE: It's somewhat of a vague concept that can mean a lot of things to a lot of different people.

Remember that in React, Integration tests are a special kind of unit test.

Defined as "More than a unit, less than the complete application". This means that at least two units are working together when this type of testing is done (e.g. the testing of a component, or more, and/or a service working together).

These types of test are used to check that a cerrtain part of the application works with another part of the same application.

### Testing of DOM interaction & routing components

TODO

### End to end testing (E2E)

This is a testing that is done against a live, running application (This means the full application with a live DB, live server, live front-end). The benefit of E2E is you can validate that your application works as a whole.

This type of test are generally done through automating the browser (Tests are written to do things like click buttons, type values into forms, navigate around, and similar tasks).

Note that most of the E2E testing tools really have nothing to do with the framework that we are using, so there exists a lot of tools for E2E that are equally applicable for React as they are to any other web app.

Some of the most known tools for E2E are:
- [Cypress](https://www.cypress.io/).
- [Playwright](https://playwright.dev/).
- [Puppeteer](https://pptr.dev/).

## The tools

The following are the tools that are usually used when testing React apps:

- Jest: it is a JavaScript testing framework that includes both a test runner and a assertion and mocking library.
- React Testing Library (RTL): This library encourages your applications to be more accessible and allows you to get your tests closer to using your components the way a user will, which allows your tests to give you more confidence that your application will work when a real user uses it.
- Mock Service Worker (MSW): Mock Service Worker is an API mocking library that uses Service Worker API to intercept actual requests. By bringing the ability of Service Workers to capture requests for the purpose of caching, Mock Service Worker enables API mocking on the highest level of the network communication chain. It is the closest thing to a mocking server without having to create one.

## Storybook, a visual verification tool

Storybook is a tool for UI development that makes faster and easier isolating components. This allows you to work on one component at a time. Also you can develop entire UIs without needing to start up a complex dev stack, force certain data into any database, or navigate around the application under development.

See [https://storybook.js.org/docs/react](https://storybook.js.org/docs/react) and [https://github.com/lauthieb/awesome-storybook](https://github.com/lauthieb/awesome-storybook) for details.

## Rendenring components

When it comes to testing the rendering of React components, we have many choices:
- Using the 'react-test-renderer' package. See [Test Renderer](https://reactjs.org/docs/test-renderer.html) for details.
- Using the 'react-dom/test-utils' package. See [Test Utilities](https://reactjs.org/docs/test-utils.html) for details.
- Using Enzyme. See [Enzyme](https://enzymejs.github.io/enzyme/) for details.
- Using the 'react-testing-library' package. See [React Testing Library](https://testing-library.com/react) for details.

NOTE: both Enzyme and 'react-testing-library' are built upon Test Utilities and Test Renderer.

### Test Utilities (the 'react-dom/test-utils' package)

It gives you the bare minimum to test a React component.

### Test Renderer (the 'react-test-renderer' package)

It is used to render React components to pure JavaScript objects, without depending on the DOM or a native mobile environment. In other words, it can be used to render a component and then make assertions against the React component abstractions (like the 'props' and 'children' objects) instead of using DOM abstractions like DOM elements and attributes.

This package also makes it easy to grab a snapshot of the platform view hierarchy (similar to a DOM tree) rendered by a React DOM or React Native component without using a browser or jsdom.

See [https://reactjs.org/docs/test-renderer.html](https://reactjs.org/docs/test-renderer.html) for details.

### Enzyme

Enzyme is a popular testing library from Airbnb. It provides us with mechanism to test the implementation details of a React component. It can provide us with access to the state, props and children components of a React component while writing test.

It offers custom renderer that doesn't require DOM (for shallow rendering), behaves differently to Test Renderer and allows things that are important for unit testing but aren't possible or straightforward with default renderer, like synchronous state updates, shallow rendering, disabling lifecycle methods, etc.

Note that Enzyme lets us test the component according to the state, props and other internals of the component but it also lets us access the DOM of the component. So using enzyme, we are not bound to test the internals but we can test the DOM too.

### React Testing Library (the 'react-testing-library' package)

Unlike Enzyme, it is not focused on the implementation details of the component, it uses the render logic. **The testing is done from user's perspective and that's why this library doesn't provide us the access to the component's properties such as it's state, props, and component's methods**. The assertions are made from the DOM elements which can be accessed by the utility provided by React Testing Library.

react-testing-library is a library for testing React components in a way that resembles the way the components are used by end users. It is well suited for unit, integration, and end-to-end testing of React components and applications. It works more directly with DOM nodes, and therefore it's recommended to use with jest-dom for improved assertions.

#### Some of the significant differences with Enzyme

Here's the list of things that React Testing Library cannot do (out of the box):

- shallow rendering (like enzyme's '[shallow](http://airbnb.io/enzyme/docs/api/shallow.html)' function).
- Static rendering (like enzyme's '[render](http://airbnb.io/enzyme/docs/api/render.html)' function).
- Pretty much most of enzyme's methods to query elements (like enzyme's '[find](http://airbnb.io/enzyme/docs/api/ReactWrapper/find.html)') which include the ability to find by a component class or even its 'displayName' (again, the user does not care what your component is called and neither should your test). Note: React Testing Library supports querying for elements in ways that encourage accessibility in your components and more maintainable tests.
- Getting a component instance (like enzyme's '[instance](http://airbnb.io/enzyme/docs/api/ReactWrapper/instance.html)').
- Getting and setting a component's props ('props()')
- Getting and setting a component's state ('state()')

All of the things above are things which users of your component cannot do, so the tests with this library don't do them out of the box either.

**The 'render' function is the heart of React Testing Library. It is more similar to Enzyme's 'mount' method**.

##### Using act() and wrapper.update()

When testing asynchronous code in Enzyme, you usually need to call act() to run your tests correctly. When using React Testing Library, you don't need to explicitly call act() most of the time because it wraps API calls with act() by default.

update() syncs the Enzyme component tree snapshot with the React component tree, so you may see wrapper.update() in Enzyme tests. React Testing Library does not have (or need) a similar method, which is good since we need to handle fewer things.

## How to avoid boilerplate in your test files with 'create-react-app'

Note: this feature is available with react-scripts@0.4.0 and higher.

If your app uses a browser API that you need to mock in your tests or if you need a global setup before running your tests, add a 'src/setupTests.js' to your project. It will be automatically executed before running your tests.

For example, suppose you want to use react-testing-library along with jest-dom. First you will need to install them:

```sh
npm install --save-dev @testing-library/react @testing-library/jest-dom
```

then, if you want to avoid boilerplate in your test files, you can create a 'src/setupTests.js' file with the following content:

```
// react-testing-library renders your components to document.body,
// this adds jest-dom's custom assertions
import '@testing-library/jest-dom';
```

### Loading 'src/setupTests.js' in non 'create-react-app'-based projects

In other projects, you have to add the reference to this script in package.json. You should manually create the property setupFilesAfterEnv in the configuration for Jest, something like the following:

```js
"jest": {
  // ...
  "setupFilesAfterEnv": ["<rootDir>/src/setupTests.js"]
}
```
