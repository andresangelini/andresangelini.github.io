This is the final chapter about an alchemist's personal quest for **Making a Responsive Medieval Board With SVG Stacks**. You may read the chapters in any order you want but I would otherwise suggest you to do it in the proper order to get the full context of the project.

## Chapters

1. [Making a Responsive Medieval Board With SVG Stacks][ch-1]
2. [Making an SVG Stretch][ch-2]
3. [Inverting SVG Elements][ch-3]
4. [Rotating SVG Elements][ch-4]
5. [Clipping SVG Elements][ch-5]
6. [Improving Organization With Defs][ch-6]
7. [Namespacing][ch-7]
8. [Styling an SVG][ch-8]
9. [Using SVG patterns][ch-9]
10. [Clipping SVG Elements With Complex Shapes][ch-10]
11. [Using an SVG stack With Fragment Identifiers][ch-11]
12. [Getting Sassy With The Board][ch-12]
13. [When Sass Met BEM][ch-13]
14. [Browser Support For SVG Stacks][ch-14]
15. [Sass Mixins For SVG Stacks][ch-15]
16. [Combining Modernizr Detects][ch-16]
17. [Tiling With Sass][ch-17]
18. [Error Handling in Sass][ch-18]

### Using an SVG stack with Fragment Identifiers.

Up until now we have been using an inline SVG for each of the pieces belonging to our [medieval board][ch-10]. However, one of our main [objectives] is to use just one SVG file for all our graphic needs to avoid unnecessary HTTP requests. Fortunatly, this can be easily done as explained over at [CSS-Tricks]. We will be using this SVG as a `background-image` by separeting it into different layers. Each layer will consist of a `group` with an `id` assigned to it and will represent each of the individual pieces we have been building so far as individual SVGs. All of the layers will be hidden by default with `display: none` to only be displayed separatelly by targeting them with the `:target` selector and setting it to `display: inline` when they are used as a `background-image`.

We will use the title plaque as an example to see how this works:

```xml
...
<g id='l-plaque' class='layer'>
  <use xlink:href='#bs-plaque-side-shadow' class='plaque--color-shadow' transform='translate(0 9.08)'/>
  <use xlink:href='#bs-plaque-side-shadow' class='plaque--color-shadow' x='-100%' transform='scale(-1, 1) translate(0 9.08)'/>
  <use xlink:href='#bs-plaque-side' class='plaque--color-depth' transform='translate(0 11.24)'/>
  <use xlink:href='#bs-plaque-side' class='plaque--color-depth' x='-100%' transform='scale(-1, 1) translate(0 11.24)'/>
  <use xlink:href='#clipped-plaque-center'/>
  <use xlink:href='#bs-plaque-side' class='plaque--color-diffuse' transform='translate(0 9.16)'/>
  <use xlink:href='#bs-plaque-side' class='plaque--color-diffuse' x='-100%' transform='scale(-1, 1) translate(0 9.16)'/>
  <use xlink:href='#bs-plaque-specular-left' class='plaque--color-specular' transform='translate(0 9.16)'/>
  <use xlink:href='#bs-plaque-specular-right' class='plaque--color-specular' x='100%' transform='translate(-21 9.16)'/>
  <use xlink:href='#c-plaque-nail' transform='translate(0 9.16)'/>
  <use xlink:href='#c-plaque-nail' x='100%' transform='translate(-29.83 9.16)'/>
</g>

```

Thanks to our previous efforts to keep everything organized, the only thing we have to in this first step is to create a `group` and `use` each of its pieces we definded earlier inside the `<defs>` tags. This is where we apply their colors using their respective `class`es and `transform` them to place them into their right positions.

Now we just create a `<div>` in our HTML markup and apply a `background-image` in CSS using an URL making sure to also giving it some dimensions to make it visibile.

```css
div {
  border: 1px solid black;
  position: absolute;
  width: 50%;
  height: 50%;
  background-image: url("https://cdn.rawgit.com/andresangelini/71186cfd071e53e4b229ef78ca60207f/raw/c33418e6d8c07c71914cc08bbc7e442d58a7f838/plaque.svg");
}
```
The end result should look something like our previous codepen using an inline SVG.

