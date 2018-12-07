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

### Using SVG `<pattern>`s

The chains for the medieval board we have being working on [up until now][ch-8] through this series of posts about [Making A Responsive Medieval Board With SVG Stacks][ch-1] are perfect for introducing a new type of element: `<pattern>`. As its name implies, this element will let us use one or more elements to create repeatable graphics to fill other elements with. The good news is that, besides basic shapes, it also allows referencing to `<symbol>` through `<use>` elements, which means we can use stretchables elements inside the `<pattern>`. Although we won't need this feature for our chains because they don't stretch, we will need it later on for making the board's wood texture. But for now, let's focus on the chains.

Before attemphing anything, let's see how the chains are made up to better understand what we need to do.

Top chains:

![Top chains][top chains svg]

Bottom chains:

![Bottom chains][bottom chains svg]

At a glance, we can clearly see there are two main pieces: a link seen from the front and another from the side, each with three levels of shading, like this:

<p data-height="265" data-theme-id="0" data-slug-hash="vWPRxz" data-default-tab="result" data-user="andresangelini" data-embed-version="2" data-pen-title="chain link composition" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/vWPRxz/">chain link composition</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async="async" src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

So, if were to split the chains up into their basic shapes, this is what we'd had:

```xml
<svg version="1.1" xmlns="http://www.w3.org/2000/svg">
  <symbol id='link-profile-diffuse'>
    <rect x='0' y='0' width='6.6490554' height='37.4776338' rx='3.32452782' ry='3.32452782' fill='#433d18'/>
  </symbol>
  <symbol id='link-profile-specular'>
    <path d='m 3.3096216...' fill='#746b2e'/>
  </symbol>
  <symbol id='link-profile-shadow'>
    <path d='m 4.4817173...' fill='#2a260f'/>
  </symbol>
  <symbol id='link-front-diffuse'>
    <path d='M 10.488082...' fill='#433d18'/>
  </symbol>
  <symbol id='link-front-specular'>
    <path d='M 10.488082...' fill='#746b2e'/>
  </symbol>
  <symbol id='link-front-shadow'>
    <path d='m 16.919798...' fill='#2a260f'/>
  </symbol>
</svg>
```

Please note that I have shortened the `d` attribute for simplicity. Check the Codepen snippet down below to see the complete version. Anyways, we have a total of six basic shapes; one rect with rounded corners and five paths made on a graphical application, in my case. Now we'll use these new shapes to make two compositions representing each link.

```xml
...

  <symbol id='link-profile'>
    <use xlink:href='#link-profile-diffuse'></use>
    <use xlink:href='#link-profile-shadow'></use>
    <use xlink:href='#link-profile-specular'></use>
  </symbol>

  <symbol id='link-front'>
    <use xlink:href='#link-front-diffuse'></use>
    <use xlink:href='#link-front-shadow'></use>
    <use xlink:href='#link-front-specular'></use>
  </symbol>
```

Now, let's try to make a pattern out of each of them and then offset the front links to make our final composition with both types of links.

```xml
...
  <pattern id='links-profile' x='0' y='100%' width='100%' height='45.9403662' patternUnits='userSpaceOnUse'>
    <use xlink:href='#link-profile' transform='translate(7 0)'></use>
  </pattern>
```

The `<pattern>`'s `height`, `width` and `patternUnit` attributes are extremely important. The first two define the size of the pattern to be repeated, while the last one sets whether these values, as well as the elemets' inside `<pattern>`, are relative to the global coordinate system or to the element which `<pattern>` is applied to. It's important to note that with the later, values range from `0` to `1`, where `1` it's equivalent to `100%` and `0.5` is equivalent to `50%`. The `patternUnits` values in question are `userSpaceOnUse` (the default one), to use the global coordinates system, and `objectBoundingBox`, to use the local ones.

