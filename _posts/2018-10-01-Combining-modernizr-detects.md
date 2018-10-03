## A more complex feature detection

In the previous post we succesfully created a workouround for displaying the boards when SVGs are not supported as multiple `background-image`. Moving on to the rest of the issues, you'll notice there are somehow related:

- Safari and Safari iOS don't support **SVG Fragment Identifiers** at all.
- Chrome versions 36 to 49 only support **SVG Fragment Identifiers** when used inside `<img>` elements.
- Safari 6, iOS Safari 6.1 and Chrome versions 25 to 19 suupport `calc()` with `-webkit-` prefix but older versions do not support it at all.
- Safari won't display background image if background size and position aren't explicity set.

The first two are related to support for SVG Fragment Identifiers while the other two are more or less trivial, so let's start with the former ones.

This time the solution is a little bit more involved because we have no other option than splitting up the SVG into different files, one for each layer (or "SVG fragment"). This is definetely not the greatest solution resourse wise but at the same time we don't expect it will affect that many users, so it's an acceptable consession.

By taking a look at the [download] section of Modernizr you will see once again that there is no **module** for detecting suppor for **SVG Fragment Identifier**. Nevertheless, we can apply the same method we have used earlier to target those browsers affected by this issue. Of all the modules available, JPEG 2000 has almost the opposite support of [SVG Fragment Identifiers]. We can use this module, then, to detect when it's supported instead by just using the `.jpeg2000` selector.

As for the second issue (Chrome 36 to 49 not supporting SVG Fragment Identifiers), there is not a single Modernizer **module** that fits the bill. The [Passive Event Listeners] modules detects the right Chrome versions but also other browsers too. On the other hand, the [Webp] has only support for Chrome, Opera, Android among other few. If we use both `.no-passiveeventlistener` and `.webp` together then we will get quite close to the range of browsers affected by this issue.

Both of these problems share the same solution so let's add this new rule together with the previous one for each board type (one inside `&--type-bulletin` and another one inside `&-type-sign`).

```scss
@at-root .jpeg2000 &,
         .no-passiveeventlistener.webp & {

}
```

The four issue can be solved very easily by creating a rule specifying the `background-size` and `background-position` of each `div` displaying an **SVG Fragment**.

```scss
@at-root .jpeg2000 &,
         .no-passiveeventlistener.webp & {
  div {
    background-size: 100% 100%;
    background-position: center top;
    position: absolute;
  }
}
```

The rules specefic to each `div` are the same as the ones used to display the SVG stack but with the exception that each `div` has a different URL path because they are pointing to different files. Thanks to the **funcitons** we made earlier, the only thing we need to change are those path. But before we, we will save them as variables in the `_variables.scss` partial.

