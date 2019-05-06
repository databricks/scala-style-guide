
1. [Testing](#testing)
    * [Intercepting Exceptions](#testing-intercepting)

## <a name='testing'>Testing</a>

### <a name='testing-intercepting'>Intercepting Exceptions</a>

When testing that performing a certain action (say, calling a function with an invalid argument) throws an exception, be as specific as possible about the type of exception you expect to be thrown. You should NOT simply `intercept[Exception]` or `intercept[Throwable]` (to use the ScalaTest syntax), as this will just assert that _any_ exception is thrown. Often times, this will just catch errors you made when setting up your testing mocks and your test will silently pass without actually checking the behavior you want to verify.

  ```scala
  // This is WRONG
  intercept[Exception] {
    thingThatThrowsException()
  }

  // This is CORRECT
  intercept[MySpecificTypeOfException] {
    thingThatThrowsException()
  }
  ```

If you cannot be more specific about the type of exception that the code will throw, that is often a sign of code smell. You should either test at a lower level or modify the underlying code to throw a more specific exception.

