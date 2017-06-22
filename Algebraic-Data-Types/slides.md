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

class Person {
  constructor(
    name: String,
    age: Number,
    address: String,
  ) { /* ... */ }
}

class Box<T> { constructor(private value: T) { /* ... */ } }
```

```haskell
data Pair u v = Pair u v

data Person = Person String Int
```

Note: Products of types represent a conjunction, “and,” of those types.

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
Note: Describe Unit
```haskell
Pair Boolean () ~~ Boolean
```

---

## Sum (Union) Types

```ts
enum PromiseState = {
  Pending,
  Fulfilled,
  Rejected,
}
```

```ts
interface Either<U, V> { /* ... */ }
class Right<V> implements Either { /* ... */ }
class Left<U> implements Either { /* ... */ }
```

```haskell
data PromiseState = Pending | Fulfilled | Rejected

data Either u v = Left u | Right v
```

<===>

# Same Properties

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
  = Pending [Listener]
  | Rejected [Listener] u
  | Fulfilled [Listener] v
```

<===>

## Distributive property

```haskell
data Promise a b =
  Promise [Listener] (
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

<===>

## Actually fixed!

```haskell
printPromise promise =
  case p of
    Pending -> putStrLn "Pending!"
    (Fulfilled value) -> putStrLn "Fulfilled!" ++ value
    (Rejected error) -> putStrLn "Rejected!" ++ error
```

---

```haskell
data Tree a
  = Leaf
  | Node (Tree a) a (Tree a)

find :: a -> Tree a -> Bool
find x Leaf = False
find x (Node left value right) =
  if x == value 
    then True
    else value > x ? find x right : find x left

tree = (
  Node Leaf 3 (
    Node Leaf 7 Leaf
  )
)

find 9 tree -- True
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