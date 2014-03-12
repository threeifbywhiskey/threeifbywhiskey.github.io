---
layout: post
title: "Non-Alphanumeric Ruby for Fun and Not Much Else"
description: "In which I write FizzBuzz in Ruby without using any numbers or letters."
category: 
comments: false
tags: []
---
{% include JB/setup %}

Inspired by Tom Stuart's [*Programming with Nothing*](http://codon.com/programming-with-nothing), I wanted to determine if there were some other way to push Ruby to its bare limits. While it's certainly not as impressive as writing FizzBuzz using nothing but `Proc`s, I did manage to discover that it's possible to write substantial Ruby programs without the superfluity of alphanumeric characters. In this post, I'm going to present an approach to writing FizzBuzz using neither letters nor numbers, and I hope you enjoy the ride.

For starters, let's see if we can't get our hands on some numbers without explicitly typing numeric characters. We're not going to have to implement Church numerals, but we do find ourselves at the mercy of Ruby to supply us with some numeric global variables at the start of our program. Let's see what they are:

{% highlight ruby %}
global_variables.select { |g| Fixnum === eval(g.to_s) }
# [:$SAFE, :$., :$$, $-W]
{% endhighlight %}

It's not much, but maybe we can work with it. `$SAFE` and `$-W` are alphabetical non-starters, of course, and `$.`, the current line of `$stdin`, is going to be 0 at program start. That only leaves us `$$`, the process ID, but it's thankfully enough; if we divide it by itself, we're going to get a 1, the numerical genesis from which all other numbers may be derived.

So, we've got numbers, but we're probably going to want to be able to store them as well; what sort of variables are we allowed to create in such a restricted environment? Well, we can use any number of underscores, and we can prefix them with either `@` or `$` to make instance and global variables, respectively. That should be enough, but just in case we need them, there're also the anomalous `$-` and `$-_` variables that we can assign to should the need arise.

Now that we can store numbers, let's go ahead and get the ones we're going to need to write FizzBuzz.

{% highlight ruby %}
_   = $$ / $$ #  1
__  =  _ -  _ #  0
@_  =  _ +  _ #  2
$_  = @_ +  _ #  3
$-  = @_ + $_ #  5
$-_ = $- * $_ # 15
{% endhighlight %}

And now let's go about deriving characters, since we're occasionally going to need to print either `"Fizz"`, `"Buzz"`, or their sum. Conveniently, Ruby once again has our back. Let's observe the behavior of the shovel operator (`<<`) on strings:

{% highlight ruby %}
'a' << 'b' << 'c'
# "abc"
{% endhighlight %}

Logically enough, it can "shovel" strings into each other. This doesn't quite help us, of course, as we're not allowed to create explicit character strings. We can, however, shovel Unicode codepoints into strings to achieve the same effect:

{% highlight ruby %}
'' << 97 << 98 << 99
# "abc"
{% endhighlight %}

Thus, to acquire a `'z'`, we can push the number `122` into an empty string, and the pattern follows for any other characters we might need. Just to get the shape of the thing, let's build `"Fizz"` and `"Buzz"` with numeric literals:

{% highlight ruby %}
z    = '' << 122
fizz = '' << 70 << 105 << z << z
buzz = '' << 66 << 117 << z << z
{% endhighlight %}

Those alphanumerics have to go, of course, so let's derive those numbers from the ones we've already defined and change the variable names.

{% highlight ruby %}
@__  = '' << 15 * (5 + 3) + 2
$___ = '' << 15 * 5 - 5 << 15 * (5 + 2) << @__ << @__
@___ = '' << 15 * 5 - 3 * 3 << 15 * (5 + 3) - 3 << @__ << @__
{% endhighlight %}

And, lastly, we'll replace the numeric literals with the variables containing their respective values.

{% highlight ruby %}
@__  = '' << $-_ * ($- + $_) + @_
$___ = '' << $-_ * $- - $- << $-_ * ($- + @_) << @__ << @__
@___ = '' << $-_ * $- - $_ * $_ << $-_ * ($- + $_) - $_ << @__ << @__
{% endhighlight %}

So, we've got the numbers and strings that we're going to need to write FizzBuzz, but how are we going to loop from 1 to 100? We're going to take a page from Tom and use recursion, of course! In fact, a little anti-climactically, it's exactly the approach Tom takes in *Programming with Nothing*. We'll define a lambda that does the work, check a variable against 100 at the end, and re-run the lambda if it's less than 100. Ruby once again supplies the means to do this out of the box via the "stabby lambda" (`->`) syntax.

