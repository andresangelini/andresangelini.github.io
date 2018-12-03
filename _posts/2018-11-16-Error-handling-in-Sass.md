We have come a long way since we started our coding adventure into making a [Responsive Medieval Board With SVG Stacks]. We first stepped into SVG territory where we met its inhabitants in a new light that few people have rarely seen before. We found mysterious `<symbol>`s in its long forgotten ruins which we soon discovered were meant to help us `<def>`ine our plans so that you **Don't Repeat Yourslef**. They aided us to [improve our organization], and along with proper [Namespacing], lead us to journey through [SVG in style]. We also discovered the true meaning of its hidden `<pattern>`s, allowing us to [clip] through parts of the SVG and thus create more [complex shapes], which from then on, we could easily recognize thanks to using proper [fragment identifiers]. But just when we thought we would end up drowning in the muddy waters of our own code... [Sass!], a brave new Hero came along to our rescue. An he wasn't alone either. A friendly guy called [BEM] also saw us in a pinch and junped in to help. Under the warm of a wood fire and the soft melody of his old guitar, he shared many stories and told us about how the world was made of **Blocks**, **Elements** and **Modifiers**. And then it clicked! We realized we probably wouldn't be the only ones visiting these lands. We needed to pave the way and support other explorers who wished to browse through its many misteries. However, an important decision had to be made first; should we start from a simple trail and improve from there, or should we aim for the most beautiful path first and then add fixes as problems come along? It wasn't quite clear at the moment, but once we came to understand the true meaning of [Graceful Degradation], clearing out the many misconceptions around it, we immediately knew this was the best choice for us, and so we started working on it. We soon realized, though, that our methedologies for detecting those problems were getting old and rusty, leading ourselves to a potential maintenance nightmare. We needed to [modernizer] them, and that is exactly what we did, [`@mixin`g] it with some JavaScript when necesary. Of course, nothing is perfect in this world, so we arrived at the conclusion that combining two or more of these detections was the best course of action for us because it allowed us to work sooner on the actual fixes, such as [how to make a tiling without SVG `<pattern>`s or `background-repeat`].

And here we are, at the very end of our incredible journey with a fully funcitonal Responsive Medieval Board, just as we planned it. You might be thinking, what else could be possible do? Is there anything that have slipped our minds? Anything we could improve? Naturally, that depends on who you ask. But to me, there is, and it is actually quite important.

Usually, when working with other languages, it is common practice to make sure your custom functions work the way you expect them to by applying some kind of **error handling**. This generally involves defining what kind of arguments are accepted and throwing an error message when an invalid one was passed to facilitate debugging our code. It is something easy to overlook when working with Sass, but it is really worth doing, even if it is code only we are ever going to see. But there is no better way to explain something than through an example, so let us start off with our freshly baked `tiling-positions()` function.

```scss
@function tiling-positions($tiling-x: 0px,
                           $tiling-y: 0px,
                           $tile-dx: 0px,
                           $tile-dy: 0px,
                           $line-dx: 0px,
                           $line-dy: 0px,
                           $a: 1,
                           $b: 0,
                           $tiles-per-line: 1,
                           $lines: 1) {
  $result: null;

  @for $i from 0 to $lines {
    $result: $result, _tiling-line-position($line-x: calc((#{$line-dx} * #{$i}) + #{$tiling-x}),
                                            $line-y: calc((#{$line-dy} * #{$i}) + #{$tiling-y}),
                                            $tile-dx: $tile-dx,
                                            $tile-dy: $tile-dy,
                                            $a: $a,
                                            $b: $b,
                                            $tiles: $tiles-per-line);
  }

  @return $result;
}
```

It has a lot of arguments and all of them will be used as `_tiling-line-position()`'s arguments except `$lines`, which is the total number of times to loop through. You might think we could perform the validation inside this function instead, but this wouldn't be a good a idea. That is because the `@error` message would point to a **private** function that the user would have trouble finding out where it is from. Besides, since it is **private**, it shouldn't be used anywhere else, so there is no need for it to validate its arguments.

Before moving on to validating the arguments, we need to think how to streamline this process as much as possible. After all, the validation will only perform a certain **check** for each argument and throw an `@error` along with the **function name**, **argument name**, **argument value** and a list of **expected arguments** if it is invalid or `true` if it is ok. In other words, each argument needs to have the following information:

- **name**: the name of the argument.
- **value**: the value of the argument.
- **check**: the check to be performed on the argument.
- **expected**: the expected arguments.

This makes the perfect oportunity to use Sass [maps] to create a two-dimensional map or "map of maps". The "parent" map will have each **argument** as a **key** and a **map** containing the correspondant details of each **argument** as its **value**.

