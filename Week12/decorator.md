# Decorators in Python

- A decorator is a function that **takes another function as input and extends or modifies its behavior**, without changing its code.
- Decorators are applied using the `@decorator_name` syntax placed **just above** a function definition.

  ```python
  def my_decorator(func):
      def wrapper():
          print("Before function call")
          func()
          print("After function call")
      return wrapper

  @my_decorator
  def say_hello():
      print("Hello!")

  say_hello()
  ```

  Output:

  ```text
  Before function call
  Hello!
  After function call
  ```

- **Equivalent to:** `say_hello = my_decorator(say_hello)`
- **Use cases:**

  - Logging and debugging
  - Authentication and authorization checks
  - Timing or performance measurement
  - Caching or memoization
