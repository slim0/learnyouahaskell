## GHCI commands

- `:l` : **Load** the given haskell file
- `:r` : **Reload** the previously loaded haskell file
- `:t` : Get the **type** of a value (function is also a value)
- `:k` : Get the **kind** of a type (`*` means 'concrete type')
- `:i` : Get **info** about the given type of typeclass. It can be usefull if you want to see what the instances of a typeclass are, or to get the 'MINIMAL' complete definition of a typeclass for example.

```sh
ghci> :k Int
Int :: *

ghci> :k Maybe
Maybe :: * -> *
```

## GHC commands

- `ghc --make program.hs` : Compile the `program.hs` file. Run `./program` to run it after that.
> To run a program, you can also use the `runhaskell` command like so: `runhaskell program.hs` and your program will be executed on the fly.

## Infix functions

Functions that takes 2 parameters can be used as an infix function.

i.e, this function `let sum x y = (x + y)` can be used :
- normally: `sum 10 20`
- or as an infix function using backticks: ```10 `sum` 20```

## Function composition

> In mathematics, function composition is defined like this: `(f . g)(x) = f(g(x))`, meaning that composing two functions produces a new function that, when called with a parameter, say, x is the equivalent of calling g with the parameter x and then calling the f with that result.
> In Haskell, function composition is pretty much the same thing. We do function composition with the . function, which is defined like so:

```haskell
(.) :: (b -> c) -> (a -> b) -> a -> c
f . g = \x -> f (g x)
```

i.e, the expression `negate . (* 3)` returns a function that takes a number, multiplies it by 3 and then negates it.

i.e, those two functions are equivalent:

```haskell
fn x = ceiling (negate (tan (cos (max 50 x))))

fn' = ceiling . negate . tan . cos . max 50
```

As you can see, it is sometimes much more readable to use function composition instead of lot of parentheses.

When function composition is used, you can read from right to left.

## Let bindings

> They have to be in the form of `let bindings in expression`, where `bindings` are names to be given to expressions and `expression` is the expression that is to be evaluated that sees them.

In list comprehension, the `in` part isn't needed, just like in a `do` block.

i.e :

```haskell
import Data.Char

main = do
    putStrLn "What's your first name?"
    firstName <- getLine
    putStrLn "What's your last name?"
    lastName <- getLine
    let bigFirstName = map toUpper firstName
        bigLastName = map toUpper lastName
    putStrLn $ "hey " ++ bigFirstName ++ " " ++ bigLastName ++ ", how are you?"
```

## Case statement

```haskell
case expression of pattern -> result
                   pattern -> result
                   pattern -> result
```
## Types

**A type is a set of values**
ex:
- Int (-100, 0, 12, 21, ...)
- Float (-12.12, 3.14, ...)

To define a new type, use the `data` keyword.

In the following example:
- `Circle` and `Rectangle` are called 'data constructors' or 'value constructors' that takes respectively 3 and 4 Floats.
- The part before the `=` sign is called the 'type constructor'. In this case, because it has no parameters, `Shape` is a 'nullary type constructor' (or simply a 'type')

```haskell
data Shape = Circle Float Float Float | Rectangle Float Float Float Float
```

Here is an exemple of a well known 'type constructor':

```haskell
data Maybe a = Nothing | Just a
```

In this case, `Maybe` is called a 'type constructor' because it involved a `a` type parameter that can hold different types (**Type parameters has to be lowercase words!**)

No value can have a type of just `Maybe`, because that's not a type per se, it's a 'type constructor'!

> Using type parameters is very beneficial, but only when using them makes sense. Usually we use them when our data type would work regardless of the type of the value it then holds inside it, like with our Maybe a type

A type can be made an instance of a typeclass:
- by derinving the typeclass
```haskell
data Shape = Circle Float Float Float | Rectangle Float Float Float Float deriving (Show)
```
- or manually using the `instance` keyword
```haskell
data TrafficLight = Red | Yellow | Green

instance Eq TrafficLight where
    Red == Red = True
    Green == Green = True
    Yellow == Yellow = True
    _ == _ = False
```

### Type synonyms using `type` keyword

Type synonyms don't really do anything per se, they're just about giving some types different names so that they make more sense to someone reading our code and documentation.
Here's how the standard library defines `String` as a synonym for `[Char]`:

```haskell
type String = [Char]
```

And here is a custom example:

```haskell
type PhoneNumber = String
type Name = String
type PhoneBook = [(Name,PhoneNumber)]

inPhoneBook :: Name -> PhoneNumber -> PhoneBook -> Bool
inPhoneBook name pnumber pbook = (name,pnumber) `elem` pbook
```

## Typeclasses

**A typeclass is a set of types. It is a sort of interface that defines some behavior**

ex:
- Num (Int, Float...)

To create a new typeclass, use the `class` keyword. Here is the definition of the `Eq` class:
```haskell
class Eq a where
    (==) :: a -> a -> Bool
    (/=) :: a -> a -> Bool
    x == y = not (x /= y)
    x /= y = not (x == y)
```

>The `a` is the type variable and it means that a will play the role of the type that we will soon be making an instance of `Eq`. It doesn't have to be called `a`, it doesn't even have to be one letter, it just has to be a lowercase word. Then, we define several functions.

You can also make typeclasses that are subclasses of other typeclasses. The class declaration for Num is a bit long, but here's the first part:

```haskell
class (Eq a) => Num a where
   ...
```

The part before the `=>` sign is called a "constraint" and in this case more specifically a "class constraint". Here, it says that `a` must be an instance of `Eq`.

Constraints are also used when declaring types or instance but you must **never add typeclass constraints in data declarations!**

## Inputs and Outputs

To extract a result from an IO action, you'll have to use the `<-` sign.

For example, `name <- getLine` wait for the `getLine` to return and bind the result to the `name` variable.

> I/O actions will only be performed when they are given a name of main or when they're inside a bigger I/O action that we composed with a do block. We can also use a do block to glue together a few I/O actions and then we can use that I/O action in another do block and so on. Either way, they'll be performed only if they eventually fall into main.


`my_string <- getLine` make sense because the type signature of the `getLine` function is: `getLine :: IO String`. So we are extracting a `String` from the `IO` action.
`nothing <- putStrLn "toto"` make no sense because the type signature of the `putStrLn` function is: `putStrLn :: String -> IO ()`. We can see here that the result encapsulated inside the `IO` action is `()` (empty tuple). So it makes no sense to extract it's result.

> A common pattern with sequence is when we map functions like print or putStrLn over lists. Doing `map print [1,2,3,4]` won't create an I/O action. It will create a list of I/O actions, because that's like writing `[print 1, print 2, print 3, print 4]`. If we want to transform that list of I/O actions into an I/O action, we have to sequence it: `sequence (map print [1,2,3,4])`

## Return keyword

**the `return` in Haskell is really nothing like the `return` in most other languages!**
In Haskell (in I/O actions specifically), it makes an I/O action out of a pure value. So in an I/O context, `return "haha"` will have a type of IO String.

**Using `return` doesn't cause the I/O do block to end in execution or anything like that!**

`return` is sort of the opposite to `<-`. While `return` takes a value and wraps it up in a box, `<-` takes a box (and performs it) and takes the value out of it, binding it to a name.

## Exceptions

Both pure code and I/O code can throw exceptions, but exceptions can only be caught in the I/O part of our code ((when we're inside a do block that goes into main)! That's because you don't know when (or if) anything will be evaluated in pure code, because it is lazy and doesn't have a well-defined order of execution, whereas I/O code does.

> Pure functions are lazy by default, which means that we don't know when they will be evaluated and that it really shouldn't matter. However, once pure functions start throwing exceptions, it matters when they are evaluated. That's why we can only catch exceptions thrown from pure functions in the I/O part of our code. And that's bad, because we want to keep the I/O part as small as possible. However, if we don't catch them in the I/O part of our code, our program crashes. The solution? Don't mix exceptions and pure code. Take advantage of Haskell's powerful type system and use types like Either and Maybe to represent results that may have failed.

# Try/catch exceptions

```haskell
import Control.Exception
import System.Environment
import System.IO
import System.IO.Error

main = toTry `catch` handler

toTry :: IO ()
toTry = do
  (fileName : _) <- getArgs
  contents <- readFile fileName
  putStrLn $ "The file has " ++ show (length (lines contents)) ++ " lines!"

handler :: IOError -> IO ()
handler e
  | isDoesNotExistError e = putStrLn "The file doesn't exist!"
  | otherwise = ioError e
```

`isDoesNotExistError` is called a predicate (over `IOError`), which means that it's a function that takes an IOError and returns a True or False.

In the handler, we are using "guard" syntax to catch different exceptions, but we could have also used an `if else`.

If the exception is not catch, we go to `otherwise` and use `ioError` to re-throw the exception that was passed by the handler with the `ioError` function.