---
title: Algebraic Data Types
separator: ---
verticalSeparator: <===>
theme: solarized
highlightTheme: atom-one-dark
revealOptions:
  transition: 'slide'
css: './resources/style.css'
---

## Algebraic Data Types

<br />
<br />
<br />
> by Giorgi Bagdavadze / @notgiorgi

---

![Comrade](./resources/lenin.jpg)

---

## Product Types

```ts
class Pair<U, V> {
    constructor(
        x: U,
        y: V,
    ) { /*...*/ }
}
```

```haskell
data Pair u v = Pair u v
```

Note: Products of types represent a conjunction, “and,” of those types.

<===>

```ts
class Person {
  constructor(
    name: String,
    age: Number,
    address: String,
  ) { /* ... */ }
}

class Box<T> {
  constructor(private value: T) { /* ... */ } 
}
```

```haskell
data Person = Person
  { name    :: String
  , age     :: Int
  , address :: String
  }

data Box a = B a
```

<===>

## Intiution of haskell

```ts
class Pair<U, V> {
    constructor(/* .. */) {}
}

let pair = new Pair<Boolean, String>(false, "Foo")
```

```haskell
data Pair u v = Pair u v
--   ^ Type     ^ Constructor

-- Type
pair :: Pair Bool String
-- Value
pair = Pair False "Foo"
```

<===>

### Why Algebraic? Why Product?

<===>

### Symmetrical

```
a * b == b * a
```

```ts
Pair<Boolean, String> ~~ Pair<String, Boolean>
```

```haskell
Pair Bool String ~~ Pair String Bool
```

Note: You could say, this is a mathematical proof, that the order you define members of your data structure in, doesn’t matter.

<===>

### Isomorphism

```ts
A ~~ B iff (A -> B && B -> A)
```

```ts
function swap<U, V>(pair: Pair<U, V>): Pair<V, U> {
  return new Pair<V, U>(pair.y, pair.x)  
}
```

<===>

### Associative

```
a * (b * c) == (a * b) * c == a * b * c
```

```csharp
Pair<bool, Pair<int, string>>
~~
Pair<Pair<bool, int>, string> 
~~
Triplet<bool, int, string>
```

```
Pair Bool (Pair Int String) ~~ Pair (Pair Bool Int) String
```

<===>

### Identity

```
a * 1 == a == 1 * a
```

```ts
Pair<Boolean, Unit> ~~ Boolean
```


```haskell
data Unit = Unit

Pair Bool Unit ~~ Bool
```

Note: Describe Unit

---

## Sum (Union) Types

```ts
enum PromiseState = {
  Pending,
  Fulfilled,
  Rejected,
}
```

```haskell
data PromiseState = Pending | Fulfilled | Rejected
```

<===>

```ts
interface Either<U, V> {/* ... */}
class Right<V> implements Either {/* ... */}
class Left<U> implements Either {/* ... */}

interface List<T> {/* ... */}
class Nil {/*...*/}
class Cons<T> {/* ... */}

type Paragraph = string | Array<string>
```

```haskell
data Either u v = Left u | Right v

data List a = Nil | Cons a (Array a)

data Paragraph = String | Array String
```

Note: Paragraph abstracts away low level types, You only think about it once

<===>

## Same Properties

```haskell
a + b == b + a

Right a | Left b ~~ Left b | Right a


a + (b + c) == (a + b) + c

Pending | (Fulfilled | Rejected) ~~ (Pending | Fulfilled) | Rejected


a + 0 = a = 0 + a

Foo a | Bar Void ~~ Foo a
```

---

## Mixed

```haskell
data Promise u v
  = Pending (Array Listener)
  | Rejected (Array Listener) u
  | Fulfilled (Array Listener) v
```

<===>

## Distributive property

```haskell
data Promise a b =
  Promise (Array Listener) (
      Pending
    | Rejected a
    | Fulfilled b
  )
```

---

## Pattern Matching

```js
switch (true) {
  case promise instanceof Fulfilled:
    console.log('Fulfilled!', promise.value)
    break
  case promise instanceof Rejected:
    console.log('Rejected!', promise.error)
    break
  default:
    throw new TypeError("Lol, ain't gonna happen")
}
```

Note: Misses `Pending` case

<===>

## "Fixed"

