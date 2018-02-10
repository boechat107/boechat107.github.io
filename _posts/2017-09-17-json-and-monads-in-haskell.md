---
layout: single
title: "JSON and Monads in Haskell"
description: "Simple example of using Haskell to read JSON files."
category: Programming
tags: [Haskell, JSON]
mathjax: true
lang:
trans:
---


The problem is simple, but it took me some hours to figure it out. I had a JSON
file composed of a single dictionary where the values were lists of numbers,
discrete and continuous, and I wanted to keep the different number types while
reading it in Haskell.

{% highlight json %}
{
    "var1": [1, 5],
    "var2": [22.5, 50],
    "var3": []
}
{% endhighlight %}

This was a challenge to me, since my practical Haskell experience had been
around implementing some classical data structure algorithms.

# Structuring the solution

From [Artyom's tutorial about Aeson](https://artyom.me/aeson#unknown-field-names),
I had the feeling that my main problem would be to define a type to describe the
numerical values inside the arrays. I started from there.

{% highlight haskell %}
data BinVal = BinValI Int | BinValF Float deriving (Show)
{% endhighlight %}

The name `BinVal` comes from the fact that those numbers represent bin edges for
data binning/categorization.

From Aeson, the most important function to me would be the `decode`:

{% highlight haskell %}
import qualified Data.Aeson as Json
import qualified Data.Aeson.Types as JT

λ> :t Json.decode
Json.decode :: JT.FromJSON a => Bsl.ByteString -> Maybe a

λ> :info JT.FromJSON
class JT.FromJSON a where
  JT.parseJSON :: JT.Value -> JT.Parser a
  ...
{% endhighlight %}

As discussed in the tutorial and seen from the type information, `parseJSON` is
another important function. At this point I knew that a good start would be
making the function `decode` to work for `BinVal`:

{% highlight haskell %}
-- This result I got only after finding my solution.
λ> Json.decode "10" :: Maybe BinVal
Just (BinValI 10)
{% endhighlight %}

I needed to make `BinVal` a member of the type class `FromJSON` and here was the
part that took me a lot of time.

{% highlight haskell %}
instance Json.FromJSON BinVal where
  parseJSON = -- Something I didn't understand.

λ> :t Json.parseJSON
Json.parseJSON :: JT.FromJSON a => JT.Value -> JT.Parser a

λ> :info JT.Value
data JT.Value
  = JT.Object !JT.Object
  | JT.Array !JT.Array
  | JT.String !T.Text
  | JT.Number !Data.Scientific.Scientific
  | JT.Bool !Bool
  | JT.Null
{% endhighlight %}

OK, understood something here. I wanted to handle the numeric case of `Value`
and it seemed that I needed to return something of the type `Parser BinVal`.
I spent a lot of time messing around `Json.withScientific` trying to make
`parseJSON` return a `Parser BinVal` and I was really lost with the types and the
tutorial examples.
[This gist](https://gist.github.com/nbogie/985645) helped me to understand that
handling a specific subtype of `Value` didn't need to be complicated, my
`parseJSON` could start like this:

{% highlight haskell %}
instance Json.FromJSON BinVal where
  parseJSON (Json.Number v) = ...
{% endhighlight %}

Cool, I didn't need `Json.withScientific`.
[Hoogle](https://www.haskell.org/hoogle/?hoogle=scientific) also helped to find
the function `floatingOrInteger` of the package **scientific**. The code was
going somewhere:

{% highlight haskell %}
import Data.Scientific (floatingOrInteger)

instance Json.FromJSON BinVal where
  parseJSON (Json.Number v) = case floatingOrInteger v of
    Left f -> -- Something like (BinValF f)
    Right i -> -- Something like (BinValI i)
{% endhighlight %}

I felt I was very close, but I still didn't know how to generate a `Parser`. I
thought about many things, tried to imagine some kind of recursion or some other
complicated typed messed stuff. Seeing the mention of `mapM` in the tutorial
made everything even more confusing at the beginning, but it actually pointed me
to the right direction.

I had noticed that `Parser` was some kind of type "container", but I didn't
realized it was a monadic type (actually, I didn't read the tutorial slowly
enough to pay attention to this). After noticing the `mapM` part, I knew that I
needed to put a `BinValF f` and a `BinValI i` inside the monadic type `Parser`.

I started to carefully read again the tutorial. The amount of times I saw
`return` made me wonder about its type signature:

{% highlight haskell %}
λ> :t return
return :: Monad m => a -> m a
{% endhighlight %}

Voilà! That is the answer! `return` could take a type and wrap it with a monadic
container.

{% highlight haskell %}
instance Json.FromJSON BinVal where
  parseJSON (Json.Number v) = case floatingOrInteger v of
    Left f -> return (BinValF f)
    Right i -> return (BinValI i)

λ> Json.decode "10" :: Maybe BinVal
Just (BinValI 10)
λ> Json.decode "15.2" :: Maybe BinVal
Just (BinValF 15.2)
{% endhighlight %}

# Complete parser

From the tutorial's section
["Unknown field names"](https://artyom.me/aeson#unknown-field-names),
I defined the complete structure of my JSON file as the simple type alias
`Bins`:

{% highlight haskell %}
import qualified Data.HashMap.Strict as HM
import qualified Data.Text as T

data BinVal = BinValI Int | BinValF Float deriving (Show)
type Bins = HM.HashMap T.Text [BinVal]

λ> Json.decode "{\"abc\": [10]}" :: Maybe Bins
Just (fromList [("abc",[BinValI 10])])
{% endhighlight %}

Cool! We are ready to go!

The complete solution looked like this:

{% highlight haskell %}
{-# LANGUAGE FlexibleContexts #-}
{-# LANGUAGE DeriveGeneric #-}

module Main where

import qualified Data.ByteString.Lazy as Bsl
import qualified Data.HashMap.Strict as HM
import qualified Data.Aeson as Json
import qualified Data.Aeson.Types as JT
import Data.Scientific (floatingOrInteger)
import qualified Data.Text as T

data BinVal = BinValI Int | BinValF Float deriving (Show)
type Bins = HM.HashMap T.Text [BinVal]

instance Json.FromJSON BinVal where
  parseJSON (Json.Number v) = case floatingOrInteger v of
    Left f -> return (BinValF f)
    Right i -> return (BinValI i)

readJsonFile :: FilePath -> IO Bins
readJsonFile fpath = do content <- Bsl.readFile fpath
                        case Json.decode content of Just x -> return x
{% endhighlight %}

# References

* [https://artyom.me/aeson](https://artyom.me/aeson)
* [http://learnyouahaskell.com/chapters](http://learnyouahaskell.com/chapters)
* [http://hackage.haskell.org/package/aeson-1.2.1.0/docs/Data-Aeson-Types.html](http://hackage.haskell.org/package/aeson-1.2.1.0/docs/Data-Aeson-Types.html)
* [https://gist.github.com/nbogie/985645](https://gist.github.com/nbogie/985645)
* [https://www.haskell.org/hoogle/?hoogle=scientific](https://www.haskell.org/hoogle/?hoogle=scientific)
* [https://en.wikibooks.org/wiki/Haskell/Applicative_functors](https://en.wikibooks.org/wiki/Haskell/Applicative_functors)