```scss
$args: ("arg": ("name": "$arg",
                "value": $arg,
                "check": boolean,
                "expected": list)
        )
```

With this we can now create a function called `check-args()` which will loop through this two-dimensional map and performs a check on each of them, throwing an `@error` if any of them is invalid or `@return`ing `@true` otherwise.

```scss
@function check-args($function-name, $args) {
  @if (type-of($function-name) != "string") {
    @error invalid-arg("check-args", "$function-name", $function-name,
                       "string");
  } @else if (type-of($args) != "map") {
    @error invalid-arg("check-args", "$args", $args, "map");
  }

  @each $key, $val in $args {
    @if (type-of($val) != "map") {
      @error invalid-arg("check-args", "$args value", $val, "map");
    } @else if (map-get($val, "check") == false) {
      @error invalid-arg($function-name,
                         "#{map-get($val, 'name')}",
                         map-get($val, "value"),
                         map-get($val, "expected"));
    }
  }

  @return true;
};
```

The functions is comprised of two part main parts; on the first part we make sure the arguments of `check-args()` itself are valid, that is `$function-name` and `$args`, using the Sass [`type-of()`] function. The former should be a `string` and the later a `map`. If everything is ok, then we check that the **key** and **value** are also ok with [`map-get()`]. If all of that is fine, we perform the check on each argument.

All this work allow us to easily do all of our tests in a simple `@if` clause.

```scss
@if (check-args("tiling-positions", $args)) {
  // Loop through each line of background-positions.
}
```

With this, we are effectively making our code easier to understand by discriminating between **logic** and raw **data**. The former is the `check-args()` function executed in that single `@if` clause we just wrote there while the later is the `$args` map containing all the information pertaining the arguments. Having not done this, we would otherwise be writing an endless amount of conditional statements which we would soon find very difficult to understand, let alone debug it when something goes wrong.

As for the check themselves, let's start from the first two arguments; `$tiling-x` and `$tiling-y`. According to Mozilla's documentation on [`background-position`], there is a total of **six** possible values:

- **keyword**: `left`, `center`, `right`, `top`, and `bottom`.
- **percentage**: any percentage value such as `3%`, `-20%`, etc.
- **length**: any rational number that has units such as `30px`, `-23em` and `0.2cm` or `0`.
- **edge offset**: a value pair comprised of a **keyword** and a **length** such as `left 45px`, `right -0.11em` or `bottom 100cm`.
- **global**: `inherit`, `initial` or `unset`.
- **calc()**: although there is no mention of what `calc()` is considered to be, we need to take it into account as a special case.

The best way to approach this is to create a function whose only purpose is to return a **boolean** for each one of these possibilities so that we can later use them in conjunction with logical operators to validate the values for positioning the tiling. However, some of the tests are so simply that you might be tempted to think they are not worth it at all. Nonetheless, it is cerainly a lot easier to read something like `@if (is-string($value)) {...}` than `@if (type-of($value) == "string") {...}`. Not only is it much shorter, but it also allows us to encapsulate and isolate whatever we are testing in a very simple piece of code that can be reused again and again.

Let's see how this works by creating a test for the **keyword** value using Sass [`index()`]. This function returns the position of a value within a list or `null` if it doesn't find it. For this reason, we will wrap it up with another function wich will actually return `true` or `false`.

```scss
@function is-in-list($value, $list) {
  @if (index($list, $value)) {
    @return true;
  } @else {
    @return false;
  }
}
```

And this function will be used to test if the argument is among the valid ones.

```scss
@function is-position-keyword($value) {
  @return is-in-list($value, left right top bottom center);
}
```

As well as for checking if it is a global value.

```scss
@function is-global($value) {
  @return is-in-list($value, initial inherit unset);
}
```

You can see here that these functions are really simple. They **do one thing and one thing only**; return `true` or `false` and nothing more.

But what happens if, let's say, `$tiling-x` is `left 150px` and `$tile-dx` is `20px`? You can't do any operations with **keywords**. We need to do some kind of converstion first in order to do calculations with them. We will use Sass [maps] once more to assign a **percentage** value for each **keyword**.

```scss
@function keyword-to-percentage($keyword) {
  $keywords: (
    left: 0,
    right: 100%,
    top: 0,
    bottom: 100%,
    center: 50%
  );

  @if (map-has-key($keywords, $keyword)) {
    @return map-get($keywords, $keyword);
  } @else {
    @return $keyword;
  }
}
```

We can expand on this to make another one for converting **edge offset** and **keyword** values into a **length** or **calc()**, or do nothing otherwise.

