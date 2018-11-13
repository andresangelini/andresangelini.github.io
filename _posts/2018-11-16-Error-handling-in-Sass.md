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

As for the check themselves, let's start from the first two arguments; `$tiling-x` and `$tiling-y`. If we take a quick look at the Mozilla's documentation on [`background-position`], we see that **each coordinate** can either have **one or two values** separated by a space, that is, the property might have up to **four values**. What those valid arguments are depends on the amount.

- **One** value:
  - **keyword**: left, center, right, top, left
  - **global**: inherit, initial, unset
  - **length**: 0, 30px, 23em, 2cm, etc
  - **percentage**: 2%, 27%, etc
  - **calc()**: although there is no mention of what `calc()` is considered, we need to take it into account as a special case.

- **Two** values:
  - **First** value:
    - **keyword**: left, center, right, top, left
  - **Second** value:
    - **length**: 0, 30px, 23em, 2cm, etc

The **keyword** value can be tested with Sass [`index()`]. This function returns the position of a value within a list or `null` if it doesn't find it. For this reason we will wrap it up with another function wich will actually return `true` or `false`.

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

This might seem like doing extra work but it actually makes it a lot easier to read and understand. This is why a **function should always try to do one thing, and one thing only**. In the same manner, we create another check for the global values.

```scss
@function is-global($value) {
  @return is-in-list($value, initial inherit unset);
}
```

A **length** value can be any **rational number that has units**. So, first we have to check if it is actually a number, with the help of Sass [`type-of()`].

```scss
@function is-number($value) {
  @return type-of($value) == "number";
}
```

And then use [`unitless()`] to see whether the argument has units or not.

```scss
@function has-units($value) {
  @if (is-number($value)) {
    @return not unitless($value);
  } @else {
    @return false;
  }
}
```

Testing for `0` is really easy and we already have a test for `calc()` called `is-calc()`, which we created for creating our `strip-calc()` function.

With all these test ready, we can move on to creating an `is-position()` function to test if `$tiling-x` and `$tiling-y` are valid positions with the aid of Sass [`nth()`] function.

```scss
@function is-position($position) {
  @if (length($position) == 1) {
    // 1-value syntax.
    @return is-position-keyword($position) or is-global($position)
            or has-units($position) or ($position == 0) or is-calc($position);
  } @else if (length($position) == 2) {
    // 2-value syntax (edge offset).
    @return is-position-keyword(nth($position, 1)) and
            (has-units(nth($position, 2)) or
            ($position == 0));
  } @else {
    // A valid <position> might only have 1 or 2 values.
    @return false;
  }
}
```



[`background-image`]: https://developer.mozilla.org/en-US/docs/Web/CSS/background-image
[`unitless()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#unitless-instance_method
[maps]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#maps
[`type-of()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#type_of-instance_method
[`map-get()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#map_get-instance_method
[`background-position`]: https://developer.mozilla.org/en-US/docs/Web/CSS/background-position
[`index()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#index-instance_method
[`unitless()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#unitless-instance_method
[`nth()`]: http://sass-lang.com/documentation/Sass/Script/Functions.html#nth-instance_method
