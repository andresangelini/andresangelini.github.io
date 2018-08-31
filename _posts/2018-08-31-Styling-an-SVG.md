### Styling an SVG

In the last two post we have been polishing things up after learning about SVG's `defs` and namespacing. But there is yet something else we could improve. See that "fill" attribute in our `<use>` element? We have a lot of them sharing the same color among many parts. Wouldn't it be wise to define our colors once [so we don't have to repeat ourselves](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). How could we do that, you ask? Easy. Luckily for us, SVGs have a `<style>` element that cane be used to style any element with using good old CSS like this:

```xml
<style>
        .frame--color-diffuse {fill: #442d18;}
        .frame--color-specular {fill: #76522e;}
        .frame--color-depth {fill: #2b1c0f;}
        .frame--color-shadow {fill: #68300e;}
        .plaque--color-diffuse {fill: #bfa162;}
        .plaque--color-specular {fill: #d7bb7d;}
        .plaque--color-depth {fill: #5f1802;}
        .plaque--color-shadow {fill: #2b1c0f;}
        .nail--color-diffuse {fill: #553d30;}
        .nail--color-specular {fill: #968378;}
        .nail--color-depth {fill: #372318;}
        .planks--color-diffuse {fill: #a2703f;}
        .planks--color-shade {fill: #673110;}
        .planks--color-shadow {stroke: #5f2301;}
        .planks--color-specular {stroke: #b98f65;}
        .link--color-diffuse {fill: #433d18;}
        .link--color-specular {fill: #746b2e;}
        .link--color-shadow {fill: #2a260f;}
</style>
```

The pattern you see in my class names is using the [B.E.M.](http://getbem.com/introduction/) methodology. As such, their structure is [block]-[modifier] where I namespaced the [modifier] for easier reading.





[B.E.M.]: http://getbem.com/introduction/