```scss
// Multiple SVGs are needed as a fallback for browsers that don't support SVG Fragment Identifiers.
$path-to-top-chains                   : "https://cdn.rawgit.com/andresangelini/f3415703d9665bc6d2e0fcdefd90c252/raw/8a9d7f56730094d681762638c76db1df3ffdd538/topChains.svg";
$path-to-bottom-chains                : "https://cdn.rawgit.com/andresangelini/96fc2fe2937f63997f972f203509bb28/raw/04eb599bf86ffce922d53071c8a10013743a3436/bottomChains.svg";
$path-to-chain-link                   : "https://cdn.rawgit.com/andresangelini/5969d4f442bb18ec3b81db61ab4202fe/raw/6a51402d5877ccfec47937952937af0d39aa7ddc/chainLink.svg";
$path-to-plaque                       : "https://cdn.rawgit.com/andresangelini/71186cfd071e53e4b229ef78ca60207f/raw/c33418e6d8c07c71914cc08bbc7e442d58a7f838/plaque.svg";
$path-to-bulletin-corners             : "https://cdn.rawgit.com/andresangelini/ba8d6301a441f9477aff3a93fc93a730/raw/32c534c1f196783a4243d4edd7ec1c268a0b9eaf/bulletinCorners.svg";
$path-to-bulletin-horizontal-sides    : "https://cdn.rawgit.com/andresangelini/6a8860e1d3de990f1ce5a7d27aa26a30/raw/2412b8174843f846425aad281f4ad7dab3146fd7/bulletinHorizontalSides.svg";
$path-to-vertical-sides               : "https://cdn.rawgit.com/andresangelini/22f602b5edb5671703ead1224719c029/raw/97d9e61f03bf75e1e75a4ce7a68b3ccaf6302f5c/verticalSides.svg";
$path-to-bulletin-holes               : "https://cdn.rawgit.com/andresangelini/0fd40f69566ddb6eed16ca9154a1cdbf/raw/5be1d70ebad0936f390a57c8fa04331361a70d76/bulletinHoles.svg";
$path-to-horizontal-sides-depth       : "https://cdn.rawgit.com/andresangelini/faa4ae5d44ce839a00161699039e70e4/raw/2a48fb59bd15ff0d96dd1e996a5157619ccea832/horizontalSidesDepth.svg";
$path-to-bulletin-corners-depth       : "https://cdn.rawgit.com/andresangelini/fc167511c668f4f22a843dd5ad032c4f/raw/9ca8e54fd838d1dcbb2ac84aee0dd0e5395b993b/bulletinCornersDepth.svg";
$path-to-horizontal-sides-shadow      : "https://cdn.rawgit.com/andresangelini/81f11231029c5ca6fbf1d8dcfa723319/raw/e2dceca30c636d82df3110583a6df4f508166b88/horizontalSidesShadow.svg";
$path-to-vertical-sides-shadow        : "https://cdn.rawgit.com/andresangelini/a9fc2dec3f4ce794ed0f4e35c5d723ac/raw/06960348119880580e6fb49b344c9f69793241bb/verticalSidesShadow.svg";
$path-to-bulletin-corners-shadow      : "https://cdn.rawgit.com/andresangelini/39b00b169ef54b45aadf7a1b90df9e9f/raw/bd9afd6fbeea29b140f32a58e038fc2c20e15836/bulletinCornersShadow.svg";
$path-to-bulletin-holes-shadow        : "https://cdn.rawgit.com/andresangelini/ea2cbe38787edd3feaa6c0b072f40b6c/raw/3dc3ba415a2d88b6bf86eb3ce830cbfcf3117b73/bulletinHolesShadow.svg";
$path-to-bulletin-planks              : "https://cdn.rawgit.com/andresangelini/04efef31dfb360e613711a8c37aa96fb/raw/311911b99a4d0855a5f47f8aec5a93ed93be67e4/bulletinPlanks.svg";
$path-to-sign-corners                 : "https://cdn.rawgit.com/andresangelini/00adf3ff339050dd484ef4133d581639/raw/9204a853448f0aef66be6aaaf09c40968d44e779/signCorners.svg";
$path-to-sign-horizontal-sides        : "https://cdn.rawgit.com/andresangelini/3e76e8ae560573c809d7af9c32d79519/raw/cecd5219dd3c54a2b6bb47014e08b5e4caec306f/signHorizontalSides.svg";
$path-to-sign-holes                   : "https://cdn.rawgit.com/andresangelini/62d7a5f742d96c5a8f4f6639f523b18e/raw/fc0c5666dfdf7daab0edf6ae2e5ea0bcf7a0da8c/signHoles.svg";
$path-to-sign-corners-depth           : "https://cdn.rawgit.com/andresangelini/ebd77441753241e7fe7715e732928f4e/raw/978b2b4cf77ac864b9b908b686c17e5c3f63d5e0/signCornersDepth.svg";
$path-to-sign-corners-shadow          : "https://cdn.rawgit.com/andresangelini/1914067a3580101541fe0c0db4b22128/raw/5a78a3dd2c18db6cd796157f2fcbbddbb9b5aeba/signCornersShadow.svg";
$path-to-sign-holes-shadow            : "https://cdn.rawgit.com/andresangelini/a1dcb47272d9745ee2a57dd4b73d5678/raw/6e0f2aa4de5bbe89e3877c0976ec470e74259189/signHolesShadow.svg";
$path-to-sign-planks                  : "https://cdn.rawgit.com/andresangelini/f0c776eef5d1a6edc29caa351c572075/raw/e20d6b5ff84f3428a45cd8a9cc8a3b0f11bde7f6/signPlanks.svg";
$path-to-grooves                      : "https://cdn.rawgit.com/andresangelini/af502b3b5a38fab3f465f6ea7aaab3fb/raw/92b99800106f4818b7f750fd768c7ee0008d7b54/grooves.svg";
$path-to-shades                       : "https://cdn.rawgit.com/andresangelini/b8c88924698d0c573c8332edd9077469/raw/48c4a78df192ce11cad5b2c6974f7f0c157af0e2/shades.svg";
```  

Then we go back to the `_boards.scss` partial and add the aforementioned rules. One for `&--type-board-bulletin`:

```scss
@at-root .jpeg2000 &,
         .no-passiveeventlistener.webp & {
  div {
    background-size: 100% 100%;
    background-position: center top;
    position: absolute;
  }

  & div:nth-child(13) {@include plaque($path-to-plaque);}
  & div:nth-child(12) {@include chains($path-to-top-chains, $bulletin-holes-width, $bulletin-chains-height, $bulletin-chains-position-top);}
  & div:nth-child(11) {@include corners($path-to-bulletin-corners, $bulletin-board-height, 0);}
  & div:nth-child(10) {@include horizontal-sides($path-to-bulletin-horizontal-sides, $bulletin-horizontal-width, $bulletin-board-height, 0);}
  & div:nth-child(9) {@include vertical-sides($path-to-vertical-sides, $bulletin-vertical-height, $bulletin-board-height, $bulletin-side-position-left, 1);}
  & div:nth-child(8) {@include holes($path-to-bulletin-holes, $bulletin-holes-width, $bulletin-board-height, 0);}
  & div:nth-child(7) {@include horizontal-sides($path-to-horizontal-sides-depth, $bulletin-horizontal-width, $bulletin-board-height, 0);}
  & div:nth-child(6) {@include corners($path-to-bulletin-corners-depth, $bulletin-board-height, 0);}
  & div:nth-child(5) {@include horizontal-sides($path-to-horizontal-sides-shadow, $bulletin-horizontal-width, $bulletin-board-height, 0);}
  & div:nth-child(4) {@include vertical-sides($path-to-vertical-sides-shadow, $bulletin-vertical-height, $bulletin-board-height, $bulletin-side-position-left, 1);}
  & div:nth-child(3) {@include corners($path-to-bulletin-corners-shadow, $bulletin-board-height, 0);}
  & div:nth-child(2) {@include holes($path-to-bulletin-holes-shadow, $bulletin-holes-shadow-width, $bulletin-board-height, 0);}
  & div:nth-child(1) {@include planks($path-to-bulletin-planks, $bulletin-board-height, 0);}
}
```

