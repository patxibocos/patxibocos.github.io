---
layout: post
title: Intro to Higher Kinded Types in Haskell
---

Higher Kinded Types (HKT) is known in Haskell as the type of types. In some way they give us the power of generics in Java but with much more flexibility and seamlessly integrated in the language. Why? Let's see it through code.

A `List<T>` in Java is a list that contains things of T type. A `Map<K, V>` will contain keys of K type mapped to values of V type. The meaning of those generics in this particular case is that they allow us to define the specific type of our list or map, so we can define a List<String>, List<User>, Map<Long, User>, etc.

This is useful as we have a container (the list or map) which can hold any type of items inside. We don't need a different type for each specific contained type because they are parameterized.

That generic in Java is telling us: "in order to build a list, you must provide the type for contained items". In Haskell this concept is called kind of a type, which basically specifies if a type expects more types in order to be built (they behave as functions for types construction). For types that do not construct other types, the kind is said to be `*`. In the case of a list, kind is `* -> *` (following the arrow you can understand that when you give a type, you get another one).

But... Can you define in Java something like "any container holding a String"? This should be like generics but reversed: if this would be possible, it could be expressed as T<String>. In Haskell this is represented as `a String`. You may understand this as a type constructor which is parameterized with a String. Let's see this in detail by an example in Haskell.

Let's say we have a User data type:

```
data User = User
  { name :: String
  , age  :: Int
  }
```

and a function that receives a given user name and returns a Maybe of User:

```
userByName :: String -> Maybe User
userByName name
  | name == "haskell" = Just $ User name 29
  | otherwise = Nothing
```

This function receives a String and in this simple case, if it is equals to "haskell" then returns a Maybe value with a User, and Nothing value otherwise.

So now we are going to define a new function which will receive a String and then look for an optional User and return its age in case the user exists. This function will basically use the previous one and then obtain user's age if it exists. We could think on doing this:

```
ageForUser :: String -> Maybe Int
ageForUser name = fmap age (userByName name)
```

This works and it's fine, but this function is needlessly coupled to Maybe. Does this function really need to now about it? 

We want to have something like this:

```
ageForUser :: String -> a Int
ageForUser name = fmap age (userByName name)
```

But this will not compile, here we are trying to say: given a String parameter, we want to return a Maybe String in our particular case (`a` will be the Maybe "hole"). But we cannot define the function this way as the function return type is open for every type whose kind is (* -> *). A client of this function will not have any way to know the concrete type, that's why the compiler even does not allow us doing it. In some way the function type signature does not meet referential transparency (at type level).

What we are going to do is receive the userByName function as a parameter, so that it will bring us the type:

```
ageForUser :: String -> (String -> a User) -> a Int
ageForUser name userFinder = fmap age (userFinder name)
```

But this does not still work :( Why? The problem now is that we cannot call fmap function for a type that does not meet the Functor type class. Let's put the constraint then:

```
ageForUser :: Functor a => String -> (String -> a User) -> a Int
ageForUser name userFinder = fmap age (userFinder name)
```

And now it compiles (and works!). Can you see the potential of both higher kinded types and type classes?
