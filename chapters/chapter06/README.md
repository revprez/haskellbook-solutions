# Type Classes

## Directions

* [Table of Contents](../../README.md)
* Solutions availables [here](exercises/README.md).

## Overview

* This chapter will define type classes and give special attention given to certain examples:
    1. `Bool`,
    2. `Eq`,
    3. `Num`,
    4. `Prd`,
    5. `Enum`,
    6. `Show`, and
    7. `Read`
* We will also learn about:
    * type-defaulting type classes,
    * type class inheritance

## What are type classes ?

* **Problem**: We have a set of [data types](../chapter04/README.md) that should be operated upon similarly. How do we do that?
    * **Solution**: We use **type classes**.
* From [Wikipedia](https://en.wikipedia.org), [Type classes](https://en.wikipedia.org/wiki/Type_class) are the mechanism by which we constrain [parametrically polymorphic types](../chapter05/README.md#Polymorphism).
* [Parametrically polymorphic types](../chapter05/README.md#Polymorphism) are useful, but only a limited number of operations can be applied to them. Type class constraints permit a richer field of operations to be applied, for example:
    * Addition `(+)` and multiplication `(*)` are defined for members of the type class `Num a => a`
    * Comparison operators `(>)`, `(<)` are defined for members of the type class `Ord a => a`
* **Type classes** declare operations.
* **Instances** supply implementation for operations declared in **typeclasses**
* **NOTE**: This is a bit of a spoiler alert, since we haven't gotten to **instances** yet, but type classes are **not** a magic way to define an operation **exactly once** such that it works on all conceivable data types assigned to a type class. **Inheritance** saves us a fair bit of time and work, but there will be cases where you must define new behavior for an operation (similar to [function overloading](https://en.wikipedia.org/wiki/Function_overloading)).

## Back to Bool

* [Data types](../chapter04/README.md) can be constrained by type classes given the following specification (`:info` leaves out the implementation, so don't forget it):
    ```haskell
    ghci> :info Bool
    data Bool = False | True        -- Defined in ‘GHC.Types’
    instance Bounded Bool -- Defined in ‘GHC.Enum’
    instance Enum Bool -- Defined in ‘GHC.Enum’
    instance Eq Bool -- Defined in ‘GHC.Classes’
    instance Ord Bool -- Defined in ‘GHC.Classes’
    instance Read Bool -- Defined in ‘GHC.Read’
    instance Show Bool -- Defined in ‘GHC.Show’
    ```
* **Instances** implement a definition of *how* a data type adheres to its type class, canonically by providing an implementation of an operation defined in a type class. We will get to how those operations are declared in a type class in the first place in [a little bit](#Eq).
* Note that the data type is instrumented with several invocations of the keyword `instance` followed by a type class constraint and the data type's name. We will learn how to define these ourselves [a bit later](#Writing-type-class-instances).
* Refer to the text for details on what these type class constraints on `Bool` actually mean. Refer to [Hackage](http://hackage.haskell.org/package/base-4.10.0.0/docs/Prelude.html#t:Bool) for `Bool`'s precise specification and source. Main takeaway is that actual work had to be done to define these instances. For example:
    ```haskell
    ------------------------------------------------------------------------
    -- Bool
    ------------------------------------------------------------------------

    -- | @since 2.01
    deriving instance Bounded Bool

    -- | @since 2.01
    instance Enum Bool where
      succ False = True
      succ True  = errorWithoutStackTrace "Prelude.Enum.Bool.succ: bad argument"

      pred True  = False
      pred False  = errorWithoutStackTrace "Prelude.Enum.Bool.pred: bad argument"

      toEnum n | n == 0    = False
               | n == 1    = True
               | otherwise = errorWithoutStackTrace "Prelude.Enum.Bool.toEnum: bad argument"

      fromEnum False = 0
      fromEnum True  = 1

      -- Use defaults for the rest
      enumFrom     = boundedEnumFrom
      enumFromThen = boundedEnumFromThen
    ```
* Type classes can encompass other type classes, for example:
    * `Num a => a` encompasses all members of
        * `Integral a => a`,
        * `Fractional a => a`,
        * `Real a => a`,
    * `Eq a => a` encompasses all members of `Ord a => a`

## Eq

* As mentioned previously,
    1. type classes declare operations, and
    2. instances supply the needed implementation.
* Let's take a look at `Eq` to reinforce the point:
    ```haskell
    ghci> :info Eq
    class Eq a where
      (==) :: a -> a -> Bool
      (/=) :: a -> a -> Bool

    -- a bunch of instance declarations follow, we'll ignore them for now
    ```
    * `Eq` declares two operations:
        1. `(==) :: Eq a => a -> a -> Bool` equivalence
        1. `(/=) :: Eq a => a -> a -> Bool` non-equivalence
    * Instances would bind these operations to data types and provide their implementation.
* Constrained types can be contrained even further by applying their operations to arguments:
    * Partial application
        ```haskell
        ghci> :t (==) 2
        (==) 2 :: (Num a, Eq a) => a -> Bool
        ```
    * Complete reduction
        ```haskell
        ghci> :t (==) 2 2
        (==) 2 2 :: Bool
        ```
* As the type signature indicates, `Eq`'s operations require arguments to be of the same type. A helpful error is emitted if equality is tested against arguments of different types:
    ```haskell
    ghci> "hello" == ("hello", 1)

    <interactive>:12:12: error:
        • Couldn't match expected type ‘[Char]’
                      with actual type ‘([Char], Integer)’
        • In the second argument of ‘(==)’, namely ‘("hello", 1)’
          In the expression: "hello" == ("hello", 1)
          In an equation for ‘it’: it = "hello" == ("hello", 1)
    ```
* Recall that partial application further constrains the type signature.
    ```haskell
    ghci> :t (==) "hello"
    (==) "hello" :: [Char] -> Bool
    ```
    * As you can see, applying the first argument obligates `[Char]` as the type for the second.
* To understand n-tuple equality, consider typeclass constraints on `(,)`.
    ```haskell
    ghci> :info (,)
    data (,) a b = (,) a b  -- Defined in ‘GHC.Tuple’
    instance (Bounded a, Bounded b) => Bounded (a, b)
      -- Defined in ‘GHC.Enum’
    instance (Eq a, Eq b) => Eq (a, b) -- Defined in ‘GHC.Classes’
    instance Monoid a => Monad ((,) a) -- Defined in ‘GHC.Base’
    instance Functor ((,) a) -- Defined in ‘GHC.Base’
    instance (Ord a, Ord b) => Ord (a, b) -- Defined in ‘GHC.Classes’
    instance (Read a, Read b) => Read (a, b) -- Defined in ‘GHC.Read’
    instance (Show a, Show b) => Show (a, b) -- Defined in ‘GHC.Show’
    instance Monoid a => Applicative ((,) a) -- Defined in ‘GHC.Base’
    instance Foldable ((,) a) -- Defined in ‘Data.Foldable’
    instance Traversable ((,) a) -- Defined in ‘Data.Traversable’
    instance (Monoid a, Monoid b) => Monoid (a, b)
      -- Defined in ‘GHC.Base’
    ```
    * `Bound`, `Eq`, `Ord`, `Show`, `Read` and `Monoid` distribute their constraints to the first and second elements of the 1-tuple.
        ```haskell
        ghci> (1, "hello") == (2, [' ', 'w', 'o', 'r', 'l', 'd'])
        False
        ghci> (1, "hello") == (1,2)

        <interactive>:24:20: error:
            • No instance for (Num [Char]) arising from the literal ‘2’
            • In the expression: 2
              In the second argument of ‘(==)’, namely ‘(1, 2)’
              In the expression: (1, "hello") == (1, 2)
        ```
    * Haskell defines instances [up to 15-tuple](https://www.haskell.org/onlinereport/basic.html#basic-tuples), and certain operations like `zip` up to 7.
    ```haskell
    -- 15 tuple

    ghci> :info (,,,,,,,,,,,,,,)
    data (,,,,,,,,,,,,,,) a b c d e f g h i j k l m n o
      = (,,,,,,,,,,,,,,) a b c d e f g h i j k l m n o
            -- Defined in ‘GHC.Tuple’
        instance (Bounded a, Bounded b, Bounded c, Bounded d, Bounded e,
                  Bounded f, Bounded g, Bounded h, Bounded i, Bounded j, Bounded k,
                  Bounded l, Bounded m, Bounded n, Bounded o) =>
                 Bounded (a, b, c, d, e, f, g, h, i, j, k, l, m, n, o)
          -- Defined in ‘GHC.Enum’
        instance (Eq a, Eq b, Eq c, Eq d, Eq e, Eq f, Eq g, Eq h, Eq i,
                  Eq j, Eq k, Eq l, Eq m, Eq n, Eq o) =>
                 Eq (a, b, c, d, e, f, g, h, i, j, k, l, m, n, o)
          -- Defined in ‘GHC.Classes’
        instance (Ord a, Ord b, Ord c, Ord d, Ord e, Ord f, Ord g, Ord h,
                  Ord i, Ord j, Ord k, Ord l, Ord m, Ord n, Ord o) =>
                 Ord (a, b, c, d, e, f, g, h, i, j, k, l, m, n, o)
          -- Defined in ‘GHC.Classes’
        instance (Read a, Read b, Read c, Read d, Read e, Read f, Read g,
                  Read h, Read i, Read j, Read k, Read l, Read m, Read n, Read o) =>
                 Read (a, b, c, d, e, f, g, h, i, j, k, l, m, n, o)
          -- Defined in ‘GHC.Read’
        instance (Show a, Show b, Show c, Show d, Show e, Show f, Show g,
                  Show h, Show i, Show j, Show k, Show l, Show m, Show n, Show o) =>
                 Show (a, b, c, d, e, f, g, h, i, j, k, l, m, n, o)
          -- Defined in ‘GHC.Show’


    -- 16 tuple
    ghci> :info (,,,,,,,,,,,,,,,)
    data (,,,,,,,,,,,,,,,) a b c d e f g h i j k l m n o p
      = (,,,,,,,,,,,,,,,) a b c d e f g h i j k l m n o p
            -- Defined in ‘GHC.Tuple’
    ```
* We can [derive](https://wiki.haskell.org/Keywords#deriving) automagically from a few type classes that are built into Haskell, including `Eq`, `Ord`, `Enum`, `Bounded`, `Read` and `Show`. As the text indicates, we'll get to the nuts and bolts of **deriving** [later](../chapter11/README.md).

## Writing type class instances

* Type class instances are typically packaged with the data types that require them. Consider this example ([trivial.hs](scratch/trivial.hs)):
    ```haskell
    module Trivial where

    data Trivial =
        Trivial

    instance Eq Trivial where
        -- infix notation
        Trivial == Trivial = True
    ```
    * Alternatively
    ```haskell
    module Trivial where

    data Trivial =
        Trivial

    instance Eq Trivial where
        -- prefix notation
        (==) Trivial Trivial = True
    ```
    * **instance** initiates the declaration.
    * `Eq` is the typeclass we wish to attach to the data type `Trivial`.
    * `Trivial` is the data type we wish to associate with `Eq` constraining operations.
    * `Trivial == Trivial = True` or `(==) Trivial Trivial = True` is essentially read "the application of `(==)` to `Trivial` and itself is `True`"
    * Loading into the REPL, we see that simply defining the equality case automatically buys us its negation.
        ```haskell
        ghci> :l scratch/trivial.hs
        [1 of 1] Compiling Trivial          ( scratch/trivial.hs, interpreted )
        Ok, modules loaded: Trivial.
        ghci> Trivial == Trivial
        True
        ghci> Trivial /= Trivial
        False
        ```
* Review `DayOfTheWeek` example in the text. Consult the implementation [here](scratch/dayoftheweek.hs) and pay particular attention to the `Eq Date` instance.
    ```haskell
    instance Eq Date where
      (==)  (Date weekday dayOfTheMonth)
            (Date weekday' dayOfTheMonth') =
            weekday == weekday'
        &&  dayOfTheMonth == dayOfTheMonth'
    ```
    * Rather than define a length list of cases, we use some math to declare that `Date` equality is satisfied when the equality test on pairs of constituents `DayOfTheWeek` and `Int` (which already have instances) are evaluated.

### Partial functions - not so strange danger

* **Partially defined functions**, not to be confused with [partial application](../chapter05/README.md#Partial-application), are functions with undefined behavior for certain inputs. The Haskell compiler and REPL will throw errors when encountering these.
* Consider [dayoftheweek_broken.hs](scratch/dayoftheweek_broken.hs). It omits the final match in the instance declaration which would return false for unequal comparisons. Let's also set `-Wall` in the REPL to examine the errors that will arise in more detail.
    ```haskell
    ghci> :set -Wall
    ghci> :l scratch/dayoftheweek.hs
    [1 of 1] Compiling DayOfTheWeek     ( scratch/dayoftheweek.hs, interpreted )
    Ok, modules loaded: DayOfTheWeek.
    ghci> Mon == Tue
    False
    ghci> :l scratch/dayoftheweek_broken.hs
    [1 of 1] Compiling DayOfTheWeek     ( scratch/dayoftheweek_broken.hs, interpreted )

    scratch/dayoftheweek_broken.hs:9:5: warning: [-Wincomplete-patterns]
        Pattern match(es) are non-exhaustive
        In an equation for ‘==’:
            Patterns not matched:
                Mon Tue
                Mon Wed
                Mon Thu
                Mon Fri
                ...
    Ok, modules loaded: DayOfTheWeek.


    ghci> Mon == Tue
    *** Exception: scratch/dayoftheweek_broken.hs:(9,5)-(15,23): Non-exhaustive patterns in function ==
    ```
    * `-Wall` provides us with helpful information, in this case the error tells us that there are conceivable patterns that remain unmatched and therefore undefined.
* The following (admittedly contrived) example also exhibits this non-exhaustive pattern behavior:
    ```haskell
    ghci> f :: Int -> Bool; f 2 = True

    <interactive>:15:19: warning: [-Wincomplete-patterns]
        Pattern match(es) are non-exhaustive
        In an equation for ‘f’:
            Patterns not matched: p where p is not one of {2}
    ```
    * We resolve this by adding a case for non-explicitly defined inputs:
    ```haskell
    ghci> f :: Int -> Bool; f 2 = True; f _ = False
    ghci> f 2
    True
    ghci> f 3
    False
    ```
* The case handling definition of a function above presages [pattern matching](../chapter07/README.md#Pattern-matching). We will cover **pattern matching** in detail starting in [Chapter 7](../chapter07/README.md).

### Sometimes we need to ask for more

* Instances may use operations that requiring constraining otherwise polymorphically unconstrained types. Consider [identity.hs](scratch/identity.hs).
    ```haskell
    data Identity a = Identity a

    -- ...

    instance Eq a => Eq (Identity a) where
        (==) (Identity v) (Identity v') = v == v'
    ```
    A type class constraint `Eq a => ...` is introduced so that the expression `v == v'` makes sense to the compiler.  Without it, we would yield an error (see [identity_insufficient.hs](scratch/identity_insufficient.hs)):
    ```haskell
    ghci> :l scratch/identity_insufficient.hs
    [1 of 1] Compiling Identity         ( scratch/identity_insufficient.hs, interpreted )

    scratch/identity_insufficient.hs:10:39: error:
        • No instance for (Eq a) arising from a use of ‘==’
          Possible fix: add (Eq a) to the context of the instance declaration
        • In the expression: v == v'
          In an equation for ‘==’: (==) (Identity v) (Identity v') = v == v'
          In the instance declaration for ‘Eq (Identity a)’
    Failed, modules loaded: none.
    ```


## Num

* Numbers richly demonstrate the interaction between data types, type classes, and instances.
    * The following are *data types*:
        * `Int`: Fixed precision: set of integers defined on the interval  `[-9223372036854775809,9223372036854775807]` or `[-2^63, 2^63]` on a 64-bit architecture.
        * `Word`: Essentially unsigned int, defined on the interval `[0,18446744073709551616]` or `[0,2^64]` on a 64-bit architecture.
        * `Integer`: Arbitrary precision integers. Interval not constrained by architecture width.
        * `Float`: Single-precision floating point
        * `Double`: Double-precision floating point
        * `Rational`: Equivalent to `Ratio a`, which is defined in GHC.Real. Represents the rational numbers.
        * Beyond the base system, there are several other numeric types:
            * `Ratio a`: Defined in GHC.Real. A [parameterized data type (more on this in Chapter 11)](../chapter11/README.md#Data-and-type-constructors) representing the ratio of of two polymorphic types.  We construct ratios with the infix operator [`%`](http://hackage.haskell.org/package/base-4.10.0.0/docs/Data-Ratio.html#v:-37-), which we will encounter in more detail in [Chapter 24](../chapter24/README.md#Parsing-fractions)
            * `Complex a`: Represents complex numbers, duh. Notable in that `Ord` is not defined on these, as complex numbers represent a pair and pairs are unordered sets.
    * The following are *type classes*:
        * `Eq`: Defines parametrically polymorphic equality
        * `Bounded`: Defines operations for bounded types. `Word`, `Int`, and `Integer` specifically in the numerical family.
        * `Ord`: Defines operations for ordered types, with `Complex` being the notable exception in the numeric family.
        * `Integral`: defines operations on `Int`
        * `Fractional`: defines operations on types `Num a`
        * `Real`: defines operations on reals
        * `Floating`: defines operations on reals and complex numbers
        * `RealFrac`: defines operations on real fractional numbers
        * `RealFloat`: defines operations on real floating-point numbers
    * *Instances* are defined for type classes on each of their pertinent monomorphic types. Run `:info <type class>` against the type classes listed above to see the list of instances defined for each one.

## Type-defaulting type classes

* Numerical computations [specify defaults](https://prime.haskell.org/wiki/Defaulting#Proposal1-nametheclass) for flowing from polymorphic representation to the monomorphic result the compiler must emit.
    ```haskell
    default Num Integer
    default Real Integer
    default Enum Integer
    default Integral Integer
    default Fractional Double
    default RealFrac Double
    default Floating Double
    default RealFloat Double
    ```
    * The text mentions this can be found in the [Haskell Report (2010)](https://www.haskell.org/onlinereport/haskell2010/), but I was unable to find it. In fact, I was only able to find mention of defaulting on the [Haskell Prime](https://prime.haskell.org/) site.
    * **Note**: evaluating the type of an expression expression like `1 / 2` in the REPL will yield the broadest constrained polymorphic expression. In fact, evaluating the type of an unannotated number typically will not yield a monomorphic type:
        ```haskell
        ghci> :t 1 / 2
        1 / 2 :: Fractional a => a
        ghci> :t 1.0 / 2
        1.0 / 2 :: Fractional a => a
        ghci> :t 1.0 / 2.0
        1.0 / 2.0 :: Fractional a => a
        ghci> :t 1.0 `div` 2.0
        1.0 `div` 2.0 :: (Fractional a, Integral a) => a
        ghci> :t 1
        1 :: Num t => t
        ghci> :t 1.0
        1.0 :: Fractional t => t
        ```
* We can further constrain, even unto monomorphism, more polymorphic operations, but we cannot do the opposite:
    ```haskell
    ghci> intProd = (*) :: Int -> Int -> Int
    ghci> :t intProd
    intProd :: Int -> Int -> Int

    ghci> integralProd = intProd :: Integral a => a -> a -> a

    <interactive>:54:16: error:
        • Couldn't match type ‘a1’ with ‘Int’
          ‘a1’ is a rigid type variable bound by
            an expression type signature:
              forall a1. Integral a1 => a1 -> a1 -> a1
            at <interactive>:54:27
          Expected type: a1 -> a1 -> a1
            Actual type: Int -> Int -> Int
        • In the expression: intProd :: Integral a => a -> a -> a
          In an equation for ‘integralProd’:
              integralProd = intProd :: Integral a => a -> a -> a

    ```

## Ord

* The `Ord` type class:
    1. Requires constituent types to have instances be defined for type class `Eq`
    1. Defines in the base system instances specifically for the primitives `Word`, `Ordering`, `Bool`, `Char`, the numeric types `Int`, `Integer`, `Float`, `Double`, as well as orderable lists and tuples of these constituents.
* `Ord` contains definitions for operations on orderable types, including the comparison operators.

## Enum

* The `Enum` typeclass describes a family of types that are **enumerable**--types with "known predecessors and successors." It defines several operations you can discover with `:info Enum`, as well as instances for the primitives as well as lists and tuples thereof.
* `Enum`'s primary purpose in the base system seems to be as a means of implementing [arithmetic sequences](https://www.haskell.org/onlinereport/haskell2010/haskellch3.html#x8-400003.10).

## Show

* The `Show` typeclass describes a family of types that are **printable**. That is the `print` function (not actually defined on the `Show` type class, but as a standalone in the [`System.IO`](http://hackage.haskell.org/package/base-4.10.0.0/docs/src/System.IO.html#print) module) can accept values of these types and (crudely put) display them on some output device. I imagine the actual implementation of Show is considerably more capable than simply printing to a monitor, but this will suffice for now.
* Like `Enum`, `Show` defines instances for the primitives as well as lists and tuples thereof.
* `Show` is our second real exposure to how Haskell deals with impure operations (input-output). The first was the `main` function we've encountered several times in exercises in order to prepare a main loop for program execution. Like `main`, the final result of `print` is of type `IO`.
    ```haskell
    ghci> :t print
    print :: Show a => a -> IO ()
    ```
    * We will discuss `IO ()` in greater detail in [Chapter 29](../chapter29/README.md). By that point, we'll also have a better understanding of **category theory** and especially **monads** (see chapters [15](../chapter15/README.md) through [18](../chapter18/README.md)) and why we can tolerate this breach in purity.

## Read

* The `Read` typeclass is essentially `Show`'s inverse. Instead of defining operations like  text to some device, it reads `[Char]` and attempts to parse it into a target (monomorphic) type.
* The function `read` is not actually defined on the type class `Read` from [`GHC.Read`](https://www.haskell.org/haddock/libraries/GHC.Read.html), but as a standalone function in the module [`Text.Read`](http://hackage.haskell.org/package/base-4.10.0.0/docs/src/Text.Read.html#read). Its implementation hides some complexity in using `GHC.Read`'s type class functions `readPrec` and `readListPrec`.
    ```haskell
    ghci> read "123" :: Int
    123

    ghci> x = read "123" :: Int
    ghci> x
    123
    ghci> :t x
    x :: Int

    ghci> x = read "True" :: Bool
    ghci> x
    True
    ghci> :t x
    x :: Bool
    ```
* `read` must always be supplied a type annotation, or shit happens.
    ```haskell
    ghci> read "123"
    *** Exception: Prelude.read: no parse
    ```
* `read` must always be supplied a **parseable** type annotation, or shit happens.
    ```haskell
    ghci> read "123" :: Bool
    *** Exception: Prelude.read: no parse
    ```
* `read` will blow up if you try to do something trivial like this:
    ```haskell
    ghci> read "123" :: [Char]
    *** Exception: Prelude.read: no parse
    ghci> read "a" :: Char
    *** Exception: Prelude.read: no parse
    ```
* And most importantly, `read` must be pinned to a monomorphism.
    ```haskell
    ghci> read "123" :: Num a => a
    *** Exception: Prelude.read: no parse
    ```

## Instances are dispatched by type

* Type classes declare operations.
* Instances supply implementations for those declarations for specific concrete or constrained type. This is similar to ad hoc polymorphism in other languages, i.e., Java:
    ```java

    public static String convertToString(Integer i) { ... }

    public static String convertToString(Double i) { ... }
    ```
* Haskell reasons dispatch from invocation of some operation declared by a type class to an instance implementation by inferring the tpe
    * In Haskell, we might represent the above example like so (in a very contrived fashion to in order to illustrate dispatch). See [printable_number_dispatch.hs](scratch/printable_number_dispatch.hs)):
    ```haskell
    module PrintableNumberDispatch where

    class PrintableNumber a where
      convertNumberToString :: a -> String

    newtype Integerish = Integerish Integer deriving (Eq, Show)
    newtype Doublish = Doublish Double deriving (Eq, Show)

    instance PrintableNumber Integerish where
      convertNumberToString x = showIntegerish x

    instance PrintableNumber Doublish where
      convertNumberToString x = showIntegerish x

    concatPrintableNumbers :: PrintableNumber a => a -> a -> String
    concatPrintableNumbers x y = concat [(convertNumberToString x), ", ", (convertNumberToString y)]

    ```
    * Within the REPL:
    ```haskell
    ghci> :l scratch/printable_number_dispatch.hs
    [1 of 1] Compiling PrintableNumberDispatch ( scratch/printable_number_dispatch.hs, interpreted )
    ghci> :t (concatPrintableNumbers (Integerish 1))
    (concatPrintableNumbers (Integerish 1)) :: Integerish -> String
    ghci> :t (concatPrintableNumbers (Doublish 1))
    (concatPrintableNumbers (Doublish 1)) :: Doublish -> String
    ```
    * The above is a truly horrid example. The function `show` does pretty much what we want against any type constrained by `Show a`, whereas we've defined **two** operations each limited to a concrete type.
    * We implement the contrived examples [instance_dispatch.hs](scratch/instance_dispatch.hs) and [instance_dispatch2.hs](scratch/instance_dispatch2.hs) in the text to further illustrate.

## Gimme more operations

* Very few operations that can be performed on unconstrained polymorphism yields. Put another way, there are few operations that can be defined **regardless** of type. I need to see if I can prove this, but I believe they'd all fall in the category of or would be isomorphic too:
    - List operations
    - Tuple operations
* Operations can be grouped under type classes that collect constrained types and define instances thereafter. Consider this example:
    ```haskell
    module ProductBroken where

    mult :: a -> a -> a
    mult = (*) -- this is point-free notation
    ```
    * Attempting to load [scratch/product_broken.hs](scratch/product_broken.hs) will result in an error and admonition to constrain `mult` to type class `Num a` (Haskell is *very* smart, and it's smart because its type ecosystem makes it easier to reason about several classes of errors and provide helpful suggestions).
    ```haskell
    ghci> :l scratch/product_broken.hs
    [1 of 1] Compiling ProductBroken    ( scratch/product_broken.hs, interpreted )

    scratch/product_broken.hs:4:8: error:
        • No instance for (Num a) arising from a use of ‘*’
          Possible fix:
            add (Num a) to the context of
              the type signature for:
                mult :: a -> a -> a
        • In the expression: (*)
          In an equation for ‘mult’: mult = (*)
    Failed, modules loaded: none.

    ```
    * The fixed version:
    ```haskell
    module ProductFixed where

    mult :: Num a => a -> a -> a
    mult = (*) -- this is point-free notation
    ```
* If we were to use concrete types in our function definition, we would inherit all the operations for every instance that covers said type under a type class.  For example, the following compiles because `(*)` is defined for `Int` in whatever relevant type classes define instances for `Int`:
    ```haskell
    mult :: Int -> Int -> Int
    mult = (*)
    ```


## Additional Reading

1. ["Type class"](https://en.wikipedia.org/wiki/Type_class), [Wikipedia](https://en.wikipedia.org)
1. ["OOP vs type classes"](https://wiki.haskell.org/OOP_vs_type_classes), [Haskell Wiki](https://wiki.haskell.org)
1. ["Research Papers: Type classes"](https://wiki.haskell.org/Research_papers/Type_systems#Type_classes), [Haskell Wiki](https://wiki.haskell.org)
1. Lipovača, Miran, ["Types and Type Classes"](http://learnyouahaskell.com/types-and-typeclasses), [Learn You A Haskell for Great Good!](https://http://learnyouahaskell.com/)
1. ["Tuples"](https://www.haskell.org/onlinereport/basic.html#basic-tuples), [Haskell 98 Language and Libraries: The Revised Report](https://www.haskell.org/onlinereport/index.html), December 2002
1. ["Numbers"](https://www.haskell.org/tutorial/numbers.html), [A Gentle Introduction to Numbers](https://www.haskell.org/tutorial/index.html), June 2000
