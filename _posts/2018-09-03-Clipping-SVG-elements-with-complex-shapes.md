### Clipping SVG elements with complex shapes.

As you have probably realized already, we can't just apply these patterns to simple rectangles because they would be visible through the board's frames and holes. We obviously need to clip them, but here's the catch: while the corners to be removed are of a fixed size, the parts that need to be visible must stretch. Remember also, that what we define by using clip-path is the visible area and not the other way around. However, we have no way of defining a clip-path that has these characteristics. Nonetheless, we can achieve the same result by, once again, taking a more indirect approach.

The parts we need to remove are the four corners, the area where the holes for the chains are, and a tiny bit of each side. Since we need to define the area we want displayed and this area needs to stretch while keeping a constant offset in pixels, we might as well create some `<rect>`s and make a composition with them to get an aproximation of the shape we are after.

We start off with the `<clipPath>` definitions. Each `<clipPath>` will be created by carefully placing one `<rect>` on top of the other to get the shape we want for each of the areas we mentioned earlier.

```xml
<svg version="1.1" xmlns="http://www.w3.org/2000/svg">
  <clipPath id='bulletin-tl-corner'>
    <rect width='100%' height='100%' transform='translate(33 33)'/>
    <rect width='100%' height='100%' transform='translate(46 7.5)'/>
    <rect width='100%' height='100%' transform='translate(7.5 46)'/>
  </clipPath>

  <clipPath id='bulletin-tr-corner'>
      <rect width='100%' height='100%' transform='translate(-7.5 46)'/>
      <rect width='100%' height='100%' transform='translate(-33 33)'/>
      <rect width='100%' height='100%' transform='translate(-46 7.5)'/>
    </clipPath>

    <clipPath id='bulletin-br-corner'>
      <rect width='100%' height='100%' transform='translate(-7.5 -46)'/>
      <rect width='100%' height='100%' transform='translate(-33 -33)'/>
      <rect width='100%' height='100%' transform='translate(-46 -7.5)'/>
    </clipPath>

    <clipPath id='bulletin-bl-corner'>
      <rect width='100%' height='100%' transform='translate(7.5 -46)'/>
      <rect width='100%' height='100%' transform='translate(33 -33)'/>
      <rect width='100%' height='100%' transform='translate(46 -7.5)'/>
    </clipPath>

    <clipPath id='bulletin-holes'>
      <rect width='100%' height='100%' x='-90%' transform='translate(46 0)'/>
      <rect width='100%' height='100%' x='90%' transform='translate(-46 0)'/>
      <rect width='100%' height='100%' transform='translate(0 33)'/>
    </clipPath>
```

Here comes the interesting part. Since we can't apply the `clip-path`s altogether, we'll have to do it one at a time by applying each one to the result of the previous, like you see in the following pen.

<p data-height="265" data-theme-id="0" data-slug-hash="vpYbPP" data-default-tab="result" data-user="andresangelini" data-embed-version="2" data-pen-title="Clipping SVG elements with complex shapes" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/vpYbPP/">Clipping SVG elements with complex shapes</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async="async" src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

First, we apply our first `clip-path` on a simple `<rect>`, like we've been doing so far, and then wrap it up in a `<symbol>` because we don't want it to be seen. Of course, we add an `id` attribute to this `<symbol>` so that we can reference to it. Please note that this differs from what you see in the pen example because in that case, we do want to display each stage. If we were to display it, we would see what we see in [figure 1][complex clip-path pen].

```xml
...
  <symbol id='clipped-bulletin-tl-corner'>
    <rect width='100%' height='100%' clip-path='url(#bulletin-tl-corner)'/>
  </symbol>
```

Now, we apply the second `clip-path` to a copy of this result, and again, we wrap it up in a `<symbol>` element to hide it.

```xml
...
  <symbol id='clipped-bulletin-tr-corner'>
    <use xlink:href='#clipped-bulletin-tl-corner' clip-path='url(#bulletin-tr-corner)'/>
  </symbol>
```

And we repeat this same process for the rest of the `clip-path`s.

```xml
...
  <symbol id='clipped-bulletin-br-corner'>
    <use xlink:href='#clipped-bulletin-tr-corner' clip-path='url(#bulletin-br-corner)'/>
  </symbol>

  <symbol id='clipped-bulletin-bl-corner'>
    <use xlink:href='#clipped-bulletin-br-corner' clip-path='url(#bulletin-bl-corner)'/>
  </symbol>

  <symbol id='clipped-bulletin-planks'>
    <use xlink:href='#clipped-bulletin-bl-corner' clip-path='url(#bulletin-holes)'/>
  </symbol>
</svg>
```

You can see the resulting shape in [figure 5][complex clip-path pen] in the code pen above together with all the previous stages leading up to it.

All there is to do now is just making two copies of this clipped shape (one for the planks and one for their shading) and add a `fill` attribute to apply our previously made `pattern`s and voila! We get the planks and shading pattern fitting perfectly into the table.