And one for `&--type-sign`:

```scss
@at-root .jpeg2000 &,
         .no-passiveeventlistener.webp & {
  div {
    background-size: 100% 100%;
    background-position: center top;
    position: absolute;
  }

  & div:nth-child(13) { @include chains($path-to-top-chains, $sign-holes-width, $sign-top-chains-height, $sign-chains-position-top);}
  & div:nth-child(12) {
    @include chains($path-to-bottom-chains, $sign-holes-width, $sign-bottom-chains-height, $sign-chains-position-top);
    top: $sign-bottom-chains-div-position-top; // Overwrite function's heihgt.
  }
  & div:nth-child(11) {@include corners($path-to-sign-corners, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(10) {@include horizontal-sides($path-to-sign-horizontal-sides, $sign-horizontal-width, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(9) {@include vertical-sides($path-to-vertical-sides, $sign-vertical-height, $sign-board-height, $sign-side-position-left, 2);}
  & div:nth-child(8) {@include holes($path-to-sign-holes, $sign-holes-width, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(7) {@include horizontal-sides($path-to-horizontal-sides-depth, $sign-horizontal-width, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(6) {@include corners($path-to-sign-corners-depth, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(5) {@include horizontal-sides($path-to-horizontal-sides-shadow, $sign-horizontal-width, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(4) {@include vertical-sides($path-to-vertical-sides-shadow, $sign-vertical-height, $sign-board-height, $sign-side-position-left, 2);}
  & div:nth-child(3) {@include corners($path-to-sign-corners-shadow, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(2) {@include holes($path-to-sign-holes-shadow, $sign-holes-shadow-width, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(1) {@include planks($path-to-sign-planks, $sign-board-height, $sign-board-position-top);}
    }
}
```

None of this will work, however, until we actually add all those `div`s with JavaSript when [JPEG 2000] is **supported**.

```javascript
Modernizr.on('jpeg2000', function(result) {
  if (result) {
    // supported
    appendElementsToBoard('div');
  } else {
    // not-supported
    // Move it back to if clause after testing.

  }
});
```

And when [Webp] is supported but [Passive Event Listeners] are not.

```javascript
if (!Modernizr.webp) {
  if (Modernizr.passiveeventlisteners) {
    // supported
  } else {
    // not-supported
    appendElementsToBoard('div');
  }
}
```

## Adding vendor prefixes with Autoprefixer

The remaining issue about browsers not supporting `calc()` can be solved quite simly by using [Autoprefixer]. To use it, just follow [the steps][Autoprefixer installation] according to your development setup.

## Sass advanced functions

Another issue that popped out was that in Safari 11 SVG pattern elements with alpha are displayed with a black background instead of transparency. The closest that we can get to catching those browsers suffering from this is by combining `.jpeg2000` and `peerconnection` since [WebRTC Peer-to-peer connections][peerconnections] is not supported by Safari 11.

Once more we add the following rule to each of the board types.

```scss
@at-root .jpeg2000.peerconnection & {

  div {
    background: none; // Make sure no background is rendered first.
  }
}
```

Not being able to use **SVG patters** means that we will have to replace every **SVG Fragment** using them with a `div` with multiple `background-image`s. However, doing such a thing would result in having to write a massive amount of declarations just to have each image slightly modified. Of course, thanks to the power of [Sass], we don't have to, and in fact, it will give us the perfect opportunity to try some other cool stuff we can do with it. Be sure not to miss it in the next and final episode of our journey to building an awesome **Responsive Medieval Board with SVG Stacks**.


[download]: https://modernizr.com/download
[JPEG 2000]: https://caniuse.com/#search=jpeg
[SVG Fragment Identifiers]: https://caniuse.com/#search=svg%20fragme
[Passive Event Listeners]: https://caniuse.com/#search=passi
[Webp]: https://caniuse.com/#search=webp
[Autoprefixer]: http://autoprefixer.github.io/
[Autoprefixer installation]: https://github.com/postcss/autoprefixer#faq
[peerconnections]: https://caniuse.com/#search=peer
[Sass]: https://sass-lang.com/
