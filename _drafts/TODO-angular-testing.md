# Testing on Angular

Always try to follow the principles:
- Never trust a test until you have seen it fail. A valuable test fails when essential code is changed or deleted. To make sure that it is working, you could make it fail by negating the main sentence temporarely.
- Make it self-explanatory. A valuable test clearly describes how the implementation code should behave.
- Follow the AAA pattern.
- Don't waste time testing third party components or services and always assume they work correctly by mocking them.

## Concepts

### Some abbreviations

- sut: short for "system under test".
-  a-a-a test structure: it suggests that you should divide your test method into three sections: arrange (the initialization part, it means acomplishing necessary preconditions and inputs), act (do some kind of action on the object or class under test) and assert (the expect statements to make sure the expected results have occured).

### Unit testing

Unit testing is done against a single "unit" of code (generally, it means a single class). Note the word "unit" could mean different thinks to different people, so in some case you may define "unit" as more than a single class (e.g. a component along with a helper class).

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

In an isolated test we simply exercise a single unit of code, either the class of a component, or the class of a service, or a pipe, we construct that class by hand, and we give it its construction parameters ourselves.

Isolated tests are state-based tests, for example, we may need to check that the state of the component had changed.

#### Interaction testing

It is a particular kind of isolated testing where we check that methods were called in the way that we anticipate they should be called.

Interaction tests are tests for checking that some invocations, that doesn't change the state of the object under test, are made. For example, we may need to check that the method of a dependency (like a service) was called with the correct parameter.

#### Angular Integration testing

In Angular, Integration tests are an special type of unit tests. These tests uses the component and his template together to make sure that the two pieces are working correctly together.

In an Angular integration test we actually create a module in which we put just the code that we're going to test, generally just one component, but we actually test that in the context of an Angular module. This is used so that we can test the component with its template.

There are two types of integration tests supported in Angular:
- Shallow integration tests: where we only test a single component.
- Deep integration tests: used when we want to test a component that has child components and we would like to check that both the parent component and the child component are working together.

### Functional testing (aka. Integration testing)

NOTE: It's somewhat of a vague concept that can mean a lot of things to a lot of different people.

Remember that in Angular, Integration tests are a special kind of unit test.

Defined as "More than a unit, less than the complete application". This means that at least two units are working together when this type of testing is done (e.g. the testing of a component and a service working together).

These types of test are used to check that a cerrtain part of the application works with another part of the same application.

### Testing of DOM interaction & routing components

TODO

### End to end testing (E2E)

This is a testing that is done against a live, running application (This means the full application with a live DB, live server, live front-end). The benefit of E2E is you can validate that your application works as a whole.

This type of test are generally done through automating the browser (Tests are written to do things like click buttons, type values into forms, navigate around, and similar tasks).

Note that most of the E2E testing tools really have nothing to do with the framework that we are using, so there exists a lot of tools for E2E that are equally applicable for Angular as they are to any other web framework.

