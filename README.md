# Stylus helpers

A few functions and mixins to help do cool things in Stylus. The two I'm proudest of are:

## rhythm

Create block styles that conform to a baseline grid or vertical rhythm. Syntax:

```
rhythm(fontsize [, baseline [, margintop [, marginbottom [, squash]]]])
```

* fontsize: a unit relative to the document's (or parent element's) base font size. Will default to 1em.

* baseline: the baseline grid unit for the document. Will default to 1.5em.

* margintop: the top margin (in baseline units, not ems). Defaults to 1.

* marginbottom: the bottom margin. Ditto.

* squash: the minimum line-height the element should tolerate. Defaults to 1.

## blend

Blend two colours together, with an optional Porter-Duff blending mode. Especially useful for creating opaque colours that mimic RGBA for browsers that don't support it (e.g., Internet Explorer). Syntax:

```
blend(colour1, colour2 [, amount [, blendmode]])
```

* colour1: the background colour

* colour2: the foreground colour

* amount: the alpha or alpha modifier of colour2; can be a float or
percentage. If blank or false, it'll try to get the alpha value from
colour2, and lastly, default to 50% for normal and 100% for all other
blending modes. If colour2 has alpha, it is reduced by this value

* blendmode: the blending or compositing mode, as explained by the W3C
draft specs for [compositing and blending in SVG](http://dev.w3.org/SVG/modules/compositing/master/) and [CSS](https://dvcs.w3.org/hg/FXTF/rawfile/tip/compositing/index.html). Note:
some of the formulae presented in the spec seem to produce completely
wrong results; I could use some help from people who know something
about Messrs Porter and Duff.

I've used different variable names from the reference formulae presented
in the specs; here's a translation guide:

* Da  = alpha1 (foreground alpha)
* Sa  = alpha2 (background alpha)
* Dca = colour1a (foreground colour, premultiplied by its alpha)
* Sca = colour2a (background colour, premultiplied by its alpha)
* m   = colour2n (background colour, de-premultiplied, used only in the soft-light blending mode)

And I also used a shorthand function called comp(), because it was a
formula that popped up often in the different blending mode formulae.
It multiplies a colour by 1 - its complement's alpha. Hence:

    comp(colour1a, alpha2) = colour1a * (1 - alpha2)
    comp(colour2a, alpha1) = colour2a * (1 - alpha1)

And now for the blending modes!

* normal (default): the usual, like translucent tracing paper

* multiply: darken colour1 by colour2, like stained glass

* screen: lighten colour1 by colour2, like shining two lights on a surface

* plus: similar to screen, only stronger -- called 'addition' in most
graphics applications

* overlay: multiplies for darker colours, screens for lighter colours,
but not as strong as either

* darken: darken the background by the foreground; similar to multiply,
but gentler, because it only darkens the background if the background
is lighter than the foreground

* lighten: lighten the background by the foreground; similar to screen,
but gentler, because it only lightens the background if the background
is darker than the foreground

* color-dodge: brighten the background by the foreground. Quite a
strong effect compared to screen, and creates lurid colours. NOTE:
the reference formulae I found do not work at all; I could use some
help with this one.

* color-burn: the background is darkened by the foreground. Again, a
stronger effect than multiply. And again, the reference formulae I
found do not work.

* hard-light: a very strong darken/lighten effect; stronger than
overlay. It almost works perfectly; some colours are a bit more
intense than reference examples of hard-light.

* soft-light: a very mild darken/lighten effect, similar to overlay.
The reference formulae don't work in the least; they give a nice
effect (halfway between normal and overlay), but it doesn't resemble
soft-light at all.

* difference: subtract the foreground from the background. Light
foregrounds have the effect of reversing background colours.

* exclusion: similar to difference, but milder.