With all this new information under our belt, we determine that we want the width of our pattern to take the whole width of its container, and that its height should be `45.94` pixels, which is equal to a link's height plus the space between the links. We also want these values to be independent of the container so we make sure to set `patternUnits='userSpaceOnUse'`, even though this is the default value to avoid having issues with some browsers. Then, we center the links horizontally by setting `transform='translate(7 0)'` on the link graphic inside `<pattern>`. Finally, we use a `<rect>` with a `width` equal or bigger to `23` pixels, since that's the width of the chains, and a `height` of `100%` to take the full height of the SVG. The reason for this is that we want to use the SVG boundries as a mask and hide out anything outside them, just like we do when using `overflow: hidden;` in CSS. To apply our `<pattern>` on this element, we have to introduce a new attribute: `fill`. Although it's commonly used to `fill` elements with a plain color, it might also be used to reference to a `<pattern>` using the `url(#elementIdName)` we've seen earlier with the `clip-path` attribute.

```xml
...
  <rect width='23' height='100%' fill='url(#links-profile)'/>
```

The result looks like the [figure 1][chains pen] in the following codepen:

<p data-height="338" data-theme-id="0" data-slug-hash="xPBapB" data-default-tab="result" data-user="andresangelini" data-embed-version="2" data-pen-title="Making an extensible chain with SVG patterns" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/xPBapB/">Making an extensible chain with SVG patterns</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async="async" src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

So far so good, but there is one more thing we need to do. As it is now, the profile links are sticked to the top, and since these ones are meant to be used for the top chains, we want them to stick to the bottom as in [figure 2][chains pen]. To do that, we just set `y='100%'` on the `<pattern>` itself, then translate its content `8.5` pixels downwards with `transform='translate(0 8.5)'`, and that's it!

Now we can move onto doing the same with the front links.

```xml
...
  <pattern id='links-front' x='0' y='100%' width='100%' height='45.9403662' patternUnits='userSpaceOnUse'>
    <use xlink:href='#link-front' transform='translate(0 8.4927324)'></use>
  </pattern>

  <rect width='23' height='100%' fill='url(#links-front)'/>
```

We basically repeat the same process, only this time using the front link element as the `<pattern>`, as seen in [figure 3][chains pen]. All there is to do now is to move these front links upwards. We determine that the amount should be something around `-23` pixels, so we go ahead and set a new `<rect>` with the same values as the previous one for all atributes except `y`, which we set to be equal to `-23` pixels.

```xml
...
  <rect y='-23' width='23' height='100%' fill='url(#links-front)'/>
```

And... what've just happened here? By looking at [figure 4][chains pen], it seems that the `<pattern>` doesn't "follow" our `<rect>`. Remember that we set `patternUnits` to `userSpaceOnUse`? Well, here we are seeing its effect in action. When we did that we told our `<pattern>` to have its position independent from the element it's referenced to and use its own `x` and `y` attributes. You might ask then why we didn't use `objectBoundingBox` instead, but this wouldn't have worked either because that would have meant all our `<pattern>`'s attributes including the attributes of the elements inside it would have been dependent on the `<rect>` dimensions, and we want the `<pattern>` height to be `45,49` pixels.

Fortunately, the solution is as simple as applying a `transform` attribute like this: `transform='translate('0 -23')'`. Why? Because while `x` and `y` set the original position of an element, `transform` just moves it, including any `pattern` it might reference to, like you see in [figure 5][chains pen].

```xml
...
  <rect width='23' height='100%' transform='translate(0 -23)' fill='url(#links-front)'/>
```

The final result as seen in [figure 6][chains pen] is the simple composition of the two pattern of links combined together.

```xml
...
  <symbol id='top-chain'>
    <rect width='23' height='100%' transform='translate(0 -23)' fill='url(#links-front)'/>
    <rect width='23' height='100%' fill='url(#links-profile)'/>
  </symbol>
```

By wrapping these two up in a `<symbol>` we make the chains reusable, which is exactly what we want since we'll need a total of four.

Let's try another application of `<pattern>`. This time, we'll make the wooden texture of the board that we mentioned earlier. Again, we start off by identifying the basic shapes that make out our composition.

