# Testing on dotnet 6

## Concepts

### Tests types

#### Unit tests

A unit test is a test that exercises individual software components or methods, also known as "unit of work" or "system under test". Unit tests should only test code within the developer's control. They do not test infrastructure concerns. Infrastructure concerns include interacting with databases, file systems, and network resources.

#### Integration tests

An integration test differs from a unit test in that it exercises two or more software components' ability to function together, also known as their "integration." These tests operate on a broader spectrum of the system under test, whereas unit tests focus on individual components. Often, integration tests do include infrastructure concerns.

#### Load tests or stress testing

A load test aims to determine whether or not a system can handle a specified load, for example, the number of concurrent users using an application and the app's ability to handle interactions responsively. 

## Reference links

- [Unit testing best practices](https://docs.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)
- [ASP.NET Core load/stress testing](https://docs.microsoft.com/en-us/aspnet/core/test/load-tests)
