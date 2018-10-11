## An otherwise mess of declarations

As briefly mentioned at the end of the last post, it would be an unimaginably tedious task, to say the least, if you had to remake an SVG pattern using multilpe `background-image`s in vanilla CSS. Just to give you an idea of how incredibly messy this would be, have a look at the line `303` of [`index.processed.css`] where the `background-image`s for the `planks` are set. If you are thinking a **mixin** could solve the problem, think again. **Mixins** can only return **declarations** and we don't want to create multiple declarations for `background-image` because we would only be overriding the previous one instead of setting multiple ones. To understand better what we need to do, let's recap how each SVG pattern is made.

The planks have actually three separate layers: a plain **background color**, the **grooves** of the wood and their **shades**. Each of them behave differently but all of them must be **clipped** so that they don't overflow through the wooden frame. Basically, as the board expands and contracts, each layer should:

- **background color**: fill all available space.
- **grooves**: repeat both horizontally and vertically.
- **shades**: have a simetric copy on each side of the board, both repeating horizontally and vertically.

Let's start with what seems easier; the **background color**.

## Clipping in CSS

Safari 11 might not support SVG patterns with alpha but it does have partial support for CSS `clip-path` if you use it along with CSS [shapes]. These last ones are a relatively new feature that allow us to draw simple figures in CSS. You have your basic shapes such as `line()`, `rectangle()` and `circle()`. To create a figure using both you only need to set a `clip-path` **property** with any of these shapes as a **value** to a `div`. For example:

```css
clip-path: circle(50px);
```

What we are actually doing, in fact, is creating a regular squared `div` and clipping it with the shape we want it to have, as shown below.

<p data-height="265" data-theme-id="0" data-slug-hash="PypmBO" data-default-tab="css,result" data-user="andresangelini" data-pen-title="Simple CSS shape" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/PypmBO/">Simple CSS shape</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

But you can also build more complex figures using `polygon()`, which is what we are going to use to reproduce our clipped SVG pattern.

<p data-height="265" data-theme-id="0" data-slug-hash="vpYbPP" data-default-tab="result" data-user="andresangelini" data-pen-title="Clipping SVG elements with complex shapes" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/vpYbPP/">Clipping SVG elements with complex shapes</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

To draw a shape using `polygon()`, we once again set it as the **value** of the `clip-path` **property** with the `x y` coordinates of each point as its arguments starting from the **top left corner** and going **clockwise**.

```css
cip-path: polygon(x-1 y-1, x-2 y-2, x-3 y-3, ...);
```
<p data-height="265" data-theme-id="dark" data-slug-hash="JmWJKE" data-default-tab="result" data-user="andresangelini" data-pen-title="CSS polygon" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/JmWJKE/">CSS polygon</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

All the usual units are allowed which is exactly what we need in order for this clip-path to be flexible. However, the boards differ not only in the size of their corners but the sign type also has holes in the bottom, so it would be wise to write a function that returns a `polygon()` depending on which board it is. To do this, we will need to use another great feature of Sass; the built-in [`if()`][if] function (not to be confused with the [`@if`][if] directive).

It basically works like this:

```
if(true, a, b) returns a
if(false, a, b) returns b
```

Which in our case would be:

```scss
if($corner-width == $sign-corner-width, $bottom-holes, null)
```

Where `$bottom-holes` represents the coordinates of the points drawing the section of the clip where the holes should be.

```scss
$bottom-holes: calc-holes-position-right($corner-width) $side-curve-right + "," +                        /* B holes BR */
               calc-holes-position-right($corner-width) calc-side-position-bottom($corner-width) + "," + /* B holes TR */
               calc-holes-position-left($corner-width) calc-side-position-bottom($corner-width) + "," +  /* B holes TL */
               calc-holes-position-left($corner-width) $side-curve-right + ",";
```

The result is a function does everything for us by simply changing its argument to the correspondant `$corner-width`.

```scss
@function planks-clip($corner-width) {
  // Sign board has bottom holes while bulletin board hasn't.
  $bottom-holes: calc-holes-position-right($corner-width) $side-curve-right + "," +                        /* B holes BR */
                 calc-holes-position-right($corner-width) calc-side-position-bottom($corner-width) + "," + /* B holes TR */
                 calc-holes-position-left($corner-width) calc-side-position-bottom($corner-width) + "," +  /* B holes TL */
                 calc-holes-position-left($corner-width) $side-curve-right + ",";                          /* B holes BL */

  @return polygon(0 calc-side-position-right($corner-width),                                               /* BL corner 3 */
                  $side-curve-left 50%,                                                                    /* L side center */
                  0 $corner-width,                                                                         /* TL corner 1 */
                  calc-corner-left($corner-width) calc-corner-left($corner-width),                         /* TL corner 2 */
                  calc-side-position-left($corner-width) 0,                                                /* TL corner 3 */
                  calc-holes-position-left($corner-width) $side-curve-left,                                /* Top holes TL */
                  calc-holes-position-left($corner-width) calc-corner-left($corner-width),                 /* Top holes BL */
                  calc-holes-position-right($corner-width) calc-corner-left($corner-width),                /* Top holes BR */
                  calc-holes-position-right($corner-width) $side-curve-left,                               /* Top holes TR */
                  calc-side-position-right($corner-width) 0,                                               /* TR corner 1 */
                  calc-corner-right($corner-width) calc-corner-left($corner-width),                        /* TR corner 2 */
                  100% calc-side-position-left($corner-width),                                             /* TR corner 3 */
                  $side-curve-right 50%,                                                                   /* R side center */
                  100% calc-side-position-right($corner-width),                                            /* BR orner 1 */
                  calc-corner-right($corner-width) calc-corner-right($corner-width),                       /* BR corner 2 */
                  calc-side-position-right($corner-width) $side-curve-right,                               /* BR corner 3 */
                  if($corner-width == $sign-corner-width, $bottom-holes, null)                             /* B holes BL */
                  calc-side-position-left($corner-width) $side-curve-right,                                /* BL corner 1 */
                  calc-corner-left($corner-width) calc-corner-right($corner-width)                         /* BL corner 2 */
                  );
}
```

