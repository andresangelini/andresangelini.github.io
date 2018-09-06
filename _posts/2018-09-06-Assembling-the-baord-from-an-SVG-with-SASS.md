### Assembling the board from an SVG with SASS

Much like when we found ourselves with a really complex SVG structure and decided to organize it using `<defs>`, namespacing and some styling, we are facing one again a bit of a problem right from the start. If we take a look back at our SVG file we can see that it's composed from several pieces which in some cases, have two variations; one for the **bulletin-board** and another for the **sign-board**. This means there will be a myriad of variables, modifications, and edge cases that our CSS will have to handle, making into a such a mess that it will be almost impossible to maintain later on.

This makes the perfect scenario for introducing [SASS] into our project. This tool is a CSS preprocessor which helps us write better style sheets by letting us use features like variables and functions that don't exits in vanilla CSS. Another of those such things is the hability to modularize our CSS into so called "partials" to make things easier to mantain. "Partials" are files where you can put different snippets of code which SASS will process and output into a single CSS file.

After [installing SASS], we create the following files in a `styles` folder.

```
/styles
  _boards.scss
  _examples.scss
  _functions.scss
  _mixins.scss
  _variables.scss
  _index.scss
```
SASS will add (or overwrite) the processed CSS output as a file called `index.processed.scss` in the same `styles` folder when you use the `sass --watch` comand (or any other name you like if you build it manually with `sass index.scss [somename].css`.

Before we even begin writing any CSS we need to store in variables the values we already know or have already stablished. These could be anything, from dimensions to colors. Since we have just created the SVG file, why don't we starts with that?

We open the `viariables.scss` and write:

```scss
// URL paths.
$path-to-board: "https://cdn.rawgit.com/andresangelini/d57f93b605eec432fdea98b969aaac72/raw/f0ee9369b781dcf01eefc37575a99d8e5dbe7b35/board.svg";
```

By the way, if you are wondering how to serve and SVG like that, just check out [this excellent article] at SitePoint.

Now the colors. We will only need one color and that will be for the board planks. All the others all already defined in the SVG itself. The reason for this will be explained later on. Anyways, even if it is only one it is still a good practice to name colors with a **descriptive name** and then with a **functional name**. The first one can be anything you like. You can be creative or if you are like in having difficulty coming up with names, go [here][name that color] and have some fun!

```scss
  // Base colors.
  $color-cape-palliser                  : #a2703f;

  // Elements colors.
  $color-planks-diffuse                 : $color-cape-palliser;
```

You can see here that I first named the color in a **descriptive** way as `cape-palliser`. Then I stored this new variables as the value of a second one named in a **functional** way as `planks-diffuse` since it is where is going to be used. Both variables were appended with `color` for a faster read.

Next are all the variables related to the elements dimensions and positions. Remember that we have two types of board (**bulletin** and **sign**), so we need to create a set of variables for each.

```scss
// Change these values to your liking. The length of the chains will adjust automatically.
$bulletin-board-height                : 95%; // 95%
$bulletin-plaque-width                : 35%;
$sign-board-height                    : 90%;

// These values are common to both types of boards. DO NOT CHANGE unless you modified the SVG graphics.
$board-depth                          : 6.5px;
$shadow-width                         : 2.16px;
$side-overlap                         : 9.4px;
$side-curve-left                      : 5px;
$side-curve-right                     : calc(100% - #{$side-curve-left});
$grooves-height                       : 527px;
$grooves-width                        : 568px;
$shade-height                         : 308px;
$chain-link-height                    : 45.94px;
```

If you look at `$side-curve-right` you can see we can make calculation with variables just like with any other programming language.



[SASS]: https://sass-lang.com/
[installing SASS]: http://sass-lang.com/install
[this excellent article]: https://www.sitepoint.com/why-hosting-your-svgs-is-hard-and-how-to-beat-it/
[name that color]: http://chir.ag/projects/name-that-color/
