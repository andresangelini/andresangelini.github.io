### Improving organization with `<defs>`

Up until now, I've been purposely avoiding this topic and thus doing unnecessary extra work so that you can come to understannd how important it is. I'm talking about, of course, `<defs>` and how it can help us to follow the [D.R.Y] principle.

If we look closely at our previous example, it's clear that most of the elements are basically the same but with some minor differences such as their position, size, etc. So it would be only logical to define some basic shapes and use them as templates for the rest of the parts. Well, meet `<defs>`. This element allows us to do just that; "define" a bunch of reusable elements without actually displaying them.

Let's see a simple example of how this would work:

<p data-height="265" data-theme-id="0" data-slug-hash="aVVrBW" data-default-tab="html,result" data-user="andresangelini" data-embed-version="2" data-pen-title="Defs in SVGs" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/aVVrBW/">Defs in SVGs</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

As you can see, we "define" a basic shape and then `<use>` it inside a `<symbol>` to make it stretch, all inside the `<defs>`. Up to this point, neither the `<path>` nor the `<symbol>` are displayed. Lastly, we `<use>` the stretcheable `<symbol>` version elsewhere, outside the `<defs>`.

This is a much cleaner way of doing things but we are not quite yet. There is still one more topic we need to touch on which will in the next post.
