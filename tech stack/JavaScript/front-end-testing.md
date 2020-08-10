# Front-end Testing

## 按照测试流组织测试 使测试点集中 - [Write fewer, longer tests](https://kentcdodds.com/blog/write-fewer-longer-tests)

Making tests too short often leads to poor testing practices and way more tests.

Example: assert a component does a lot of rendering work and turn those into individual test cases.

There are a few **problems** with it:

1. The tests are not at all isolated (read Test Isolation with React)

2. Mutable variables are shared between tests (read Avoid Nesting when you're Testing)

3. Asynchronous things can happen between tests resulting in you getting act warnings (for this particular example)

> It's notable that the first two points there are applicable regardless of what you're testing. The third is a bit of an implementation detail between jest and act.

After combined tests, yes, we've violated that "one assertion per test" rule, but **that rule was originally created because** testing frameworks did a poor job of giving you the contextual information you needed to determine what was causing your test failures.

**Conclusion**: Don't arbitrarily separate your assertions into individual test blocks, there's no good reason to do so.

**The principle**: Think of a test case workflow for a manual tester and try to make each of your test cases include all parts to that workflow. This often results in multiple actions and assertions which is fine.

## 测试奖杯理论 - [Static vs Unit vs Integration vs E2E Testing for Frontend Apps](https://kentcdodds.com/blog/unit-vs-integration-vs-e2e-tests)

In the Testing Trophy, there are 4 types of tests(from top to bottom):

1. End to End: A helper robot that behaves like a user to click around the app and verify that it functions correctly. Sometimes called "functional testing" or e2e.

2. Integration(most): Verify that several units work together in harmony.

3. Unit: Verify that individual, isolated parts work as expected.

4. Static: Catch typos and type errors as you write the code.

### Test Types

#### End 2 End

run the entire application (both frontend and backend) and your test will interact with the app just like a typical user would, usually written with frameworks like `cypress`.

#### Integration

The idea behind integration tests is to mock as little as possible. I pretty much only mock:

* Network requests (see axiosMock)

* Components responsible for animation (because who wants to wait for that in your tests?)

#### Unit

single components, pure functions, etc.

#### Static

pretty much don't need this if we're using TypeScript.

### Trade-offs

Why do you write tests? => CONFIDENCE

Trade-offs:

* Cost: As you move up the testing trophy, the tests become more costly.

* Speed: As you move up the testing trophy, the tests typically run slower.

#### How to make trade-offs?

If you're operating at a low level you need more tests to cover the same number of lines of code in your application as a single test could higher up the trophy. In fact, as you go lower down the testing trophy, there are some things that are impossible to test.

> In particular, static analysis tools are incapable of giving you confidence in your business logic. Unit tests are incapable of ensuring that when you call into a dependency that you're calling it appropriately (though you can make assertions on how it's being called, you can't ensure that it's being called properly with a unit test). UI Integration tests are incapable of ensuring that you're passing the right data to your backend and that you respond to and parse errors correctly.

> If you try to use an E2E test to check that typing in a certain field and clicking the submit button for an edge case in the integration between the form and the URL generator, you're doing a lot of setup work by running the entire application (backend included). That might be more suitable for an integration test. If you try to use an integration test to hit an edge case for the coupon code calculator, you're likely doing a fair amount of work in your setup function to make sure you can render the components that use the coupon code calculator and you could cover that edge case better in a unit test. If you try to use a unit test to verify what happens when you call your add function with a string instead of a number you could be much better served using a static type checking tool like TypeScript.

## 常见测试误区 - [Common Testing Mistakes](https://kentcdodds.com/blog/common-testing-mistakes)

1. Testing implementation details

```jsx
test('the increment method increments count', () => {
  const wrapper = mount(<Counter />)
  // don't ever do this:
  expect(wrapper.instance().state.count).toBe(0)
  wrapper.instance().increment()
  expect(wrapper.instance().state.count).toBe(1)
})
```

Here are two truths about tests that focus on implementation details like the test above:

* I can break the code and not the test (eg: I could make a typo in my button's onClick assignment)

* I can refactor the code and break the test (eg: I could rename increment to updateCount)

avoid this because it doesn't give you very much confidence that your application is working and it slows you down when refactoring.

2. 100% code coverage

What code coverage is telling you:

* This code was run when your tests were run.

What code coverage is NOT telling you:

* This code will work according to the business requirements.

* This code works with all the other code in the application.

* The application cannot get into a bad state

I use the code coverage report to help me after I've already identified which parts of my application code are critical. It helps me to know if I'm missing some edge cases the code is covering but my tests are not.

3. Repeat testing

Repeat testing is a common mistake that people make when writing E2E tests that contribute to the poor performance and reliability.

eg. instead of include the registration process in each test that needs an authenticated user, make the same HTTP calls in the tests which will be much faster.

## 尽可能避免过度抽象和嵌套测试 - [Avoid Nesting when you're Testing](https://kentcdodds.com/blog/avoid-nesting-when-youre-testing)

Why using hooks like beforeEach as a mechanism for code reuse leads to unmaintainable tests and how to avoid it.

### Why

over-abstraction incur the cost for maintainers to have to look around the file for where those functions are defined.

nesting: what makes nested tests so complex?

You have to find where are the used variables coming from and what's their value, you also have to figure out where it's assigned, and then you have to make sure that it's not actually being assigned to something else in a further nested test.

The more you have to hold in your head for menial things like that, the less room there is for accomplishing the important task at hand.

### How

Apply AHA principle

> Avoid Hasty Abstractions: prefer duplication over the wrong abstraction and optimize for change first.

For grouping tests: The describe function is intended to group related tests together and can provide for a nice way to visually separate different tests, especially when the test file gets big. But I don't like it when the test file gets big. So instead of grouping tests by describe blocks, I group them by file.

For setup and cleanup: sometimes there are definitely good use cases for before*, but they're normally matched with a cleanup that's necessary in an after*. Like starting and stopping a server, testing console.error calls. Just, don't use these hooks as a mechanism for code reuse.

## 如何合理划分和隔离测试 - [Test Isolation with React](https://kentcdodds.com/blog/test-isolation-with-react)

Why your tests should be completely isolated from one another?

=> Improve the reliability of the tests, simplify the code, and increase the confidence your tests and provide as well.
