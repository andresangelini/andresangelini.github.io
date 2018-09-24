In the last post we created a new CSS rule for those scenearios where SVGs don't scale properly when used as `background-image` and the only browser affected by this particular issue was Internet Explorer up to version 10. Sadly, Modernizr doesn't have any module for detecting support for SVG as `background-image`, though. However, by carefully checking out [Can I Use] we discovered that there Internet Explorer 10 and older do not support CSS [pointer-events] either, which does have a module, so we used that instead.

```scss
.board{
  // Some declarations.

  &--type-bulleting {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {

    }
  }

  &--type-sign {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {

    }
  }
}
```

The root of the problem seems to be that those versions of IE don't like using SVG in multiple `background-image`s. A way around that then is using multiple `div`s each with a single `background-image` pointing to its respective SVG fragment instead.

First we make sure they all have their **position** as **absolute**.

```scss
.board{
  // Some declarations.

  &--type-bulleting {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {
      & div {
        position: absolute;
      }
    }
  }

  &--type-sign {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {
      & div {
        position: absolute;
      }
    }
  }
}
```

Again, the `@at-root` Sass directive let us specify that we want all child `div`s of `.board--type-bulletin` and `.board--type-sign` to have `position: absolute` only when `.no-csspointerevents` is added to `html` element writing this rule inside our rules for `.board--type-bulletin` and `.board--type-sign`.

If at this point we were to write our rules in plain CSS we would soon find ourselver doing some ardous and repetetive task. Let's try with the plaque first to see how this would be like:

```scss
.board{
  // Some declarations.

  &--type-bulleting {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {
      & div {
        position: absolute;
      }

      /* Instead of using a single div with multiple background images, use multiple divs each with their own background image pointing to its respective layer in the SVG stack. */
      & div:nth-child(13) {
        margin-left: calc(#{$bulletin-plaque-width} / (-2));
        margin-left: calc(#{strip-calc($bulletin-plaque-width)} / (-2));    // For browsers that don't support nested calc() functions.
        left: 50%;
        width: #{$bulletin-plaque-width};
        background-image: url("#{$path-to-board}#l-plaque");
        bottom: 0;
        height: $bulletin-board-height;
      }
    }
  }

  &--type-sign {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {
      & div {
        position: absolute;
      }

      /* Instead of using a single div with multiple background images, use multiple divs each with their own background image pointing to its respective layer in the SVG stack. */
      & div:nth-child(13) {
        margin-left: calc(#{$sign-plaque-width} / (-2));
        margin-left: calc(#{strip-calc($sign-plaque-width)} / (-2));    // For browsers that don't support nested calc() functions.
        left: 50%;
        width: #{$sign-plaque-width};
        background-image: url("#{$path-to-board}#l-plaque");
        bottom: 0;
        height: $bulletin-board-height;
      }
    }
  }
}
```

Ugh! What a mess that is. I hope you can see how much repetition we can avoid here. Let's start with the most obvious ones at the bottom:

```css
background-image: url($path);
bottom: 0;
height: $bulletin-board-height;
```

Now let's throw some Sass [mixin] magic to the mix to fix this issue without repeating ourselves.





[Can I Use]: https://caniuse.com/
[pointer-events]: https://caniuse.com/#feat=pointer-events
[mixin]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#defining_a_mixin
