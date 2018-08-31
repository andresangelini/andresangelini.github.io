### Clipping SVG elements

Looking carefully at the top side of the frame of the medieval board, you'll notice that if left as it is, it will visible right through the holes of the top chains. Since we want to reuse this piece for all four sides of the frame, we would have to find some way to "cut" a hole right where the holes are without modifying the other sides. This is where the `clip-path` attribute comes into play. Its only parameter, `url()`, allows us to reference to **a path or any basic shape** to define **the visible area** of our element. This might feel counter intuitive at first since most image manipulation software out there have us accostumed to the opposite. Also, note that `<symbol>` elements cannot be used as `clip-path`. This is quite sad because their ability of being stretcheable through the use of `viewBox` and `preserveAspectRatio` would really come in handy. Fortunately for us, there is another way this can be dealt with. Let's try to remove the center part of the frame's top side where the entire title area should be. Remember though, what we have to define is the visible area, which means we need to use three separated pieces: one to cover just a little bit of top side in its entire width, and one for each side.

```xml
<svg version="1.1" xmlns="http://www.w3.org/2000/svg">

  <!-- Clip path -->
  <clipPath id='top-side-clip-path'>
    <rect x='0' y='0' width='100%' height='7'/>
    <rect x='-90%' y='0' width='100%' height='100%' transform='translate(3.16 0)'/>
    <rect x='90%' y='0' width='100%' height='100%' transform='translate(-3.16 0)'/>
  </clipPath>

  <!-- Reusable graphic or template. -->
  <symbol id='horizontal-side' width='100%' height='16.25' viewBox='0 0 905 16.25' preserveAspectRatio='none'>
    <path d='m 0,0 c 301.53,5.7865 603.05,5.7864 904.58,0 l 0,11.9 C 603.1,17.691 301.57,17.692 0,11.9 Z' fill='#442d18'/>
  </symbol>

  <!-- Use the graphic. -->
  <use xlink:href='#horizontal-side' clip-path='url(#top-side-clip-path)'/>
</svg>
```
The end result should look something like this:

<p data-height="265" data-theme-id="0" data-slug-hash="ooqder" data-default-tab="html,result" data-user="andresangelini" data-embed-version="2" data-pen-title="clip-path on a stretchable SVG element" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/ooqder/">clip-path on a stretchable SVG element</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async="async" src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

That's it for now. In the next post, we will be taking a short detour to talk about how to orginze our SVG elements to work more efficiently.
