# Control Flow

There are type classes that control the flow of the program (e.g. whether the program should do X and then Y or should do X and Y at the same time).

## Functor, Apply, and Bind Type Classes Explained in Pictures

We've linked to an article below that explains these abstract notions in a clear manner using pictures and the `Maybe a` data structure. However, since these concepts are explained in Haskell, which uses different terminology than Purescript, use the following table to `map` Haskell terminology to Purescript terminology:

| Haskell Terminology | Purescript Terminology |
| --- | --- |
| `fmap` (function) | `map` (function) |
| `Applicative` (type class) | `Apply` (type class) |
| `Array`/`[]` (syntax sugar for `List a`) | `List a` |
| `map` (Array function) | see [the implementation in Purescript](#lists-map-function-in-purescript) |
| `->` (syntax sugar for `Function`) | `Function` |
| `IO ()` | `Effect Unit`, which will be explained/used in a later part of this folder |

Here's the link to the article:
http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html

### Lists' Map Function in Purescript

Here's the `map` List function implemented in Purescript:
```purescript
data List a = Nil | Cons a (List a)

instance Functor List where
  map :: forall a b. (a -> b) -> List a -> List b
  map f Nil = Nil
  map f (Cons head tail) = Cons (f head) (map f tail)
```

## Functor, Apply, Applicative, Bind, Monad

| Typeclass | "Plain English" | Function | Infix | Laws | Usage
| -- | -- | -- | -- | -- | -- |
| [Functor](https://pursuit.purescript.org/packages/purescript-prelude/4.1.0/docs/Data.Functor) | Mappable | `map :: forall a b. (a -> b) -> f a -> f b` | `<$>` <br> (Left 4) | <ul><li>identity: `map (\x -> x) fa == fa`</li><li>composition: `map (f <<< g) = map f <<< map g`</li></ul> | Given a box-like type, `f`, with a value(s) of some type, `a`, change the `a` to `b` without changing the box-like type itself. |
| [Apply](https://pursuit.purescript.org/packages/purescript-prelude/4.1.0/docs/Control.Apply) | Boxed Mappable | `apply :: forall a b. f (a -> b) -> f a -> f b` | `<*>` <br> (Left 4) | <ul><li>Associative composition: `(<<<) <$> f <*> g <*> h == f <*> (g <*> h)`</li></ul> | Given a box-like type, `f`, with value(s) of some type, `a`, change the `a` to `b` without changing the box-like type itself. |
| [Applicative](https://pursuit.purescript.org/packages/purescript-prelude/4.1.0/docs/Control.Applicative) | Liftable <br> Parallel Computation | `pure :: forall a. a -> f a` |  | <ul><li>identity: `(pure (\x -> x) <*> v == v)`</li><li>composition: `pure (<<<) <*> f <*> g <*> h == f <*> (g <*> h)`</li><li>Homomorphism: `(pure f) <*> (pure x) == pure (f x)`</li><li>interchange: `u <*> (pure y) == (pure (_ $ y)) <*> u`</li></ul> | Put a value into a box <br> Run code in parallel |
| [Bind](https://pursuit.purescript.org/packages/purescript-prelude/4.1.0/docs/Control.Bind) | Chainable | `bind :: forall m a b. m a -> (a -> m b) -> m b` | `>>=` <br> ()| Associativity: `(x >>= f) >>= g == x >>= (\x' -> f x' >>= g)` | Given an instance of a box-like type, `m`, that contains a value, `a`, extract the `a` from `f`, and create a new `m` instance that stores a new value, `b`. <br> Take `f a` and compute it via `bind` to produce a value, `a`. Then, use `a` to describe (but not run) a new computation, `m b`. When `m b` is computed (via a later `>>=`), it will return `b`. |
| [Monad](https://pursuit.purescript.org/packages/purescript-prelude/4.1.0/docs/Control.Monad) | Sequential Computation | | | | The data structure used to run FP programs by executing code line-by-line, function-by-function, etc. |

## Simplest Useless Monad Implementation

```purescript
data Box a = Box a

instance f :: Functor Box where
  map f (Box a) = Box (f a)

instance a1 :: Apply Box where
  apply (Box f) (Box a) = Box (f a)

instance a2 :: Applicative Box where
  pure a = Box a

instance b :: Bind Box where
  bind (Box a) f = f a

-- no need to implement a Monad instance as
-- the compiler will infer that it is possible
-- since Bind and Applicative have been defined
```

## Monad laws re-examined

Another way to think about the laws for Monad are:
```purescript
-- given a function whose type signature is...
(>=>) :: Monad m => (a -> m b) -> (b -> m c) -> a -> m c
(aToMB >=> bToMC) a = aToMB a >>= (\b -> bToMC b)

-- and Monad could be defined by these laws:
(function >>> id) a == function a -- Functor's identity law
 aToMB    >=> pure  == aToMB

-- and its inverse
(id   >>> f) a == f a
 pure >=> f    == f

-- and function composition
f >>> (g >>> h) == (f >>> g) >>> h -}
f >=> (g >=> h) == (f >=> g) >=> h
```
This was taken from [this slide in this YouTube video](https://youtu.be/EoJ9xnzG76M?t=7m9s)

## Do notation

At this point, you should look back at the Syntax folder to read through the file on `do notation`. You should also become familiar with the `ado notation` (Applicative Do).

Be aware of where the parenthesis appear when using `m a >>= aToMB >>= bToMC` by reading the section called "Do notation" in [this article](https://sras.me/haskell/miscellaneous-enlightenments.html). Below provides a summary of what it says:
```purescript
data Maybe a = Nothing | Just a

instance b :: Bind Maybe where
  bind (Just a) f = f a
  bind Nothing f = nothing

half :: Int -> Maybe Int
half x | x % 2 == 0 = x / 2
       | otherwise = Nothing

-- This statement
(Just 128) >>= half >>= half >>= half == Just 16
-- desugars first to
(Just 128) >>= (\original -> half original >>= half >>= half ) == Just 16
-- which can be better understood as
(Just 128) >>= aToMB == Just 16
-- since the latter >>= calls are nested inside of the first one, as in
-- "Only continue if the previous `bind` call was successful."

-- Similarly
Nothing    >>= half >>= half >>= half == Nothing
-- desguars first to
Nothing    >>= (\value -> half value >>= half >>= half) == Nothing
-- which can be better understood as
Nothing    >>= aToMB == Nothing
-- and, looking at the instance of Bind above, reduces to Nothing

-- Thus
half3Times :: Maybe Int -> Maybe Int
half3Times maybeI = do
  original <- maybeI
  first <- half original    -- ===
  second <- half first      --  | a -> m b
  third <- half second      --  |
  pure third                -- ===
-- doesn't actually run when passed Nothing as its argument
```