To get a feel for the approach, let's momentarily ignore the alphanumeric restriction and write a recursive FizzBuzz using a lambda.

{% highlight ruby %}
n = 0

fizzbuzz = -> {
  n += 1
  puts n % 15 == 0 ? 'FizzBuzz'
    :  n %  3 == 0 ? 'Fizz'
    :  n %  5 == 0 ? 'Buzz'
    :  n
  n < 100 ? fizzbuzz[] : 0
}

fizzbuzz[]
{% endhighlight %}

The logic is simple enough; we start a variable at 0, increment it, then print accordingly. As long as we've named our lambda, we'll be able to call it from within itself via the `#[]` alias for `#call`. Of course, we won't be able to give it such a descriptive name once we're done. In fact, let's go about replacing everything we can.

{% highlight ruby %}
___ = -> {
  $. += _
  puts $. % $-_ == __ ? $___ + @___
    :  $. % $_  == __ ? $___
    :  $. % $-  == __ ? @___
    :  $.
  $. < 100 ? ___[] : __
}

___[]
{% endhighlight %}

That was all pretty straightforward, but `$.` is a new one. As mentioned earlier, it's normally used as the current line of standard input; here, though, we've repurposed it as another `0` to play around with; since we need `__` for modulo checks, we can't increment it. And, just like that, we're almost there. We need only now to derive the `100` and figure out a way to get rid of the `puts`. Once again, Ruby finds a way: `$>` is an alias for `$stdout`, and `File`-like objects can be shoveled into. The shoveling isn't *quite* like that for strings, though, as pushing a number will print the number itself. That is, `$> << 97` prints `97`, whereas `$> << ('' << 97)` is how we'd go about printing an `'a'`. Still, that's plenty functionality for us to finish up. Below is presented the full program with a bit more commentary.

{% highlight ruby linenos %}
_   = $$  / $$ #  1
__  =  _  -  _ #  0
@_  =  _  +  _ #  2
$_  = @_  +  _ #  3
$__ = @_  + $_ #  5
$-_ = $__ * $_ # 15

@__  = '' << $-_ * ($__ + $_) + @_ # z
$___ = '' << $-_ * $__ - $__ << $-_ * ($__ + @_) << @__ << @__ # Fizz
@___ = '' << $-_ * $__ - $_ * $_ << $-_ * ($__ + $_) - $_ << @__ << @__ # Buzz

(___ = -> { # the fizzbuzz lambda
  $. += _   # increment n
  $> << ($. % $-_ == __ ? $___ + @___ # "FizzBuzz" if mod-15
       : $. % $_  == __ ? $___        # "Fizz" for 3
       : $. % $__ == __ ? @___        # "Buzz" for 5
       : $.) <<                       # Otherwise, n
       ('' << $__ * @_)               # and a newline
  $. < ($__ * @_) ** @_ ? ___[] : _   # Check n against 100.
})[] # Immediately invoke the lambda.
{% endhighlight %}

And that's that. For giggles and visual confirmation, I've presented a compacted version of the program below.

{% highlight ruby %}
_=$$/$$;__=_-_;@_=_+_;$_=@_+_;$__=@_+$_;$-_=$__*$_
@__ =''<<$-_*($__+$_)+@_
$___=''<<$-_*$__-$__<<$-_*($__+@_)<<@__<<@__
@___=''<<$-_*$__-$_*$_<<$-_*($__+$_)-$_<<@__<<@__
(___=->{$.+=_;$><<($.%$-_==__ ?$___+@___:$.%$_==__ ?$___:$.%
$__==__ ?@___:$.)<<(''<<$__*@_);$.<($__*@_)**@_?___[]:_})[]
{% endhighlight %}

I'm surprised Pygments did anything with that at all. Well, I'll sign off here, but I hope you had fun reading. There's [a repository](https://github.com/threeifbywhiskey/narfnme){:target="new"} of similar programs on GitHub if you'd like to take a peek, and if inspiration strikes, feel free to contribute a non-alphanumeric program of your own.

I'd like to thank [@HonoreDB](https://github.com/honoredb){:target="new"} for [calling my attention](http://www.reddit.com/r/ruby/comments/1vxoh3/nonalphanumeric_ruby_for_fun_and_not_much_else/){:target="new"} to a few neat tricks that I would otherwise not have remembered.
