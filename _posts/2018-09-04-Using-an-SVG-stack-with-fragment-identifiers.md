### Using an SVG stack with Fragment Identifiers.

Up until now we have been using an inline SVG for each of the pieces belonging to our [medieval board]. However, one of our main [objectives] is to use just one SVG file for all our graphic needs to avoid unnecessary HTTP requests. Fortunatly, this can be easily done as explained over at [CSS-Tricks]. We will be using this SVG as a `background-image` by separeting it into different layers. Each layer will consist of a `group` with an `id` assigned to it and will represent each of the individual pieces we have been building so far as individual SVGs. All of the layers will be hidden by default with `display: none` to only be displayed separatelly by targeting them with the `:target` selector and setting it to `display: inline` when they are used as a `background-image`.








[CSS-Tricks]: https://css-tricks.com/svg-fragment-identifiers-work/