<p data-height="265" data-theme-id="0" data-slug-hash="vzmrjv" data-default-tab="css,result" data-user="andresangelini" data-pen-title="SVG Fragment as background-image" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/vzmrjv/">SVG Fragment as background-image</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

The rest of the layers follow the same general idea, each with their own particular adjustments, of course.

```xml
<!-- LAYERS: each layer is meant to be used as a background image. -->
        <g id='l-plaque' class='layer'>
            <use xlink:href='#bs-plaque-side-shadow' class='plaque--color-shadow' transform='translate(0 9.08)'/>
            <use xlink:href='#bs-plaque-side-shadow' class='plaque--color-shadow' x='-100%' transform='scale(-1, 1) translate(0 9.08)'/>
            <use xlink:href='#bs-plaque-side' class='plaque--color-depth' transform='translate(0 11.24)'/>
            <use xlink:href='#bs-plaque-side' class='plaque--color-depth' x='-100%' transform='scale(-1, 1) translate(0 11.24)'/>
            <use xlink:href='#clipped-plaque-center'/>
            <use xlink:href='#bs-plaque-side' class='plaque--color-diffuse' transform='translate(0 9.16)'/>
            <use xlink:href='#bs-plaque-side' class='plaque--color-diffuse' x='-100%' transform='scale(-1, 1) translate(0 9.16)'/>
            <use xlink:href='#bs-plaque-specular-left' class='plaque--color-specular' transform='translate(0 9.16)'/>
            <use xlink:href='#bs-plaque-specular-right' class='plaque--color-specular' x='100%' transform='translate(-21 9.16)'/>
            <use xlink:href='#c-plaque-nail' transform='translate(0 9.16)'/>
            <use xlink:href='#c-plaque-nail' x='100%' transform='translate(-29.83 9.16)'/>
        </g>

        <g id='l-bulletin-corners' class='layer'>
            <use xlink:href='#bs-bulletin-corner' class='frame--color-diffuse'/>
            <use xlink:href='#bs-bulletin-corner' x='-100%' transform='scale(-1, 1)' class='frame--color-diffuse'/>
            <use xlink:href='#bs-bulletin-corner' x='-100%' y='-100%' transform='scale(-1, -1) translate(0 6.5)' class='frame--color-diffuse'/>
            <use xlink:href='#bs-bulletin-corner' y='-100%' transform='scale(1, -1) translate(0 6.5)' class='frame--color-diffuse'/>
            <use xlink:href='#bs-bulletin-corner-outer-specular' class='frame--color-specular'/>
            <use xlink:href='#bs-bulletin-corner-outer-specular' x='-100%' transform='scale(-1, 1)' class='frame--color-specular'/>
            <use xlink:href='#bs-bulletin-corner-inner-specular' x='-100%' y='-100%' transform='scale(-1, -1) translate(0 6.5)' class='frame--color-specular'/>
            <use xlink:href='#bs-bulletin-corner-inner-specular' y='-100%' transform='scale(1, -1) translate(0 6.5)' class='frame--color-specular'/>
        </g>

        <g id='l-sign-corners' class='layer'>
            <use xlink:href='#bs-sign-corner' class='frame--color-diffuse'/>
            <use xlink:href='#bs-sign-corner' x='-100%' transform='scale(-1, 1)' class='frame--color-diffuse'/>
            <use xlink:href='#bs-sign-corner' x='-100%' y='-100%' transform='scale(-1, -1) translate(0 6.5)' class='frame--color-diffuse'/>
            <use xlink:href='#bs-sign-corner' y='-100%' transform='scale(1, -1) translate(0 6.5)' class='frame--color-diffuse'/>
            <use xlink:href='#bs-sign-corner-outer-specular' class='frame--color-specular'/>
            <use xlink:href='#bs-sign-corner-outer-specular' x='-100%' transform='scale(-1, 1)' class='frame--color-specular'/>
            <use xlink:href='#bs-sign-corner-inner-specular' x='-100%' y='-100%' transform='scale(-1, -1) translate(0 6.5)' class='frame--color-specular'/>
            <use xlink:href='#bs-sign-corner-inner-specular' y='-100%' transform='scale(1, -1) translate(0 6.5)' class='frame--color-specular'/>
        </g>

        <g id='l-bulletin-horizontal-sides' class='layer'>
            <use xlink:href='#c-horizontal-side' clip-path='url(#cp-horizontal-side-bottom)' class='frame--color-diffuse' width='100%' height='16.25'/>
            <use xlink:href='#c-horizontal-side' class='frame--color-diffuse' y='-100%' width='100%' height='16.25' transform='scale(1, -1) translate(0 6.5)'/>
            <use xlink:href='#c-horizontal-side-specular' width='100%' height='16.25'/>
            <use xlink:href='#c-horizontal-side-specular' y='-100%' width='100%' height='16.25' transform='scale(1, -1) translate(0 16.24)'/>
        </g>

        <g id='l-sign-horizontal-sides' class='layer'>
            <use xlink:href='#c-clipped-horizontal-side'/>
            <use xlink:href='#c-clipped-horizontal-side' y='-100%' transform='scale(1, -1) translate(0 6.5)'/>
            <use xlink:href='#c-horizontal-side-specular' width='100%' height='16.25' class='frame--color-specular'/>
            <use xlink:href='#c-clipped-horizontal-side-specular' y='-100%' transform='scale(1, -1) translate(0 16.25)'/>
        </g>

        <g id='l-vertical-sides' class='layer'>
            <use xlink:href='#c-vertical-side' class='frame--color-diffuse' width='16.25px' height='100%'/>
            <use xlink:href='#c-vertical-side' class='frame--color-diffuse' x='-100%' width='16.25px' height='100%' transform='scale(-1, 1)'/>
            <use xlink:href='#c-vertical-side-specular' class='frame--color-specular' width='16.25px' height='100%'/>
            <use xlink:href='#c-vertical-side-specular' class='frame--color-specular' x='-100%' width='16.25px' height='100%' transform='scale(-1, 1) translate(10 0)'/>
        </g>

        <g id='l-bulletin-holes' class='layer'>
            <use xlink:href='#bs-hole-depth' transform='translate(0 6.49)' class='frame--color-depth'/>
            <use xlink:href='#bs-hole-depth' x='-100%' transform='scale(-1, 1) translate(0 6.49)' class='frame--color-depth'/>
            <use xlink:href='#c-bulletin-clipped-holes' class='frame--color-depth' clip-path='url(#cp-holes-left)' transform='translate(0 4.32)'/>
            <use xlink:href='#bs-hole' transform='translate(0 6.49)' class='frame--color-diffuse'/>
            <use xlink:href='#bs-hole' x='-100%' transform='scale(-1, 1) translate(0 6.49)' class='frame--color-diffuse'/>
            <use xlink:href='#c-bulletin-clipped-holes' class='frame--color-diffuse'/>
            <use xlink:href='#bs-hole-specular' class='frame--color-specular' transform='translate(0 6.49)'/>
            <use xlink:href='#bs-hole-specular' class='frame--color-specular' x='100%' transform='translate(-32 6.49)'/>
        </g>

        <g id='l-sign-holes' class='layer'>
            <use xlink:href='#bs-hole-depth' transform='translate(0 6.49)' class='frame--color-depth' clip-path='url(#cp-sign-hole-bottom)'/>
            <use xlink:href='#bs-hole-depth' x='-100%' transform='scale(-1, 1) translate(0 6.49)' class='frame--color-depth' clip-path='url(#cp-sign-hole-bottom)'/>
            <use xlink:href='#bs-sign-holes-depth' class='frame--color-depth' transform='translate(0 6.48)' clip-path='url(#cp-sign-holes-depth)'/>
            <use xlink:href='#bs-hole' transform='translate(0 6.49)' class='frame--color-diffuse' clip-path='url(#cp-sign-hole-bottom)'/>
            <use xlink:href='#bs-hole' x='-100%' transform='scale(-1, 1) translate(0 6.49)' class='frame--color-diffuse' clip-path='url(#cp-sign-hole-bottom)'/>
            <use xlink:href='#c-sign-clipped-holes' class='frame--color-diffuse'/>
            <use xlink:href='#bs-hole-specular' class='frame--color-specular' transform='translate(0 6.49)'/>
            <use xlink:href='#bs-hole-specular' class='frame--color-specular' x='100%' transform='translate(-32 6.49)'/>
            <use xlink:href='#bs-hole-depth' y='100%' transform='translate(0 -34.49)' class='frame--color-depth' clip-path='url(#cp-sign-hole-top)'/>
            <use xlink:href='#bs-hole-depth' x='-100%' y='100%' transform='scale(-1, 1) translate(0 -34.49)' class='frame--color-depth' clip-path='url(#cp-sign-hole-top)'/>
            <use xlink:href='#bs-hole' y='-100%' transform='scale(1, -1) translate(0 12.98)' class='frame--color-diffuse' clip-path='url(#cp-sign-hole-bottom)'/>
            <use xlink:href='#bs-hole' x='-100%' y='-100%' transform='scale(-1,-1) translate(0 12.98)' class='frame--color-diffuse' clip-path='url(#cp-sign-hole-bottom)'/>
            <use xlink:href='#bs-hole-specular' class='frame--color-specular' y='100%' transform='translate(0 -34.24)'/>
            <use xlink:href='#bs-hole-specular' class='frame--color-specular' y='100%' x='100%' transform='translate(-32 -34.24)'/>
            <use xlink:href='#c-sign-clipped-holes' y='-100%' transform='scale(1, -1) translate(0 6.49)' class='frame--color-diffuse'/>
            <use xlink:href='#c-clipped-bottom-holes-specular' class='frame--color-specular' clip-path='url(#cp-bottom-holes-specular-left)'/>
            <use xlink:href='#bs-bottom-hole-specular' class='frame--color-specular' y='100%' transform='translate(0 -36.4)'/>
            <use xlink:href='#bs-bottom-hole-specular' class='frame--color-specular' x='-100%' y='100%' transform='scale(-1, 1) translate(0 -36.4)'/>
        </g>

        <g id='l-horizontal-sides-depth' class='layer'>
            <use xlink:href='#c-horizontal-side' class='frame--color-depth' height='16.25' width='100%' transform='translate(0 4.32)' clip-path='url(#cp-horizontal-side-center)'/>
            <use xlink:href='#c-horizontal-side' class='frame--color-depth' y='-100%' height='16.25' width='100%' transform='scale(1, -1)'/>
        </g>

        <g id='l-bulletin-corners-depth' class='layer'>
            <use xlink:href='#bs-bulletin-corner' transform='translate(0 4.32)' class='frame--color-depth'/>
            <use xlink:href='#bs-bulletin-corner' x='-100%' transform='scale(-1, 1) translate(0 4.32)' class='frame--color-depth'/>
            <use xlink:href='#bs-bulletin-corner' x='-100%' y='-100%' transform='scale(-1, -1)' class='frame--color-depth'/>
            <use xlink:href='#bs-bulletin-corner' y='-100%' transform='scale(1, -1)' class='frame--color-depth'/>
            <use xlink:href='#bs-bulletin-corner-depth-fill' x='0' y='100%' transform='translate(0 -53.5)' class='frame--color-depth'/>
            <use xlink:href='#bs-bulletin-corner-depth-fill' x='-100%' y='100%' transform='scale(-1, 1) translate(0 -53.5)' class='frame--color-depth'/>
        </g>

        <g id='l-sign-corners-depth' class='layer'>
            <use xlink:href='#bs-sign-corner' transform='translate(0 4.32)' class='frame--color-depth'/>
            <use xlink:href='#bs-sign-corner' x='-100%' transform='scale(-1, 1) translate(0 4.32)' class='frame--color-depth'/>
            <use xlink:href='#bs-sign-corner' x='-100%' y='-100%' transform='scale(-1, -1)' class='frame--color-depth'/>
            <use xlink:href='#bs-sign-corner' y='-100%' transform='scale(1, -1)' class='frame--color-depth'/>
            <use xlink:href='#bs-bulletin-corner-depth-fill' x='0' y='100%' transform='translate(0 -27)' class='frame--color-depth'/>
            <use xlink:href='#bs-bulletin-corner-depth-fill' x='-100%' y='100%' transform='scale(-1, 1) translate(0 -27)' class='frame--color-depth'/>
        </g>

        <g id='l-horizontal-sides-shadow' class='layer'>
            <use xlink:href='#c-horizontal-side' clip-path='url(#cp-horizontal-side-center)' class='frame--color-shadow' transform='translate(0 6.48)' height='16.25' width='100%'/>
        </g>

        <g id='l-vertical-sides-shadow' class='layer'>
            <use xlink:href='#c-vertical-side' class='frame--color-shadow' width='16.25px' height='100%' transform='translate(2.16 0)'/>
            <use xlink:href='#c-vertical-side' class='frame--color-shadow' x='-100%' width='16.25px' height='100%' transform='scale(-1, 1) translate(2.16 0)'/>
        </g>

        <g id='l-bulletin-corners-shadow' class='layer'>
            <use xlink:href='#bs-bulletin-corner' class='frame--color-shadow' transform='translate(2.16 6.48)'/>
            <use xlink:href='#bs-bulletin-corner' x='-100%' transform='scale(-1, 1) translate(2.16 6.48)' class='frame--color-shadow'/>
        </g>

        <g id='l-sign-corners-shadow' class='layer'>
            <use xlink:href='#bs-sign-corner' class='frame--color-shadow' transform='translate(2.16 6.48)'/>
            <use xlink:href='#bs-sign-corner' x='-100%' transform='scale(-1, 1) translate(2.16 6.48)' class='frame--color-shadow'/>
        </g>

        <g id='l-bulletin-holes-shadow' class='layer'>
            <rect x='0' y='0' width='100%' height='62.48px' rx='21.51px' ry='21.51px' clip-path='url(#cp-bulletin-holes)' class='frame--color-shadow'/>
        </g>

        <g id='l-sign-holes-shadow' class='layer'>
            <rect x='0' y='0' width='100%' height='36.27px' rx='13px' ry='13px' clip-path='url(#cp-sign-holes)' class='frame--color-shadow'/>
        </g>

        <g id='l-bulletin-planks' class='layer'>
            <use xlink:href='#c-bulletin-clipped-planks' fill='url(#p-planks)'/>
            <use xlink:href='#c-bulletin-clipped-planks' fill='url(#p-shades)'/>
        </g>

        <g id='l-sign-planks' class='layer'>
            <use xlink:href='#c-sign-clipped-planks' fill='url(#p-planks)'/>
            <use xlink:href='#c-sign-clipped-planks' fill='url(#p-shades)'/>
        </g>

        <g id='l-top-chains' class='layer'>
            <use xlink:href='#c-top-chain' transform='translate(6 0)'/>
            <use xlink:href='#c-top-chain' x='100%' transform='translate(-26 0)'/>
        </g>

        <g id='l-bottom-chains' class='layer'>
            <use xlink:href='#c-bottom-chain' transform='translate(6 0)'/>
            <use xlink:href='#c-bottom-chain' x='100%' transform='translate(-26 0)'/>
</g>
```
With this we conclude on the SVG side. In the [next post][ch-12], we will see how to build both types of boards out of this single file with HTML and CSS.



[ch-1]: ../Making-a-responsive-medieval-board-with-SVG-stacks
[ch-2]: ../Making-an-SVG-stretch
[ch-3]: ../Inverting-SVG-elements
[ch-4]: ../Rotating-SVG-elements
[ch-5]: ../Clipping-SVG-elements
[ch-6]: ../Improving-organization-with-defs
[ch-7]: ../Namespacing
[ch-8]: ../Styling-an-SVG
[ch-9]: ../Using-SVG-patterns
[ch-10]: ../Clipping-SVG-elements-with-complex-shapes
[ch-11]: ../Using-an-SVG-stack-with-fragment-identifiers
[ch-12]: ../Getting-Sassy-with-the-board
[ch-13]: ../When-Sass-met-BEM
[ch-14]: ../Browser-support-for-SVG-stacks
[ch-15]: ../Sass-mixins-for-SVG-stacks
[ch-16]: ../Combining-modernizr-detects
[ch-17]: ../Tiling-with-Sass
[ch-18]: ../Error-handling-in-Sass
[objectives]: ../Making-a-responsive-medieval-board-with-SVG-stacks/#the-objectives
[CSS-Tricks]: https://css-tricks.com/svg-fragment-identifiers-work/