Check out the Pen bellow and play around with `$corner-width` argument in the `planks-clip()` function of each board type's `clip-path` property. Try replacing `$bulleting-corner-width` with `$sign-corner-width` in for the **bulletin** type and vice versa for the **sign** type.

<p data-height="315" data-theme-id="dark" data-slug-hash="NOjXbW" data-default-tab="css,result" data-user="andresangelini" data-pen-title="CSS clip-path with polygon()" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/NOjXbW/">CSS clip-path with polygon()</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## Making patterns in plain CSS

A very basic way to simulate the grooves pattern of the planks in vanilla CSS, for example, would be to do something along these lines:

```css
& div:nth-child(1) {
  background-image: url("grooves.svg"),
                    url("grooves.svg"),
                    url("grooves.svg"),
                    //...
                    url("grooves.svg");
  background-position: 0,
                       527px 568px,
                       1054px 1136px,
                       //...
                       1527px 5680px;
  background-size: 100%;
}
```

The intention here is to create a tile pattern out of the grooves so that there are a total of 10 tiles next to each other in both `x` and `y` axis. But of course, we already know this could be a lot better by doing it the Sass way.

## Multiple `background-image`s with Sass

First off, let's make a simple function for creating multiple `background-image`s. It should have two **parameters**; the path **URL** to the image and the **amount** of copies we want. A [`@for`][for] loop will do the trick here.

```scss
@function background-images($path, $amount) {
  $result: url($path);

  @for $i from 1 to $amount {
    $result: $result, url($path);
  }

  @return $result;
}
```

However, whay would happen if the user of this function enters `0`, `-2`, `32px` or even `sfd` whether by accident or even intentionally? We need to apply some kind of **error handling** so as to let the user know what's wrong. To begin with, the `$amount` parameter should default to `1` because that's the minimum amount of images one could possible use. Just like in so many other languages, Sass let's us specify which **parameter** goes with which **value** regardless of any order by declaring it as:

```scss
@function shake($fruit: "banana", $drink: "milk") {
  // Make the ultimate milkshake!
}
```

If the user of this function doesn't change `$fruit` or `$drink` when invoking it, these parameters will use their default values of `"banana"` and `"milk"` respectively. So, in our case it will be `$amount: 1`.

Now, we can check if the user set `amount` to a value other than a number by using Sass built-in function [`unitless()`][unitless] and execute our [`@for`][for] loop on `true` or throw and [`@error`][error] on `false`. The erro message coud be something like `"background-images: $amount is invalid, was 32px, expected integer"`. But as you know by now, we should make things as generic and reusable as possible becuase we will probably handling more errors in the future and don't want be doing the same thing over and over again. Let's make a new and even more generic function that takes the **function's name**, the **argument** causing the issue, its current **value** and the **expected ones**.

```scss
@function invalid-arg($function-name, $arg-name, $arg, $expected-args) {
  @return "#{$function-name}: #{$arg-name} is invalid, was #{$arg}, " +
          "expected #{$expected-args}.";
};
```

Where `#{$variable}` is the other method Sass has to interpolate a a variable with a string by inserting it direclty innto it.

With that out of the way, now we can complete our `background-images` function.

```scss
@function background-images($path, $amount: 1) {
  $result: url($path);

  @if (unitless($amount)) {
    @for $i from 1 to $amount {
      $result: $result, url($path);
    }
  } @else {
    @error invalid-arg("background-images", "$amount", $amount, "integer");
  }

  @return $result;
}
```
From now on, whenever we need to set multiple `background-image`s using the same image, we will only have to do:

```scss
background-image: background-images(../path/to/cats.png, 20);
```
And even:

```scss
.pets {
  background-image: background-images("../path/to/cats.png", 20),
                    background-images("../path/to/dogs.png", 50),
                    background-images("../path/to/birds.png"),
                    background-images("../path/to/rabbits.png", 2);
}
```

We are not working with pets, but with **grooves** and **shades**, though. By trial and error I figured out that I needed **80** tiles of **shades** and **100** of **grooves** just to make sure the pattern won't brake in large screens.

```scss
& div:nth-child(1) {
  background-image: background-images($path-to-shades, 80),
                    background-images($path-to-grooves, 100);
}
```

It's important to remember that, contrary to what one would normally expect, the first `background-image` is the one closest to the user and the ones than come after it are placed behind.

## Multiple `background-position`s with Sass

[`index.processed.css`]: https://codepen.io/andresangelini/project/editor/Aarxxz
[shapes]: https://css-tricks.com/the-shapes-of-css/
[if]: https://sass-lang.com/documentation/file.SASS_REFERENCE.html#if
[for]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#for
[unitless]: http://sass-lang.com/documentation/Sass/Script/Functions.html#unitless-instance_method
[error]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#error