- A background filled with a plain color.
- The lit area of the planks' grooves.
- The unlit area of the planks' grooves.

As we can see, the last two ones are actually the same with a different color, so we could just make a copy for each and then assign the colors.

```xml
...
  <symbol id='grooves'>
    <path d='m205.36 680.53c158.58...' opacity='0.5' stroke-width='2.8421' fill='none'/>
  </symbol>
```

Then, we make our new pattern and add a path with a shape of a rectangle to set a background color. We also use two copies of the planks grooves, each one with
a different color and we move the last one a little bit downwards.

```xml
...
<pattern id='planks' x='0' y='0' width='568' height='568' patternUnits='userSpaceOnUse'>
  <path d='m205.37 133.88v568h568v-568z' fill='#a2703f' transform='translate(-205.28 -133.88)'/>
  <use xlink:href='#grooves' stroke='#5f2301'/>
  <use xlink:href='#grooves' stroke='#b98f65' transform='translate(0 2.8421)'/>
</pattern>
```

Finnaly, we apply this new `<pattern>` to a simple `<rect>`.

```xml
...
  <rect width='100%' height='100%' fill='url(#planks)'/>
```

The end result looks like this:

<p data-height="265" data-theme-id="0" data-slug-hash="JMPQpg" data-default-tab="html,result" data-user="andresangelini" data-embed-version="2" data-pen-title="SVG pattern independent of container size" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/JMPQpg/">SVG pattern independent of container size</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async="async" src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

There is one more pattern we need to tackle: the planks shades. For this last one, we want the shades to stretch only on the horizontal axis. Even more, we also want to reuse one side to use it on the opposite side.

As always, we begin by defining a basic shape and make it stretch horizontally at the same time since we are at it.

```xml
...
<symbol id='shade' viewBox='0 0 307 768' preserveAspectRatio='none'>
  <path fill='#673110' opacity='0.1' d='m0 0v764.57h270...'/>
</symbol>
```

The pattern we are going to make requires some more thought, though. We need the shades to stretch horizontally, but not vertically. Since we want to reuse the same basic shape while avoiding being too much repetitive, we'll have to offset each shade, which in turn will make have to create more copies so as to prevent having any gaps in the pattern.

```xml
...
<!-- Define a pattern that stretches horizontally. -->
<pattern id='shades' x='0' y='0' width='100%' height='760' patternUnits='userSpaceOnUse'>
  <g id='left-shades'>
    <!-- Four shades on the left side with different horizontal offsets. -->
    <use id='shade-1' xlink:href='#shade' x='0' y='0' width='50%' height='765'/>
    <use id='shade-2' xlink:href='#shade' x='-15%' y='200' width='50%' height='765'/>
    <use id='shade-3' xlink:href='#shade' x='-20%' y='400' width='50%' height='765'/>
    <use id='shade-4' xlink:href='#shade' x='-30%' y='600' width='50%' height='765'/>
    <!-- Three vertical copies with different vertical offsets. -->
    <g transform='translate(0 -761.5)'>
      <use xlink:href='#shade-2'/>
      <use xlink:href='#shade-3'/>
      <use xlink:href='#shade-4'/>
    </g>
  </g>

  <!-- Copy and invert left side shades to use the right. -->
  <use xlink:href='#left-shades' x='-100%' transform='scale(-1, 1)'/>
</pattern>

<!-- Apply pattern to an element. -->
<rect width='100%' height='100%' fill='url(#shades)'/>
```

Take a better look at the code here:

<p data-height="265" data-theme-id="0" data-slug-hash="wpvYgb" data-default-tab="html,result" data-user="andresangelini" data-embed-version="2" data-pen-title="Having an SVG pattern stretch only horizontally." class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/wpvYgb/">Having an SVG pattern stretch only horizontally.</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async="async" src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

Finally! Exactly what we were looking for. Next up, we will see [how to clip SVG elements with more complex shapes][ch-10].



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