```xml
...
  <use xlink:href='#clipped-bulletin-planks' fill='url(#planks)'/>
  <use xlink:href='#clipped-bulletin-planks' fill='url(#shades)'/>
```

It should look like [figure 4][pattern on a complex clipped element] of this new code pen:

<p data-height="265" data-theme-id="0" data-slug-hash="dJozLp" data-default-tab="result" data-user="andresangelini" data-embed-version="2" data-pen-title="Applying an SVG pattern on a clipped complex shape" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/dJozLp/">Applying an SVG pattern on a clipped complex shape</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async="async" src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

Now we can do the same with the metal plaque where the title goes. But let's recap how its structure is and what we need to do here first.

The plaque is made with two peaces of fixed size on each side and one center piece, which is what actually need to stretch. That is what we have just done with the wooden pattern of the board. We only need to change a few things.

First, we add our basic shapes inside the `defs`.

```xml
<defs>
...
  <path id='bs-plaque-side' d='...'/>
  <rect id='bs-plaque-center' x='0' y='9.16' width='100%' height='35.67'/>
  <path id='bs-plaque-specular-left' d='...'/>
  <path id='bs-plaque-specular-right' d='...'/>
  <path id='bs-plaque-side-shadow' d='...'/>

  <circle id='bs-nail' cx='15.095px' cy='17.831px' r='4.866px'/>
  <path id='bs-nail-specular' d='...'/>
</defs>
```

Then, we crate the `clip-paths`, also inside the `defs`. Since we define the visible area, we'll just create two of them covering the full length of the plaque and slide one of them slightly to the right and the other slightly to the left.

```xml
<defs>
...
  <clipPath id='cp-plaque-left-side'>
    <rect x='18' y='0' width='100%' height='100%'/>
  </clipPath>

  <clipPath id='cp-plaque-right-side'>
    <rect x='-18' y='0' width='100%' height='100%'/>
</clipPath>
</defs>
```

We are ready now to start building our components. We start off with the nails since they are very easy to make.

```xml
<defs>
...
  <symbol id='c-plaque-nail'>
    <use href='#bs-nail' class='nail--color-depth' transform='translate(0 2.08)'/>
    <use href='#bs-nail' class='nail--color-diffuse'/>
    <use href='#bs-nail-specular' class='nail--color-specular'/>
  </symbol>
</defs>
```

The only thing worth mentioning here is the order we `use` our basic shapes. Just like in HTML, what we add in a line in the editor goes on "top" or closer to our point of view (or Z axis) and thus covering whatever is "below" or behind it, so when building our components we start with the shadow layer, then the depth, followed by the diffuse one and finally the specular one, if present. The plaque center is one more example of this. We will build it in this way as a `group` so that we can `clip` next.

```xml
<defs>
  <g id='c-unclipped-plaque-center'>
    <use href='#bs-plaque-center' class='plaque--color-shadow' transform='translate(0 4.32)'/>
    <use href='#bs-plaque-center' class='plaque--color-depth' transform='translate(0 2.16)'/>
    <use href='#bs-plaque-center' class='plaque--color-diffuse'/>
    <rect x='0' y='9.16' width='100%' height='2.16' class='plaque--color-specular'/>
  </g>
</defs>
```

Finally, we `clip` it in a simple two step process by applying one of our `clip-path`s to an instance of the plaque center and then aplpying the other `clip-path` to that same clipped instance we got as a result, just like we did earlier with bulletin corners.

```xml
<defs>
...
  <use id='c-clipped-plaque-left' href='#c-unclipped-plaque-center' clip-path='url(#cp-plaque-left-side)'/>
  <use id='clipped-plaque-center' href='#c-clipped-plaque-left' clip-path='url(#cp-plaque-right-side'/>
</defs>
```

Now they are ready to `use` anytime you like. The end result should look something like this:

<p data-height="331" data-theme-id="0" data-slug-hash="YOzYxL" data-default-tab="html,result" data-user="andresangelini" data-pen-title="stretching only some areas of an SVG" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/YOzYxL/">stretching only some areas of an SVG</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

This is quite literally the last piece we needed to build the board. Although we haven't finished yet, the rest of our tasks should be somehow trivial.





[top chains svg]: https://cdn.rawgit.com/andresangelini/f3415703d9665bc6d2e0fcdefd90c252/raw/8a9d7f56730094d681762638c76db1df3ffdd538/top chains svg.svg "Top chains"
[bottom chains svg]: https://cdn.rawgit.com/andresangelini/96fc2fe2937f63997f972f203509bb28/raw/04eb599bf86ffce922d53071c8a10013743a3436/bottom chains svg.svg "Bottom chains"
[chains pen]: https://codepen.io/andresangelini/pen/xPBapB/
[complex clip-path pen]: https://codepen.io/andresangelini/pen/vpYbPP/
[pattern on a complex clipped element]: https://codepen.io/andresangelini/pen/dJozLp
