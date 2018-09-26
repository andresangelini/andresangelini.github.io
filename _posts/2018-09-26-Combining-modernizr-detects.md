## A more complex feature detection

In the previous post we succesfully created a workouround for displaying the boards when SVGs are not supported as multiple `background-image`. Moving on to the rest of the issues, you'll notice there are somehow related:

- Safari and Safari iOS don't support **SVG Fragment Identifiers** at all.
- Chrome versions 36 to 49 only support **SVG Fragment Identifiers** when used inside `<img>` elements.
- Safari 6, iOS Safari 6.1 and Chrome versions 25 to 19 suupport `calc()` with `-webkit-` prefix but older versions do not support it at all.
- Safari won't display background image if background size and position aren't explicity set.

The first two are related to support for SVG Fragment Identifiers while the other two are more or less trivial, so let's start with the former ones.

This time the solution is a little bit more involved because we have no other option that splitting up the SVG into different files, one for each layer (or "SVG fragment"). This is definetely not the greatest solution resourse wise but at the same time we don't expect it will affect that many users, so it's an acceptable consession.

By taking a look at the [download] section of Modernizr you will see once again that there is no **module** for detecting suppor for **SVG Fragment Identifier**. Nevertheless, we can apply the same method we have used earlier to target those browsers affected by this issue. Of all the modules available, JPEG 2000 has almost the opposite support of [SVG Fragment Identifiers]. We can use this module then to detect when it's supported instead by just using the `.jpeg2000` selector.

As for the second issue (Chrome 36 to 49 not supporting SVG Fragment Identifiers), there is not a single Modernizer **module** that fits the bill. The [Passive Event Listeners] modules detects the right Chrome versions but also other browsers too. On the other hand, the Webp has only support for Chrome, Opera, Android and a few less used browsers.

```scss
.board {
  // Some declarations

  &--type-bulletin {
    // Some declarations

    // Safari and Safari iOS don't support SVG Fragment Identifiers.
    @at-root .jpeg2000 & {

    }
  }

  &--type-sign {
    // Some declarations

    // Safari and Safari iOS don't support SVG Fragment Identifiers.
    @at-root .jpeg2000 & {

    }
  }
}
```





[download]: https://modernizr.com/download
[JPEG 2000]: https://caniuse.com/#search=jpeg
[SVG Fragment Identifiers]: https://caniuse.com/#search=svg%20fragme
[Passive Event Listeners]: https://caniuse.com/#search=passi