Some of the most known tools for E2E are:
- [Protractor](https://www.protractortest.org/) plus a supported BDD test framework like Jasmine, Mocha or [cucumber](https://cucumber.io/). See [Choosing a Framework](https://www.protractortest.org/#/frameworks) for details.
- [Cypress](https://www.cypress.io/).

### TDD (Test first)

TODO

## The tools

The tools that are included in a default angular application (without needing of installing them manually) and that are usually used when testing it:

- Karma: the test runner, this is what actually executes our tests in a browser.
- Jasmine: it is the tool we use to create mocks, and it's the tool that we use to make sure that the tests work the way that we want them to using expectations.
- TestBed: is a class and the main testing tool provided by Angular for unit tests (specially for integration tests). This tool allows us to test both a component and its template running together.

There are quite a few other unit testing tools that are available for unit testing with Angular:

- Jest: it's really popular with other frameworks but can be used with Angular too.
- Mock Service Worker (MSW): Mock Service Worker is an API mocking library that uses Service Worker API to intercept actual requests. By bringing the ability of Service Workers to capture requests for the purpose of caching, Mock Service Worker enables API mocking on the highest level of the network communication chain. It is the closest thing to a mocking server without having to create one.
- Mocha and Chai: those are replacements for Jasmine. They are somewhat popular and they're easy to drop in and replace Jasmine with.
- Sinon: it is a specialized mocking library. If we find that the mocking capabilities in Jasmine aren't good enough then we can use something like Sinon.
- TestDouble: a competitor to Sinon, that's gaining some popularity, but is still far less popular than Sinon.
- Wallaby: this is a paid tool that allows you to see the code coverage of your tests right in your IDE. It's very convenient and it's getting to be a popular tool.
- Cypress: is traditionally considered to be an end-to-end testing tool, but they are developing capabilities to do more integration types of testing, so in the future we may see Cypress become more popular with Angular.
- Other E2E tools: there are tons of end-to-end testing tools (e.g. protactor, selenium, cypress), it is important to know that if you are going to write end-to-end tests with Angular you have a lot of choices.
- Storybook: It is a visual verification tool that allows the isolation of a component for testing interactively the UI renderization and functionality.

## The meaning of "spec" on test filenames

NOTE: `.spec.ts` is important, that's what lets our unit testing tool Karma know that this is a test file.

"spec" is short for specification, it's a common word used when writing unit tests, so **we need to make sure that all of our unit tests end with `.spec.ts`**.

It's customary with tests to write the spec file using the same exact name as the file to test, but replacing the word `.ts` by `.spec.ts` in the end of the name of the test file. And it's also customary to put it into the same directory. That way it's easy to see whether or not a piece of code has a test for it, and if not we can then write tests for it if we wish.

## Errors on Karma

Note that sometimes errors don't show up, but they will show up in the browser console of the window with the Karma results. And indeed, if we were to go and look at our test output in the console, we could see warnings.

## Jasmine functions

A `.spec.ts` constains a 'describe' function, this is a Jasmine function that lets us group tests together.

The 'describe' function has two parameters. The first one is a string (which is kind of a descriptor of this part of our tests) and the second one is s callback function (that will contain our tests).

Note that we can nest multiple levels of 'describe's.

The 'beforeEach' function inside 'describe' does a common setup that's going to run before every test. You need to put code inside this function that resets the state so that you make sure that with every test you don't have any effects from a previous test that's holding over and perhaps polluting the state of future tests.

Each 'it' function inside 'describe' is a test. Just like the 'describe' prototype, it takes two parameters, the first is a string that describes the test, and the second that is a callback (usually called the "it statement") where we finally write our test.

For the string on the 'it' function, it's very customary to start with the word "should". The reason for this is that when we see the output of our tests we're going to see the string of the test that either passes or fails, and that's going to be appended with any 'describe's that that 'it' function sits inside of. **It's a really good idea when writing your tests to make sure that these strings (from 'describe' and 'it') when appended together make sense**.

The 'expect' statement is a Jasmine function that allow us to create assertions based in the parameter it receives. Any method chained after the 'expect' function invocation is called a 'matcher'. There exists many matchers (such as toBe, to be truthy, to be greater than or equal to). See [Matchers that come with Jasmine out of the box](https://jasmine.github.io/api/4.3/matchers.html) for the list.

Again, learning Jasmine is not the point of this course, the point of this course is to learn to test Angular. Now this whole test, as it stands right now,

### A silly example

The following is a little bit of a silly example since all we do is create a property on an object, initialize it to one value, set it to another value, and then assert that it is the second value. This is the core essence of what a test is. We get our initial state, we change that state, and then we assert that the new state is what we expect it to be.

```ts
// file: first-test.spec.ts

describe('my first test', () => {
    let sut: any;

    beforeEach(() => {
        sut = {}
    });

    // For this example, this string will show up next to our failing notification:
    // """
    // my first test
    //   should be true if true
    // """
    it('should be true if true', () => {
        // arrange
        sut.a = false;

        // act
        sut.a = true;

        // assert
        expect(sut.a).toBe(true);
    });
});
```

We can execute this test by running `ng test`, `npm test` or `ng test --include='**/first-test.spec.ts'`.

## Storybook, a visual verification tool

Storybook is a tool for UI development that makes faster and easier isolating components. This allows you to work on one component at a time. Also you can develop entire UIs without needing to start up a complex dev stack, force certain data into any database, or navigate around the application under development.

See [https://storybook.js.org/docs/angular](https://storybook.js.org/docs/angular) and [https://github.com/lauthieb/awesome-storybook](https://github.com/lauthieb/awesome-storybook) for details.

## Code coverage, reports and good practices

Code coverage, also called test coverage, tells you which parts of your code are executed by running the unit and integration tests. Code coverage is typically expressed as percent values, for example, 79% statements, 53% branches, 74% functions, 78% lines.

- Statements are, broadly speaking, control structures like if and for as well as expressions separated by semicolon.
- Branches refers to the two branches of if (…) {…} else {…} and … ? … : … conditions.
- Functions and lines are self-explanatory.

In Angular’s Karma and Jasmine setup, [Istanbul](https://istanbul.js.org/) is used for measuring test coverage. Istanbul rewrites the code under test to record whether a statement, branch, function and line was called. Then it produces a comprehensive test report.

To activate Istanbul when running the tests, add the `--code-coverage` parameter:

```sh
ng test --code-coverage
```

After the tests have completed, Istanbul saves the report in the /coverage directory located in the Angular project directory.

The report is a bunch of HTML files you can open with a browser. Start by opening coverage/index.html in the browser of your choice.

### Use it to find uncovered code

The coverage report reveals code behavior that is not yet tested. The report not only guides your testing, it also deepens your understanding of how your tests work.

**Testing should be part of the development routine. New features should include tests, bug fixes should include a test as proof and to prevent regressions**. Whatever your current coverage score is, use the reporting to monitor and improve your testing practice.

### Keep or improve the score

**Writing new code and changing existing code should not lower the coverage score, but gradually increase it**. This means if your existing tests cover 75% lines of code, new code needs to be at least 75% covered. Otherwise the score slowly deteriorates.

**It is common practice to run the unit and integration tests in a continuous integration environment and measure the code coverage. To enforce a certain coverage score and to prevent decline, you can configure the coverage reporting tool with thresholds**. For example, you can configure thresholds in the Karma configuration in karma.conf.js, you can add global thresholds for statements, branches, functions and lines.

```js
coverageReporter: {
  /* … */
  check: {
    global: {
      statements: 75,
      branches: 75,
      functions: 75,
      lines: 75,
    },
  },
},
```

In the configuration above, all values are set to 75%. If the coverage drops below that number, the test execution fails even if all specs succeeded.

### Raise the bar, try to improve

**When new code is added to the project with a test coverage better than average, you can raise the thresholds slowly but steadily – for example, from 75 to 75.1, 75.2, 75.3 and so on. Soon these small improvements add up**.

Test coverage should not be a pointless competition that puts developers under pressure and shames those that do not meet an arbitrary mark. Measuring coverage is a tool you should use for your benefit. Keep in mind that writing meaningful, spot-on tests does not necessarily increase the coverage score.

For beginners and experts alike, the coverage report helps to set up, debug and improve their tests. For experienced developers, the score helps to keep up a steady testing practice.