```scss
@function quantify-position($position) {
  $edge-offsets: null;

  @if (is-position-keyword($position)) {
    @return keyword-to-percentage($position);
  } @else if (is-edge-offset($position)) {
    $edge-offsets: (
      left: nth($position, 2),
      right: calc(100% - #{nth($position, 2)}),
      top: nth($position, 2),
      bottom: calc(100% - #{nth($position, 2)}),
    );

    @return map-get($edge-offsets, nth($position, 1));
  }@else {
    @return $position;
  }
}
```

Now let's create another function for validating **percentages**, which must be any rational number with a `%` appended to it. We can easily test if a value is a rational number with the help of Sass [`type-of()`], which we will wrap in a new function called `is-number()`.

```scss
@function is-number($value) {
  @return type-of($value) == "number";
}
```

Now we can test if the value has a `%` in it with Sass [`unit()`].

```scss
@function is-percentage($value) {
  @if (is-number($value)) {
    @return unit($value) == "%";
  } @else {
    @return false;
  }
}
```

It is important to remember that these boolean funcitons **should only return `true` or `false`**, even if the argument is the wrong type. In other words, they should never throw any `@error`. Our `check-args()` function will be the one in charge of doing of that.

A **length** is any rational number that has units, including `0`, but is not a **percentage**.

```scss
@function is-length($value) {
  @if (is-number($value)) {
    @return unit($value) != "%" or $value == 0;
  } @else {
    @return false;
  }
}
```  

Thanks to our previous efforts, validating an **edge offset** becomes really easy and the code here should be self-explanatory.

```scss
@function is-edge-offset($value) {
  @return length($value) == 2 and
          is-position-keyword(nth($value, 1)) and
          nth($value, 1) != "center" and
          is-length(nth($value, 2));
}
```

We already have a test for `calc()` called `is-calc()`, which we created for our `strip-calc()` function in the post [Getting Sassy with the board]. But let us include it, just in case you forgot.

```scss
@function is-calc($value) {
  @return str_slice("#{$value}", 0, 4) == "calc";
}
```

With all these tests ready, we can move on to creating an `is-position()` function to test if `$tiling-x` and `$tiling-y` are valid positions.

```scss
@function is-position($position) {
  @return is-position-keyword($position) or
          is-percentage($position) or
          is-length($position) or
          is-edge-offset($position) or
          is-global($position) or
          is-calc($position);
}
```

Validating `$tile-dx`, `$tile-dy`, `$line-dx` and `$line-dy`, on the other hand, is much more simpler.

```scss
@function is-delta($delta) {
  @return is-percentage($delta) or is-length($delta);
}
```

As for `$a` and `$b`, which are used to modifiy `$i`, they must be **rational numbers** without **units**. So basically, the only two things we have to ask are if the value **is a number** and if **it has units**. We already have `is-number()`, so let's work on `has-units()`.

```scss
@function has-units($value) {
  @if (is-number($value)) {
    @return not unitless($value);
  } @else {
    @return false;
  }
}
```

One thing to note here is that Sass [`unitless()`] throws an `@error` if its argument is the wrong type, so we need to make sure the one we are passing to our `has-units()` function **is actually a number**. Then, the validation for both `$a` and `$b` will be the result of `is-number($value) and not has-units($value)` because a **string** doesn't have units after all, since it's not even a number, so `not has-units($value)` alone would return `true`, which is wrong.

Moving on, the only two arguments remaining are `$tiles-per-line` and `$lines`. Both of them need an **integer**...

```scss
@function is-integer($value) {
  @if (is-number($value) and not has-units($value)) {
    @return $value % 1 == 0;
  } @else {
    @return false;
  }
}
```

And bigger than `0`.

```scss
@function is-positive($value) {
  @if (is-number($value)) {
    @return $value > 0;
  } @else {
    @return false;
  }
}
```

I hope you see how easy it becomes to read and understand these type of functions when you write them as `true` or `false` questions. They undeniably go a long way toward helping us write cleaner code. As a testament of that, testing the rest of the arguments now results trivial.

First, we define what the valid arguments are in separated lists at the begining of `tiling-positions()` and then we create the **two dimensional map** of arguments to be passed through `check-arg()`.

