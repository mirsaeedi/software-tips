# Unit test

## Goal of unit testing

* The single most important benefit of unit testing is to be able to sleep tight at night ;)

* Another advantage of unit testing is gaining confidence in refactoring and modifying the code. Though, only well-written and thoughtful tests are capable of catching bugs introduced by code modification.

* The goal of unit testing is not 100% coverage. 

## Less known facts

* It’s overwhelmingly easy to write bad unit tests that add very little value to a project while increasing the cost of code changes substantially.

* Writing unit tests does not necessarily lead to confidence. You can achieve high code coverage without having thoughtful tests or getting edge cases covered. The sole existence of a unit test suite doesn’t provide any guarantees.

* Writing unit tests does not necessarily lead to a better design of production code. You can achieve high code coverage and still have a mess. However, if the code is hard to unit test, then it probably requires improvement. The sole existence of a unit test suite doesn’t provide any guarantees.

* Unit tests do not necessarily speed up the development process. Tests that are coupled to implementation details are fragile. So, a slight change in production code could break several tests. Developers become reluctant to improve the production code as they also need to put a lot of time into fixing the broken tests.

## What to test

* Writing and maintaining unit-tests is expensive. Hence, we should be careful where we invest our time and how we can have the best return of investment. [Steven Sanderson](http://blog.stevensanderson.com/2009/11/04/selective-unit-testing-costs-and-benefits/) and [Khorikov](https://enterprisecraftsmanship.com/posts/painless-tdd/) advise us to only unit-test domain logic and business-critical parts of the code. Trivial code or a code that directly communicates with external dependencies, e.g., network or database, does not need to be unit-tested. 

* Invest your time in writing tests for business requirements. The most valuable tests verify the observable behavior as it seems to appear from the end user’s perspective. The rest is implementation detail and is subject to change. The closer you can get to this kind of verification, the better. Overall, try to constantly ask yourself a question: does this test verify some business requirements? If the answer is no, remove it. The most valuable tests are always the tests that have at least some connection to the business requirements your codebase is ought to address.

* The unit under test is not a method or a class. The unit of test is a business requirement that can expand across multiple classes and methods.

* In writing unit tests, we just need to shift our focus from hows of the SUT to its whats and verify the end result instead.

* Maintainable tests do not care how the method under test is implemented. They just validate the output for a given input. Hence, using mocks should be avoided as much as possible, as excess use of mocks indicates tests are coupled to the implementation details of the method under test.

## Good tests

* Writing good unit tests requires a different skill-set compared to regular coding. Like most aspects of programming, testing requires **thoughtfulness**.

* Unit tests should be **fast** to enable us to get feedback quickly,otherwise developers lose their interest in executing tests regularly. 

* Unit tests should be **deterministic**. In other words, running a test should always lead to the same outcome with a given input. Code that works with time or external resources is not deterministic.

* All tests should be run **randomly** in parallel without a specific order. It allows to shorten the execution time of the test suit and get feedback quickly. It implies that tests should be isolated from each other and have no side-effect.

* I find it really important to focus on keeping tests **isolated**. Properly isolated tests can be run in any sequence.

* A unit test has three phases that are known as **Arrange**, **Act**, and **Assert**, or simply **AAA**. In tests, separate each section using a comment line.

* A unit test is [valuable](https://enterprisecraftsmanship.com/posts/unit-test-value-proposition/) if it has the three following properties:  
  * Has a high chance of catching a regression bug. Not all unit tests are capable of catching bugs. Some of them are just written to satisfy management by increasing the coverage.
  * Has a low chance of producing a false positive. A false positive is a situation where your test suite raises a false alarm: indicates an error, whereas, in reality, everything works fine. It happens when tests are non-deterministic or are coupled to implementation details, e.g. excess use of mocks.
  * Provides fast feedback.

## Code Coverage

*  Many in our industry claim that any unit tests are better than none, but Steven Sanderson [disagrees](http://blog.stevensanderson.com/2009/08/24/writing-great-unit-tests-best-and-worst-practises/): a test suite can be a great asset, or it can be a great burden that contributes little. It depends on the quality of those tests, which seems to be determined by how well its developers have understood the goals and principles of unit testing. 

* Test coverage is of little use as a numeric statement of how good your tests are. It is not but a useful tool for finding untested parts of a codebase. 

* High coverage numbers are too easy to reach with low quality testing. Make a certain level of coverage a target, people will attain it. But it guarantees nothing.

* Certainly low coverage numbers, say below half, are a sign of trouble. But high numbers don't necessarily mean much. 

## Non Determinism

* [Non-deterministic](https://martinfowler.com/articles/nonDeterminism.html) (flaky) tests, that sometimes pass and sometimes fail without any noticeable change in the code, tests, or environment, can completely destroy the value of an automated regression suite. Such tests fail, then you re-run them and they pass. Test failures for such tests are seemingly random. Quarantine them intially but fix them soon.

* The root cuase of non-determinism is twofold, side effects and using external resources. For example, if a test modifies a shared resource, e.g., file, database, or variable, that is used by another test, we might observe a different behavior each time we run the test suit as the test execution is concurrent and randomly ordered. Besides that, using external resources, such as time, 
network, file system, and database, might bring non-determinism to your tests, because consuming these resources might fail for a variety of reasons, e.g., timeout, permission error, or disk failure.

* Few things are more non-deterministic than a call to the system clock. Each time you call it, you get a new result, and any tests that depend on it can thus change. Therefore, avoid using `DateTime.Now` or any other static method for getting time, because static methods are not mockable and time changes between test runs, and it might result in 
non-deterministic behavior. Instead, [redesign your code](https://martinfowler.com/bliki/ClockWrapper.html
), to get `DataTime` or `IDateTimeService` as a parameter. 

* Static is evil. When communicating with out-of-process resouces by using static methods, e.g. `DateTime.Now`, `Guid.NewGuid()`, `Directory.Exists(string path)`, 
`Environment.GetEnvironmentVariable("PATH")`, or a static singleton instance of a custom class, your code would become non-testable, because you cannot mock static methods.

## Test Quality & Coupling to Production Code

* David Hansson introduced the concept of **test-induced design damage** introduced in a [series of discussion](http://martinfowler.com/articles/is-tdd-dead/) with Martin Fowler and Kent Beck. It basically says that damaging your code is inevitable when you make it testable. In other words, testablity introduces a degree of complexity to your code. For example, you must avoid using static methods of other classes in your code or you must pass all dependencies explicitly through the constructor/method parameters.

* Dependencies can be divided into two categories: stable and volatile. 
  * Volatile dependencies are dependencies that work with the outside world, for example, with the database, an external HTTP service, time, file system, network, or OS. These dependencies are non-deterministic. You might get a different result each time you consume them, e.g. unexpected failures. You should mock volotile dependencies to eradicate non-determinism in your tests.
  * Stable dependencies, on the other hand, are self-contained and don’t interact with any resources outside the process in which the SUT is executed. These dependencies are deterministic. Mocking stable dependencies just doesn’t make any sense because both the mock and the original object have predictable behavior which doesn’t change because of external factors

* Unit-tests are here to give us confidence in refactoring and modifying code. But not well-designed tests do quite the opposite, because each change to the code breaks lots of tests, and it makes developers to not be interested in improving the code. To reduce the coupling between tests and code, we need to make sure tests are not depenendent on implementation details of the code. **Excess use of mocks**, and [collaboration verification](https://enterprisecraftsmanship.com/posts/styles-of-unit-testing) through **`mock.Verify`** [couples](https://enterprisecraftsmanship.com/posts/most-important-tdd-rule/) tests to implementation details and impedes us from refactoring the code with confidence. 

* One of the pillars of having testable code is to pass dependencies through parameters. However, on the other hand, passing dependencies through parameters make your tests fragile, because each change to the constructor/method signature breaks all associated tests. It's a [pain in the neck](https://enterprisecraftsmanship.com/posts/test-induced-design-damage-or-why-tdd-is-so-painful/) and makes your test solution less maintainable.

* [Test doubles are only for extenal dependencies](https://enterprisecraftsmanship.com/posts/tdd-best-practices/). We should not mock dependencies if they are part of our code and do not communicate with the external world. Therefore, we should pass the actual dependencies to exercise all our code and the communication between components. Using mocks instead of actual classes prevents us from finding possible bugs that happen during interaction of components. Also, using mocks couples tests with implementation details which  makes tests fragile, as we need to change the test every time method signature changes.

* Unit tests with long list of arrangements and mocks are less maintainable, because a change to the method signature of the SUT or any of the depenencies breaks tests.

* Use `new` with caution in your code. It makes your code coupled to a specific concrete type, as a result, you cannot mock that dependency if it's required for testing. Consider injecting the dependency through parameters to make the class testable and more readable as its dependencies would become explicit.

* Do not introduce additional code to your main code base in order to enable unit testing. Don’t introduce production code that doesn’t run in production. For example, don’t expose state getters solely for satisfying a test. If something is not going to be observable from the outside world, it should not be observable to tests, because it is implementation details.

* If you use mocks, you most likely couple your unit tests to the SUT’s implementation details. There are a few legitimate use cases for mocks. They can be useful when you want to substitute a volatile dependency you don’t control. Also, mocks as an instrument also can be quite beneficial if you use them to create stubs.

* Excess use of mocks defeats the whole purpose of unit testing: having a solid test suite which you can trust and rely upon, because mocks couple tests to the SUT's implementation details, and, as a result, changes in production code could easily break tests which leads to false positives, tests that are broken because of code change.

* Your code should either depend on the outside world, or represent business logic, but never both

## Integration Tests

* In integration tests, we might need to make calls to database. One trick that helps to keep tests's side-effects isolated, is to conduct your tests inside a transaction, and then to rollback the transaction at the end of the test. That way the transaction manager cleans up for you, reducing the chance of errors.

* Only integration tests can give us confidence that the application we develop actually works as a whole. They are also a good substitute for mocks in the cases where you can’t test important business logic without involving external dependencies.

* With integration tests, check only a single happy path per application service method. Also, if there are any edge cases which cannot be covered with unit tests, check them as well.

* Even with good practices on writing integration tests, end-to-end tests are more prone to non-determinism problems, which can undermine trust in them.
  * Tests that run end-to-end through the UI are: brittle, expensive to write, and time consuming to run. 
  * End to End tests are there as a second line of test defense. If you get a failure in a high level test, not just do you have a bug in your functional code, you might also have a missing or incorrect unit test.

## Patterns

* [**Constrained Non-Determinism**](https://blog.ploeh.dk/2009/03/05/ConstrainedNon-Determinism/): Try not to hard-code dummy values in tests. Instead, we can use well-defined, but random, input, because when input is random, 
we do not accidentally hard-code any assumptions. For example, for generating random strings, we can use `Guid.NewGuid().ToString()`.

* An [object mother](https://www.martinfowler.com/bliki/ObjectMother.html) is a kind of class used in testing to help create example objects that you use for testing.

* Name your unit tests clearly and consistently. Two popular naming patterns are **What When Should** or **Given When Then**.

## Test pyramid

* The [test pyramid](https://martinfowler.com/bliki/TestPyramid.html) is a way of thinking about how different kinds of automated tests should be used to create a balanced portfolio. Its essential point is that you should have many more low-level unit tests than high level end to end running through a GUI.

*  One sign you are testing too much is if your tests are slowing you down. If it seems like a simple change to code causes excessively long changes to tests, that's a sign that there's a problem with the tests. This may not be so much that you are testing too many things, but that you have duplication in your tests.

*  So the pyramid argues that you should do much more automated testing through unit tests than you should through traditional GUI based testing.


## Must Reads

1. [Styles of unit testing](https://enterprisecraftsmanship.com/posts/styles-of-unit-testing)
2. [Test-induced design damage](https://dhh.dk/2014/test-induced-design-damage.html)
3. [Unit tests value proposition](https://enterprisecraftsmanship.com/posts/unit-test-value-proposition/)
4. [Pragmatic unit testing](https://enterprisecraftsmanship.com/posts/pragmatic-unit-testing/)
5. [Writing Great Unit Tests: Best and Worst Practices](http://blog.stevensanderson.com/2009/08/24/writing-great-unit-tests-best-and-worst-practises/)
6. [Selective Unit Testing – Costs and Benefits](http://blog.stevensanderson.com/2009/11/04/selective-unit-testing-costs-and-benefits/)
7. [Test Coverage](https://martinfowler.com/bliki/TestCoverage.html)
8. [Eradicating Non-Determinism in Tests](https://martinfowler.com/articles/nonDeterminism.html)
9. [Test Pyramid](https://martinfowler.com/bliki/TestPyramid.html)
10.[Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)
11.[When to Mock](https://enterprisecraftsmanship.com/posts/when-to-mock/)
12.[Unit Testing Dependencies: The Complete Guide] https://enterprisecraftsmanship.com/posts/unit-testing-dependencies

## Must Watch
1. [Ian Cooper - TDD, Where Did It All Go Wrong](https://www.youtube.com/watch?v=EZ05e7EMOLM)
