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

The `tiling-positions()` function is a little more complicated, though. It has a lot more variables, for one thing.

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

All of them will be used as `_tiling-line-position()`'s arguments except `$lines`, which is the total number of times to loop through. You might think we could perform the validation inside this function instead, but this wouldn't be a good a idea, really. And that is because the `@error` message would point to a **private** function that the user would have trouble finding out where it is from. Besides, since it is **private**, it shouldn't be used anywhere else, so there is no need for it to validate its arguments.

Before moving on to validating the arguments, we need to think how to streamline this process as much as possible. After all, the validation will only perform a certain **check** for each argument and throw an `@error` along with the **function name**, **argument name**, **argument value** and a list of **expected arguments** if it is invalid or `true` if it is ok. In other words, each argument needs to have the following information:

- **name**: the name of the argument
- **value**: the value of the arguments
- **check**: the check to be performed on the argument
- **expected**: the expected arguments

This makes the perfect oportunity to use Sass [maps] to create a two-dimensional map or "map of maps". The "parent" map will have each **argument** as a **key** and a **map** containing the correspondant details of each **argument** as its **value**.

```scss
$args: ("arg": ("name": "arg",
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
    } @else if (map-get($val, "check")) {
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

All this work allow us to easily do all our tests in a simple `@if` clause.

```scss
@if (check-args("tiling-positions", $args)) {
  // Loop through each line of background-positions.
}
```

As for the check themselves, let's start from the first two arguments; `$tiling-x` and `$tiling-y`. According to Mozilla's documentation on [`background-position`], there is a total of **six** possible values:

- **keyword**: `left`, `center` or `right` for `x` or `top`, `center` or `bottom` for `y`.
- **percentage**: any percentage value such as `3%`, `20%`, etc.
- **length**: any rational number such as `30px`, `23em`, `2cm`, etc. `0` is the only value that might or might not use units.
- **edge offset**: a value pair comprised of a **keyword** and a **length** such as `left 45px`, `center 11em` or `bottom 100cm`.
- **global**: `inherit`, `initial` or `unset`.
- **calc()**: although there is no mention of what `calc()` is considered to be, we need to take it into account as a special case.

The best way to approach this is to create a function whose only purpose is to return a **boolean** for each one of these possibilities so that we can later use them in conjunction with logical operators to validate the values for positioning the tiling. However, some of the test are so simply that you might be tempted to think they are not worth it. Why should we bother, right? Well, because we should always strive to maintain our code as clean and as readible possible, and reading a set of simple `true` or `false` questions is definitely easy enough. You will see this in action soon but, for now, let's create the first one.

The **keyword** value can be tested with Sass [`index()`]. This function returns the position of a value within a list or `null` if it doesn't find it. For this reason, we will wrap it up with another function wich will actually return `true` or `false`.

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
@function is-position-keyword($axis, $value) {
  @if ($axis == "x") {
    @return is-in-list($value, left center right);
  } @else if ($axis == "y") {
    @return is-in-list($value, top center bottom);
  } @else {
    @return false;
  }
}
```

This might seem like doing extra work but it actually makes it a lot easier to read and understand. This is why a **function should always try to do one thing, and one thing only**.

Now let's create another function for validating **percentages**, which must be any rational number with a `%` appended to it. We can easily test if a value is a rational number with the help of Sass [`unitless()`]. However, Sass considers `%`, as well as any other **string** as a **unit**, so if you pass any **percentage** value, [`unitless()`] will actually return `false`. This is why we need to invert the result with `not unitless()`. Then, we can exclude non-percentage value using Sass [`unit()`].

```scss
@function is-percentage($value) {
  @return not unitless($value) and unit($value) == "%";
}
```

A **length** must be any rational number, including `0`, and must have units, but it is not a percentage.

```scss
@function is-length($value) {
  @return not unitless($value) and unit($value) != "%" or $value == 0;
}
```  

And now we can test whether a value is an **edge offset**.

```scss
@function is-edge-offset($axis, $value) {
  @return length($value) == 2 and is-position-keyword($axis, nth($value, 1)) and
          is-length(nth($value, 2));
}
```

In the same manner, we create another check for the global values.

```scss
@function is-global($value) {
  @return is-in-list($value, initial inherit unset);
}
```

We already have a test for `calc()` called `is-calc()`, which we created for our `strip-calc()` function in the post [Getting Sassy with the board].

With all these tests ready, we can move on to creating an `is-position()` function to test if `$tiling-x` and `$tiling-y` are valid positions with the aid of Sass [`nth()`] function.

```scss
@function is-position($axis, $position) {
  @return is-position-keyword($axis, $position) or
          is-percentage($position) or
          is-length($position) or
          is-edge-offset($axis, $position) or
          is-global($position) or
          is-calc($position);
}
```

And then do the same for `$tile-dx`, `$tile-dy`, `$line-dx` and `$line-dy`.

```scss
@function is-delta($delta) {
  @return is-percentage($delta) or is-length($delta);
}
```

I hope you see how easy it becomes to read and understand these type of functions when you write them as `true` or `false` questions. They undeniably go a long way toward helping us write cleaner code. As a testament of that, testing the rest of the arguments now results trivial.

First, we define lists of valid values at the begining of `tiling-positions()` and then we create the **two dimensional map** of arguments to be passed through `check-arg()`.

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
  $valid-amounts: "non 0 integer";

  $args: ("x": ("name": "$tiling-x",
                "value": $tiling-x,
                "check": is-position("x", $tiling-x),
                "expected": $valid-positions),
          "y": ("name": "$tiling-y",
                "value": $tiling-y,
                "check": is-position("y", $tiling-y),
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
                      "check": unitless($a),
                      "expected": "<number>"),
          "b": ("name": "$b",
                      "value": $b,
                      "check": unitless($b),
                      "expected": "<number>"),
          "tiles-per-line": ("name": "$tiles-per-line",
                             "value": $tiles-per-line,
                             "check": unitless($tiles-per-line),
                             "expected": $valid-amounts),
          "lines": ("name": "$lines",
                            "value": $lines,
                            "check": unitless($lines),
                            "expected": $valid-amounts),
          );

  // Check if there is any invalid argument and if so, throw an error.
  // Otherwise prepare oue result.
  @if (check-args("background-positions", $args)) {
    @for $i from 0 to $tiles {
      $result: $result, calc((#{$tile-dx} * (#{$a} * #{$i} + #{$b})) + #{$line-x}) +
      " " +
      calc((#{$tile-dy} * (#{$a} * #{$i} + #{$b})) + #{$line-y});
    }
  }

  @return $result;
}
```

The resulting function is certainly bulkier than before, but what is making that bulk is the necessary details for each argument stored in `$args` and not a bunch of never ending conditional statements chained together made up of even more obscure deep nested conditionals.

[`background-image`]: https://developer.mozilla.org/en-US/docs/Web/CSS/background-image
[`unitless()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#unitless-instance_method
[maps]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#maps
[`type-of()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#type_of-instance_method
[`map-get()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#map_get-instance_method
[`background-position`]: https://developer.mozilla.org/en-US/docs/Web/CSS/background-position
[`index()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#index-instance_method
[`unitless()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#unitless-instance_method
[`nth()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#nth-instance_method
[`unit()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#unit-instance_method
