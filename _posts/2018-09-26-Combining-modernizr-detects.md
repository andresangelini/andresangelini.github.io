## A more complex feature detection

Moving on to the rest of the issues, you'll notice there are somehow related:

- Safari and Safari iOS don't support **SVG Fragment Identifiers** at all.
- Chrome versions 36 to 49 only support **SVG Fragment Identifiers** when used inside `<img>` elements.
- Safari 6, iOS Safari 6.1 and Chrome versions 25 to 19 suupport `calc()` with `-webkit-` prefix but older versions do not support it at all.
- Safari won't display background image if background size and position aren't explicity set.
