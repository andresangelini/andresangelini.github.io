We have come a long way since we started our coding adventure into making a [Responsive Medieval Board With SVG Stacks]. We first stepped into SVG territory where we met its inhabitants in a new light that few people have rarely seen before. We found mysterious `<symbol>`s in its long forgotten ruins which we soon discovered were meant to help us `<def>`ine our plans so that you **Don't Repeat Yourslef**. They aided us to [improve our organization], and along with proper [Namespacing], lead us to journey through [SVG in style]. We also discovered the true meaning of its hidden `<pattern>`s, allowing us to [clip] through parts of the SVG and thus create more [complex shapes], which from then on, we could easily recognize thanks to using proper [fragment identifiers]. But just when we thought we would end up drowning in the muddy waters of our own code... [Sass!], a brave new Hero came along to our rescue. An he wasn't alone either. A friendly guy called [BEM] also saw us in a pinch and junped in to help. Under the warm of a wood fire and the soft melody of his old guitar, he shared many stories and told us about how the world was made of **Blocks**, **Elements** and **Modifiers**. And then it clicked! We realized we probably wouldn't be the only ones visiting these lands. We needed to pave the way and support other explorers who wished to browse through its many misteries. However, an important decision had to be made first; should we start from a simple trail and improve from there, or should we aim for the most beautiful path first and then add fixes as problems come along? It wasn't quite clear at the moment, but once we came to understand the true meaning of [Graceful Degradation], clearing out the many misconceptions around it, we immediately knew this was the best choice for us, and so we started working on it. We soon realized, though, that our methedologies for detecting those problems were getting old and rusty, leading ourselves to a potential maintenance nightmare. We needed to [modernizer] them, and that is exactly what we did, [`@mixin`g] it with some JavaScript when necesary. Of course, nothing is perfect in this world, so we arrived at the conclusion that combining two or more of these detections was the best course of action for us because it allowed us to work sooner on the actual fixes, such as [how to make a tiling without SVG `<pattern>`s or `background-repeat`].

And here we are, at the very end of our incredible journey with a fully funcitonal Responsive Medieval Board, just as we planned it. You might be thinking, what else could be possible do? Is there anything that have slipped our minds? Anything we could improve? Naturally, that depends on who you ask. But to me, there is, and it is actually quite important.

Usually, when working with other languages, it is common practice to make sure your custom functions work the way you expect them to work by applying some kind of **error handling**. This generally implies defining what kind of arguments are accepted and throwing an error message when an invalid one was passed for easier debugging. It is something easy to overlook when working with Sass, but it is really worth doing, even if it is code only we are going to see. But there is no better way to explain something than through an example, so let us start off with `tiling-images()`, the first function we created for remaking the tiling on the board.

To recap, the original function looked like this.

```scss
// Returns the same background image multiple times.
@function tiling-images($path, $amount: 1) {
  $result: url($path);

  @for $i from 1 to $amount {
    $result: $result, url($path);
  }

  @return $result;
}
```

It has two arguments: `$path` and `$amount`. The first one is required, meaning, Sass will throw an error if nothing is entered. On the other hand, the second one defaults to `1` if nothing is passed. So far so good. Now let us think for a moment what would happen if we entered the wrong path, or something not even remote to one. Well, the image would simple not load, so we would immediately notice something went wrong. Not only that but the CSS [`background-image`] property accepts many values besides a simple URL we would have to test for. So in this case, it is better not to validate the value. The `$amount` parameter however, is a whole different story. It should ony accept an **integer** representing the number of `background-image`s we want the function to `@return`. We can easily test for that by passing it as the argument of Sass' [`unitless()`] function in an `@if` clause wrapping our function's `@for` loop. In the `@else` clause, Sass will throw an `@error` message through our `ìnvalid-arg()` function.  

```scss
@function tiling-images($path, $amount) {
  $result: url($path);

  @if (unitless($amount)) {
    @for $i from 1 to $amount {
      $result: $result, url($path);
    }
  } @else {
    @error invalid-arg("tiling-images", "$amount", $amount, "integer");
  }

  @return $result;
}
```

The `tiling-positions()` function is a little more complicated, though. It has a lot more variables, for one thing. It uses `_tiling-line-position()` as its engine but since it takes almost the same arguments and it is a **private** function, these don't need to be validated. We will perform the check inside `tiling-positions()`, displaying an `@error` with `invalid-arg()`, as mentioned earlier. This function takes `$function-name`, `$arg-name`, `$arg` and `$expected-args` as its arguments, so for each argument we should know its `$name`, `$value`, the `$check` to be performed, and the `$expected` arguments. This sounds like the perfect scenario for using Sass [maps] to **loop** through, don't you think?

```scss
$args: ("tiling-x": ("name": "$tiling-x",
                     "value": $tiling-x,
                     "check": // Do some check,
                     "expected": // Valid arguments),
        "tiling-y": ("name": "tiling-y",
                     "value": $tiling-y,
                     "check": // Do some check,
                     "expected": // Valid arguments),
        "tile-dx": ("name": "$tile-dx",
                    "value": $tile-dx,
                    "check": // Do some check,
                    "expected": // Valid arguments),
        "tile-dy": ("name": "$tile-dy",
                    "value": $tile-dy,
                    "check": // Do some check,
                    "expected": // Valid arguments),
        "a": ("name": "$a",
              "value": $a,
              "check": // Do some check,
              "expected": $valid-a),
        "b": ("name": "$b",
              "value": $b,
              "check": // Do some check,
              "expected": // Valid arguments),
        "tiles-per-line": ("name": "$tiles-per-line",
                           "value": $tiles-per-line,
                           "check": // Do some check,
                           "expected": // Valid arguments),
        "lines": ("name": "$lines",
                  "value": $lines,
                  "check": // Do some check,
                  "expected": // Valid arguments),
        );
```





[`background-image`]: https://developer.mozilla.org/en-US/docs/Web/CSS/background-image
[`unitless()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#unitless-instance_method
[maps]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#maps