```scss
@function tiling-positions($tiling-x: 0px,
                           $tiling-y: 0px,
                           $tile-dx: 0px,
                           $tile-dy: 0px,
                           $line-dx: 0px,
                           $line-dy: 0px,
                           $a: 1,
                           $b: 0,
                           $tiles-per-line: 1,
                           $lines: 1) {
  $result: null;
  $valid-positions: "keyword", "<percentage>", "<length>", "edge offset", "global", "calc()";
  $valid-deltas: "<percentage>", "<length>";
  $valid-amounts: "integer > 0";

  $args: ("tiling-x": ("name": "$tiling-x",
                "value": $tiling-x,
                "check": is-position($tiling-x),
                "expected": $valid-positions),
          "tiling-y": ("name": "$tiling-y",
                "value": $tiling-y,
                "check": is-position($tiling-y),
                "expected": $valid-positions),
          "tile-dx": ("name": "$tile-dx",
                      "value": $tile-dx,
                      "check": is-delta($tile-dx),
                      "expected": $valid-deltas),
          "tile-dy": ("name": "$tile-dy",
                      "value": $tile-dy,
                      "check": is-delta($tile-dy),
                      "expected": $valid-deltas),
          "line-dx": ("name": "$line-dx",
                      "value": $line-dx,
                      "check": is-delta($line-dx),
                      "expected": $valid-deltas),
          "line-dy": ("name": "$line-dy",
                      "value": $line-dy,
                      "check": is-delta($line-dy),
                      "expected": $valid-deltas),
          "a": ("name": "$a",
                      "value": $a,
                      "check": is-number($a) and not has-units($a),
                      "expected": "<number>"),
          "b": ("name": "$b",
                      "value": $b,
                      "check": is-number($b) and not has-units($b),
                      "expected": "<number>"),
          "tiles-per-line": ("name": "$tiles-per-line",
                             "value": $tiles-per-line,
                             "check": is-integer($tiles-per-line) and
                                      is-positive($tiles-per-line),
                             "expected": $valid-amounts),
          "lines": ("name": "$lines",
                            "value": $lines,
                            "check": is-integer($lines) and
                                     is-positive($lines),
                            "expected": $valid-amounts),
          );

  @if (check-args("tiling-positions", $args)) {
    @for $i from 0 to $lines {
      $result: $result, _tiling-line-position($line-x: calc((#{$line-dx} * #{$i}) + #{strip-calc(quantify-position($tiling-x))}),
                                              $line-y: calc((#{$line-dy} * #{$i}) + #{strip-calc(quantify-position($tiling-y))}),
                                              $tile-dx: $tile-dx,
                                              $tile-dy: $tile-dy,
                                              $a: $a,
                                              $b: $b,
                                              $tiles: $tiles-per-line);
      }
  }

  @return $result;
}
```

The resulting function is certainly bulkier than before, but this is due to the `args` map which contains all the information needed to validate each argument. Not only does this makes modifying the conditions easier but it also saves us from creating never ending chains of obscure conditional statements.

The next function to have **error handling** added to will be `tiling-images()`.

```scss
@function tiling-images($path, $amount) {
  $result: url($path);

  @for $i from 1 to $amount {
    $result: $result, url($path);
  }

  @return $result;
}
```

Luckly for us, it only has two arguments; `$path` and `$amount`. In the first one, we will leave up to the user to make sure the actual location is right. What we will be validating, however, is that this is done using the proper methods. The [`background-image`] accepts only two values: the **keyword** `none` or the [`<image>`] element. The later itself can be either an [`element()`] function or a [`<gradient>`] element, which also has four possible options: `linear-gradient()`, `radial-gradient()`, `repeating-linear-gradient()`, and lastly, `repeating-radial-gradient()`.

Much in the same way we did for our previous function, we will create simple boolean functions for each one of these elements. The first thing we want to test for is if a given `$value` is a `string`, since that is what's used to determine the `$source` of a `background-image`.

```scss
@function is-string($value) {
  @return type-of($value) == "string";
}
```

The first thing a `background-image` accepts and the easiest to create a check for is the **keyword** `none`.

```scss
@function is-image-keyword($value) {
  @if (is-string($value)) {
    @return $value == "none";
  } @else {
    @return false;
  }
}
```

As we have seen, the `<image>` element accept many CSS functions, so the next thing we need to create is a check for determining whether a value is a function or not.

```scss
@function is-function($function, $value) {
  @if (is-string($function) and is-string($value)) {
    @return str-slice($value, 0, str-index($value, "(")) ==
            str-slice($function, 0, str-index($function, "(")) and
            str-slice($value, -1) == ")";
  } @else {
    @return false;
  }
}
```

After making sure both of this function's arguments are actually `strings`, we compare the first part up to the `(` of the `$value` to the same part of the target `$function` while also making sure the last character of `$value` is `)`.

With this relatively simple function we can now create two more checks very easily.

