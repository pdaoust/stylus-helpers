# Stylus helpers

A few functions and mixins to help do cool things in Stylus. The two I'm proudest of are:

## rhythm

Create block styles that conform to a baseline grid or vertical rhythm. Syntax:

    rhythm(fontsize [, baseline [, margintop [, marginbottom [, squash]]]])

* **`fontsize`**: a unit relative to the document's (or parent element's) base font size. Will default to 1em.

* **`baseline`**: the baseline grid unit for the document. Will default to 1.5em if not specified and a `baseline` variable is not specified anywhere in your CSS.

* **`margintop`**: the top margin (in baseline units, not ems). Defaults to 1.

* **`marginbottom`**: the bottom margin. Ditto.

* **`squash`**: the minimum line-height the element should tolerate. Defaults to 1.

## blend

Blend two colours together, with an optional Porter-Duff blending mode. Especially useful for creating opaque colours that mimic RGBA for browsers that don't support it (e.g., Internet Explorer). Syntax:

    blend(colour1, colour2 [, amount [, blendmode]])

* **`colour1`**: the background colour

* **`colour2`**: the foreground colour

* **`amount`**: the alpha or alpha modifier of `colour2`; can be a float or
percentage. If blank or `false`, it'll try to get the alpha value from
`colour2`, and lastly, default to 50% for normal and 100% for all other
blending modes. If `colour2` has an alpha channel, it is reduced by this value.

* **`blendmode`**: the blending or compositing mode, as explained by the W3C
draft specs for [compositing and blending in SVG](http://dev.w3.org/SVG/modules/compositing/master/) and [CSS](https://dvcs.w3.org/hg/FXTF/rawfile/tip/compositing/index.html). Note:
some of the formulae presented in the spec seem to produce completely
wrong results; I could use some help from people who know something
about Messrs Porter and Duff.

### A big caveat

Because no (or at least very few) web browsers actually support blending modes, please understand that this is completely _fake_ compositing. It will only work with solid RGBA or RGB colours over top of solid RGBA or RGB colours.

### Interpreting the code against the reference formulae

I've used different variable names from the reference formulae presented in the W3C specs; here's a translation guide:

<table>
	<thead>
		<tr>
			<th>Spec variable</th>
			<th>My variables</th>
			<th>What it means</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>Da</td>
			<td>alpha1</td>
			<td>alpha of foreground colour</td>
		</tr>
		<tr>
			<td>Sa</td>
			<td>alpha2</td>
			<td>alpha of background colour</td>
		</tr>
		<tr>
			<td>Dca</td>
			<td>colour1a</td>
			<td>foreground colour, premultiplied by its alpha</td>
		</tr>
		<tr>
			<td>Sca</td>
			<td>colour2a</td>
			<td>background colour, premultiplied by its alpha</td>
		</tr>
		<tr>
			<td>m</td>
			<td>colour2n</td>
			<td>background colour, de-premultiplied, used only in the soft-light blending mode</td>
		</tr>
		<tr>
			<td>Da'</td>
			<td>colour3</td>
			<td>alpha of final colour</td>
		</tr>
		<tr>
			<td>Dca'</td>
			<td>colour3a</td>
			<td>Blended colour, before de-multiplication by final alpha</td>
		</tr>
	</tbody>
</table>

And I also used a shorthand function called `comp()`, because it was a
formula that popped up often in the different blending mode formulae.
It multiplies a colour by 1 - its complement's alpha. Hence:

    comp(colour1a, alpha2) = colour1a * (1 - alpha2)
    comp(colour2a, alpha1) = colour2a * (1 - alpha1)

### And now for the blending modes!

* **normal** (default): the standard blending mode, equivalent to `src-over` compositing, like translucent tracing paper

* **multiply**: darken the background by the foreground, like stained glass

* **screen**: lighten the background by the foreground, like shining two lights on a surface

* **plus**: similar to screen, only stronger -- called 'addition' in most
graphics applications

* **overlay**: multiplies for darker colours, screens for lighter colours,
but not as strong as either

* **darken**: darken the background by the foreground; similar to multiply,
but gentler, because it only darkens the background if the background
is lighter than the foreground

* **lighten**: lighten the background by the foreground; similar to screen,
but gentler, because it only lightens the background if the background
is darker than the foreground

* **color-dodge**: _(broken)_ brighten the background by the foreground. Quite a
strong effect compared to screen, and creates lurid colours. The reference formulae I found in the CSS spec do not work at all; I could use some
help with this one.

* **color-burn**: _(broken)_the background is darkened by the foreground. Again, a
stronger effect than multiply. And again, the reference formulae I
found do not work.

* **hard-light**: _(broken)_ a very strong darken/lighten effect; stronger than
overlay. It almost works perfectly; some colours are a bit more
intense than reference examples of hard-light.

* **soft-light**: _(broken)_ a very mild darken/lighten effect, similar to overlay.
The reference formulae don't work in the least; they give a nice
effect (halfway between normal and overlay), but it doesn't resemble
soft-light at all.

* **difference**: subtract the foreground from the background. Light
foregrounds have the effect of reversing background colours.

* **exclusion**: similar to difference, but milder.

## calculators

Currently only contains two functions for converting your fancy Photoshop mockup to relative, scaleable stylesheets: `px2em()` and `em2px()`. They do what they say on the can, assuming that the base font size is `100%` or `1em`. If you want a different base font size (e.g., if you're setting a width on an inline-block element inside an `<h1>` whose font size is `2em`), either put it as the second argument, or set a variable called `basefontsize` somewhere in your stylesheet. (**Note**: this variable should reflect the actual font size in your `:root` (i.e., `<html>`) element or your `<body>` element, and should be a relative value. You'll find it quite useless for `px`- or `pt`-based layouts.)

These calculators do not take into account the user agent's base font size, so if their font size is larger than 16px, a CSS pixel calculated using these methods will be larger than a device pixel. This is by design, so that layouts will scale up with the user agent's font size. After all, that's what we use relative units for!

    em2px(ems [, fontsize])
    px2em(px [, fontsize])

## mixins

A set of mixins I find myself using regularly.

* **`collapseinnermargins()`**: set this on a container whose first and last direct descendants should be butted up agains the top and bottom of the container (e.g., to remove the `margin-top` from an `<h2>` and the `margin-bottom` from the final `<p>`). Especially useful whenever you create a new flow context (floats, `overflow: auto`, containers with borders, etc).

* **`inlineblock()`** and **`.inlineblock`**: includes `hasLayout` and `-moz-inline-stack` hacks for legacy browsers.

* **`clearfix()`**, **`cf()`**, **`.clearfix`**, and **`.cf`**: A cross-browser clearfix that can be relied upon to work almost anywhere. A bit heavier than other clearfixes, but works great. I also included a `cf` alias for both the function and class, for convenience.

* **`shadeborder(basecolor, amount)`**: shades the border as if there's a light above -- the top is lightened and the bottom is darkened by the specified amount.