```js
switch (true) {
  case promise instanceof Pending:
    console.log('Pending!', promise.value)
    break
  case promise instanceof Fulfilled:
    console.log('Fulfilled!', promise.value)
    break
  case promise instanceof Rejected:
    console.log('Rejected!', promise.error)
    break
  default:
    throw new TypeError("Lol, ain't gonna happen")
}
```

Note: gets non-existent `value` property on `Pending` and doens't know in compile time

<===>

## No switches, Same problems

```ts
interface SumType {
  matchCase(cases: object): any
}

promise.matchCase({
  Pending: () => console.log('Pending'),
  Rejected: error => console.log(error),
  Fulfilled: value => console.log(value),
})
```

<===>

## Actually fixed!

```haskell
printPromise promise =
  case p of
    Pending -> putStrLn "Pending!"
    (Fulfilled value) -> putStrLn "Fulfilled!" ++ value
    (Rejected error) -> putStrLn "Rejected!" ++ error
```

### Compiler is a cool guy

- Cannot write non-total function
- Cannot construct illegal promise
- Cannot incorrectly pattern match


Note: cannot construct illegal Promise, cannot pattern match illegally, notified in compile time B)

<===>

## Cool stuff

```haskell
fibonacci :: Int -> Int
fibonacci 0 = 1
fibonacci 1 = 1
fibonacci x = fibonacci (x - 1) + fibonacci (x - 2)
```

```haskell
head :: Array a -> Maybe a
head []      = Nothing
head [x]     = Just x -- not needed, but it's cool you can do that
head (x: xs) = Just x
```

Note: dude wtf is Just/Nothing?

---

## Declare with Invariants!

```haskell
data Tree 
  = Empty
  | Leaf Int
  | Node Tree Tree

depth :: Tree -> Int
depth Empty = 0
depth (Leaf n) = 1
depth (Node l r) = 1 + max (depth l) (depth r)
```

Note: Defining data structure in such a way using only type system, that you cannot construct an instance of that type that violates that invariant

<===>

## Declare with Invariants!

```haskell
type alias Field =
  Field
   { fieldName : String
   , selectionStatus: SelectionStatus
   }

type SelectionStatus
    = Immutable
    | Unselected
    | Selected FieldRequirement

type FieldRequirement
    = Required
    | Optional
```

<===>

## if (x != null) - Bad

```ts
class Vector {
  /* ... */
  divide(that: Vector): Vector {
    if (that.x !== 0 || that.y !== 0)
      return new Vector(this.x / that.x, thix.y / that.y)
    return null
  }
}

/* ... */
const v3 = v1.divide(v2)
if (v3 !== null) {
  v3.add(/* ... */)
}
```
<===>

## Better

```ts
type Vector = NullVector | FullVector

class FullVector {
  /*...*/
  divide(that: Vector): Vector {
    if (that.x !== 0 || that.y !== 0)
      return new FullVector(this.x / that.x, thix.y / that.y)
    return new NullVector()
  }
}

class NullVector {
  /*...*/
  divide(that: Vector): Vector { return this }
}

const v3 = v1
  .divide(v2)
  .add(v5)
  .render()
```

<===>

## The best

```ts
type Maybe<T> = Just<T> | Nothing

class Vector {
  divide(that: Vector): Maybe<Vector> {
    if (that.x !== 0 || that.y !== 0)
      return new Just(
        new Vector(this.x / that.x, thix.y / that.y)
      )
    return new Nothing()
  }
}


const v3 = v1
  .divide(v2)
  .chain(v => v.multiply(2))
  .matchCase(
    v => console.log(v.x, v.y),
    () => console.log('Error')
  )
```

<===>

```haskell
data Route
  = Home
  | Post Int
  | Posts
  | User Int
  | UserPost Int Int
  | Search String
  | NotFound

parseUrl :: Url -> Route
parseUrl (Url "/Home" []) = Home
parseUrl (Url "/Posts/:id" [id]) = Post id
parseUrl (Url "UserPosts/:uID/:pID" [uID, pID]) = UserPost uID pID
parseURl (Url "*") = NotFound
-- etc

requestHandler :: Route -> IO Response
requestHandler Home = homepage
requestHandler (Post id) = postPage id
requestHandler NotFound = notFoundPage
-- etc

Server :: Request -> IO Response
Server = requestHandler . parseUrl
```