```scss
@function is-url($value) {
  @return is-function("url()", $value);
}

@function is-element($value) {
  @return is-function("element()", $value);
}
```

We will use a `list` for the `<gradient>` check because it cleaner and easier to understand than queuing a bunch of booleans.

```scss
@function is-gradient($value) {
  $functions: "linear-gradient()", "radial-gradient()",
              "repeating-linear-gradient()", "repeating-radial-gradient()";

  @each $function in $functions {
    @if (is-function($function, $value)) {
      @return true;
    }
  }

  @return false;
}
```

Note that we don't even bother to check if the `$value` is a string. The `is-function()` function already does it for us.

With these we already have all what's needed to create a check for `<image>`. Again, each of the inner functions already check if the `$value` is a `string`.

```scss
// Checks if a value is an <image> and returns a boolean.
@function is-image($value) {
  @return is-url($value) or
          is-gradient($value) or
          is-element($value);
}
```

All that's left is adding the **error handling** to the `tiling-images()` function just like we did with `tiling-position()`.

```scss
@function tiling-images($source, $amount: 1) {
  $result: unquote("#{$source}");
  $valid-sources: "none", "url()", "linear-gradient()", "element()";
  $valid-amounts: "integer > 0";

  $args: ("source": ("name": "$source",
                   "value": $source,
                   "check": is-image($source) or is-image-keyword($source),
                   "expected": $valid-sources),
          "amount": ("name": "$amount",
                     "value": $amount,
                     "check": is-integer($amount) and is-positive($amount),
                     "expected": $valid-amounts)
         );

  @if (check-args("tiling-images", $args)) {
    @for $i from 1 to $amount {
      $result: $result, unquote($source);
    }
  }

  @return $result;
}
```

You can see that the structure of the function is the same. What changes is their **data**, that is, the **arguments** and naturally, the contents of the `map`. Pay attention to the `$result` value we capture at the begining, though. Contrary to `tiling-positions()`, this function accepts both **quoted** and **unquoted** `string`s, which means we need to make sure the final CSS output is **unquoted**. However, the Sass `unquote()` function raises an `@error` if its value is not a `string`. To solve this, we simply convert whatever the value may be to a `string` so that it can be **unquoted** by `unquote()` before reaching any of our checks. We also need to do the same when it's added to the previous `$result` in the `@for` loop, otherwise only the first `background-image` would be **unquoted**.

The last function we need to work on is `tiling-sizes()`. The only difference wordthy of attention is that the `height` value shouldn't be passed if either of the **keyword**, `cover` and `contain`, is used.

```scss
@function tiling-sizes($width: inherit, $height: inherit, $amount: 1) {
  $result: null;
  $valid-sizes: "keyword", "global", "auto", "<length>", "<percentages>",
                "calc()";
  $valid-amounts: "integer > 0";

  $function-name: "tiling-sizes";

  $args: ("width": ("name": "$width",
                     "value": $width,
                     "check": is-size($width),
                     "expected": $valid-sizes),
          "height": ("name": "$height",
                     "value": $height,
                    "check": is-size($height),
                    "expected": $valid-sizes),
          "amount": ("name": "$amount",
                    "value": $amount,
                    "check": is-integer($amount) and is-positive($amount),
                    "expected": $valid-amounts),
          );

  @if (check-args("tiling-sizes", $args)) {
    @for $i from 1 through $amount {
      @if (is-size-keyword($width)) {
        @if ($height != inherit) {
          @error "tiling-sizes: When a keyword is used, $height is not " +
                 "needed. Was #{$height}, expected null.";
        } @else {
          $result: $result, unquote("#{$width}");
        }
      } @else {
        $result: $result, unquote("#{$width}") unquote("#{$height}");
      }
    }
  }

  @return $result;
}
```

Again, we make sure the CSS output is unquoted by interpolating `$width` and `$height` and passing them through Sass `unquote()`.





[`background-image`]: https://developer.mozilla.org/en-US/docs/Web/CSS/background-image
[`unitless()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#unitless-instance_method
[maps]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#maps
[`type-of()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#type_of-instance_method
[`map-get()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#map_get-instance_method
[`background-position`]: https://developer.mozilla.org/en-US/docs/Web/CSS/background-position
[`<image>`]: https://developer.mozilla.org/en-US/docs/Web/CSS/image
[`<gradient>`]: https://developer.mozilla.org/en-US/docs/Web/CSS/gradient
[`element()`]: https://developer.mozilla.org/en-US/docs/Web/CSS/element
[`index()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#index-instance_method
[`unitless()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#unitless-instance_method
[`nth()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#nth-instance_method
[`unit()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#unit-instance_method
