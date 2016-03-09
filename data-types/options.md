# Options

`Option` is a container data type that is either empty (`None`) or full (`Some(value)`), which is a safe alternative to use `null` in writing scala code.

[TL;DR.](#final-notes)

## Missing Values

Suppose you have a method that may either return some value or null if the value doesn't exist, you may write something like:

```scala
def foo(shouldReturnNull: Boolean): Something =
  if (shouldReturnNull) { null }
  else { new Something() }
```

Yet, this will make later usages of `foo` become complicated. This requires you to check the existence of `Something` every time, while you may not know you need to do so.

Instead, if you write `foo()` in the following way:

```scala
def foo(shouldReturnNone: Boolean): Option[Something] =
  if (shouldReturnNone) { None }
  else { Some(new Something()) }
```

They looks no difference at all. However, with the `Option` container, we may manipulate the value inside without knowing `Something`'s existence. Or you could say, `Option` explicitly states the checking for emptiness is required for any manipulation.

If you would like to provide a default value to existing `Option`, you should use `getOrElse()` instead of pattern matching:

```scala
val value = optValue.getOrElse(default)
```

## Conditional Execution

In some cases, you may like to work on some nullable values, and explicit checking is needed to avoid runtime errors:

```scala
if (task !== null) {
  task.run()
}
```

If `task` where encapsulated with the `Option` data type, we may write such expression in:

```scala
optTask foreach { task =>
  task.run()
}
```

This may looks quite odd at first glance, however, despite it provides better safety to the execution of `run()`, it also makes sense if you consider `Option[Task]` as a collection of tasks, and you would like to `run()` all tasks exist in the collection.

In other cases that you may need to run some default task if the actual task doesn't exist, you can make use of the powerful pattern matching in scala to achieve branching:

```scala
optTask match {
  case Some(task) => task.run()
  case None => otherActions()
}
```

## Transforming Options

Since `Option` is a collection, we may apply `map()` and `flatMap()` to an `Option` like applying these functions to `List` data type.

In traditional imperative programming, if you want to transform a nullable value to another one, you will write:

```scala
def bar(foo: Foo): Bar =
  if (foo != null) { transform(foo) }
  else { null }
```

This will gives you the result of an instance of `Bar`, if the provided `Foo` value is not `null`.

With the collection API provided by `Option`, such procedure can be changed into:

```scala
def bar(optFoo: Option[Foo]): Option[Bar] =
  optFoo.map { foo => transform(foo) }
```

In the imperative example, the very same piece of code can be used if `transform()` is returning a nullable value as well. But let's consider the `Option` example, suppose `transform()` is returning an `Option[Bar]`, using `map()` method will result in `Option[Option[Bar]]` instead of `Option[Bar]`, to deal with such scenario, you would like to use `flatMap()`:

```scala
def bar(optFoo: Option[Foo]): Option[Bar] =
  optFoo.flatMap { foo => transform(foo) }
```

As stated by its name, `flatMap` is returning a flattened result of `map()`.

## Composing Options

Sometimes you're given several `Option`s, and you will need to construct a new `Option` only when all the given `Option`s contains a value.

Using `map()` and `flatMap()` extensively can achieve your desired result, but they'll become deeply nested like [callback hell](http://callbackhell.com/) in JavaScript code. To relieve yourself from the hell, for-comprehension comes to rescue.

```scala
val result = for {
  foo <- foo() // foo() => Option[Foo]
  bar <- bar() // bar() => Option[Bar]
  one <- Some(1) // Wraps with Some(1) for some known concrete value
} yield (foo, bar, one) // Option[(Foo, Bar, Int)]
```

In the above example, `result` will be `None` if any of `foo()` or `bar()` return `None`. And `result` will be `Some((foo, bar, 1))` if both of them return an `Option` contains a value inside.

## Java Compatibility

In traditional Java applications, or we are required to use some Java-based libraries for our application development. Then we are forced to use the value `null` for further computations.

The `Option` data type has a constructor method for converting a possible `null` value to an `Option`:

```scala
val nullableValue = foo()
val opt = Option(nullableValue)
```

`opt` with either be `Some(value)` if `javaCall()` returns something, or it will be `None` if `javaCall()` returns `null`, and you may continue to work on your code with the powerful `Option` data type.

## <a name="final-notes"></a>Final Notes

- Prefer `Option` over `null` for optional values.
- Use `getOrElse()` to provide default values for missing data.
- Use `foreach()` to execute side effects on an existing value.
- Use pattern matching if you need branching.
- Use `map()` and `flatMap()` to perform transformation.
- Use for-comprehension to compose `Option` from `Option`s.
- Use `Option(value)` constructor to wrap nullable values returned by Java-API.

## Reference Materials

- [Effective Scala - Options](http://twitter.github.io/effectivescala/#Functional%20programming-Options)
- [Scala's Option API](http://www.scala-lang.org/api/2.7.4/scala/Option.html)
