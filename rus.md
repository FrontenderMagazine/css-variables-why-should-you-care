CSS variables, more accurately known as CSS custom properties, are landing in
Chrome 49. They can be useful for reducing repetition in CSS, and also for 
powerful runtime effects like theme switching and potentially extending/
polyfilling future CSS features.

## CSS Clutter {#css-clutter}

When designing an application it’s a common practice to set aside a set of
brand colors that will be reused to keep the look of the app consistent. 
Unfortunately, repeating these color values over and over again in your CSS is 
not only a chore, but also error prone. If, at some point, one of the colors 
needs to be changed, you could throw caution to the wind and “find-and-replace” 
all the things, but on a large enough project this could easily get dangerous.

In recent times many developers have turned to CSS preprocessors like SASS or
LESS which solve this problem through the use of preprocessor variables. While 
these tools have boosted developer productivity immensely, the variables that 
they use suffer from a major drawback, which is that they’re static and can’t be
changed at runtime. Adding the ability to change variables at runtime not only 
opens the door to things like dynamic application theming, but also has major 
ramifications for responsive design and the potential to polyfill future CSS 
features. With the release of Chrome 49, these abilities are now available in 
the form of CSS custom properties.

## Custom properties in a nutshell {#custom-properties-in-a-nutshell}

Custom properties add two new features to our CSS toolbox:

The ability for an author to assign arbitrary values to a property with an
author-chosen name. The`var()` function, which allows an author to use these
values in other properties.

Here’s a quick example to demonstrate

    <span class="nd">:root</span> <span class="p">{</span>
      <span class="o">--</span><span class="n">main</span><span class="o">-</span><span class="k">color</span><span class="o">:</span> <span class="m">#06c</span><span class="p">;</span>
    <span class="p">}</span>
    
    <span class="nf">#foo</span> <span class="nt">h1</span> <span class="p">{</span>
      <span class="k">color</span><span class="o">:</span> <span class="n">var</span><span class="p">(</span><span class="o">--</span><span class="n">main</span><span class="o">-</span><span class="k">color</span><span class="p">);</span>
    <span class="p">}</span>

`--main-color` is an author defined custom property with a value of #06c. Note
that all custom properties begin with two dashes.

The `var()` function retrieves and replaces itself with the custom property
value, resulting in`color: #06c;` So long as the custom property is defined
somewhere in your stylesheet it should be available to the`var` function.

The syntax may look a little strange at first. Many developers ask, “Why not
just use`$foo` for variable names?” The approach was specifically chosen to
be as flexible as possible and potentially allow for`$foo` macros in the future
. For the backstory you can read[this post][1] from one of the spec authors,
Tab Atkins.

## Custom property syntax {#custom-property-syntax}

The syntax for a custom property is straightforward.

Note that custom properties are case sensitive, so `--header-color` and 
`--Header-Color` are different custom properties. While they may seem simple at
face value, the allowed syntax for custom properties is actually quite 
permissive. For example, the following is a valid custom property:

    <span class="nt">--foo</span><span class="o">:</span> <span class="nt">if</span><span class="o">(</span><span class="nt">x</span> <span class="o">></span> <span class="nt">5</span><span class="o">)</span> <span class="nt">this</span><span class="nc">.width</span> <span class="o">=</span> <span class="nt">10</span><span class="o">;</span>

While this would not be useful as a variable, as it would be invalid in any
normal property, it could potentially be read and acted upon with JavaScript at 
runtime. This means custom properties have the potential to unlock all kinds of 
interesting techniques not currently possible with today’s CSS preprocessors. So
if you’re thinking
“*yawn* I have SASS so who cares…” then take a second look! These are not
the variables you’re used to working with.

### The cascade {#the-cascade}

Custom properties follow standard cascade rules, so you can define the same
property at different levels of specificity

    <span class="nd">:root</span> <span class="p">{</span> <span class="o">--</span><span class="k">color</span><span class="o">:</span> <span class="nb">blue</span><span class="p">;</span> <span class="p">}</span>
    <span class="nt">div</span> <span class="p">{</span> <span class="o">--</span><span class="k">color</span><span class="o">:</span> <span class="nb">green</span><span class="p">;</span> <span class="p">}</span>
    <span class="nf">#alert</span> <span class="p">{</span> <span class="o">--</span><span class="k">color</span><span class="o">:</span> <span class="nb">red</span><span class="p">;</span> <span class="p">}</span>
    <span class="o">*</span> <span class="p">{</span> <span class="k">color</span><span class="o">:</span> <span class="n">var</span><span class="p">(</span><span class="o">--</span><span class="k">color</span><span class="p">);</span> <span class="p">}</span>

    <span class="nt"><p></span>I inherited blue from the root element!<span class="nt"></p></span>
    <span class="nt"><div></span>I got green set directly on me!<span class="nt"></div></span>
    <span class="nt"><div</span> <span class="na">id=</span><span class="s">"alert"</span><span class="nt">></span>
      While I got red set directly on me!
      <span class="nt"><p></span>I’m red too, because of inheritance!<span class="nt"></p></span>
    <span class="nt"></div></span>

This means you can leverage custom properties inside of media queries to aid
with responsive design. One use case might be to expand the margining around 
your major sectioning elements as the screen size increases:

    <span class="nd">:root</span> <span class="p">{</span>
      <span class="o">--</span><span class="n">gutter</span><span class="o">:</span> <span class="m">4px</span><span class="p">;</span>
    <span class="p">}</span>
    
    <span class="nt">section</span> <span class="p">{</span>
      <span class="k">margin</span><span class="o">:</span> <span class="n">var</span><span class="p">(</span><span class="o">--</span><span class="n">gutter</span><span class="p">);</span>
    <span class="p">}</span>
    
    <span class="k">@media</span> <span class="o">(</span><span class="nt">min-width</span><span class="o">:</span> <span class="nt">600px</span><span class="o">)</span> <span class="p">{</span>
      <span class="nd">:root</span> <span class="p">{</span>
        <span class="o">--</span><span class="n">gutter</span><span class="o">:</span> <span class="m">16px</span><span class="p">;</span>
      <span class="p">}</span>
    <span class="p">}</span>

It’s important to call out that the above snippet of code is not possible
using today’s CSS preprocessors which are unable to define variables inside of 
media queries. Having this ability unlocks a lot of potential!

It’s also possible to have custom properties that derive their value from
other custom properties. This can be extremely useful for theming:

    <span class="nd">:root</span> <span class="p">{</span>
      <span class="o">--</span><span class="n">primary</span><span class="o">-</span><span class="k">color</span><span class="o">:</span> <span class="nb">red</span><span class="p">;</span>
      <span class="o">--</span><span class="n">logo</span><span class="o">-</span><span class="k">text</span><span class="o">:</span> <span class="n">var</span><span class="p">(</span><span class="o">--</span><span class="n">primary</span><span class="o">-</span><span class="k">color</span><span class="p">);</span>
    <span class="p">}</span>

## The var() function {#the-var-function}

To retrieve and use the value of a custom property you’ll need to use the 
`var()` function. The syntax for the `var()` function looks like this:

    var(<custom-property-name> [, <declaration-value> ]? )
    

Where `<custom-property-name>` is the name of an author defined custom
property, like`--foo`, and `<declaration-value>` is a fallback value to
be used when the referenced custom property is invalid. Fallback values can be a
comma separated list, which will be combined into a single value. For example
`var(--font-stack,
"Roboto", "Helvetica");` defines a fallback of `"Roboto", "Helvetica"`. Keep in
mind that shorthand values, like those used for margin and padding, are not 
comma separated, so an appropriate fallback for padding would look like this.

    <span class="nt">p</span> <span class="p">{</span>
      <span class="k">padding</span><span class="o">:</span> <span class="n">var</span><span class="p">(</span><span class="o">--</span><span class="n">pad</span><span class="o">,</span> <span class="m">10px</span> <span class="m">15px</span> <span class="m">20px</span><span class="p">);</span>
    <span class="p">}</span>

Using these fallback values, a component author can write defensive styles for
their element:

    <span class="c">/* In the component’s style: */</span>
    <span class="nc">.component</span> <span class="nc">.header</span> <span class="p">{</span>
      <span class="k">color</span><span class="o">:</span> <span class="n">var</span><span class="p">(</span><span class="o">--</span><span class="n">header</span><span class="o">-</span><span class="k">color</span><span class="o">,</span> <span class="nb">blue</span><span class="p">);</span>
    <span class="p">}</span>
    <span class="nc">.component</span> <span class="nc">.text</span> <span class="p">{</span>
      <span class="k">color</span><span class="o">:</span> <span class="n">var</span><span class="p">(</span><span class="o">--</span><span class="k">text</span><span class="o">-</span><span class="k">color</span><span class="o">,</span> <span class="nb">black</span><span class="p">);</span>
    <span class="p">}</span>
    
    <span class="c">/* In the larger application’s style: */</span>
    <span class="nc">.component</span> <span class="p">{</span>
      <span class="o">--</span><span class="k">text</span><span class="o">-</span><span class="k">color</span><span class="o">:</span> <span class="m">#080</span><span class="p">;</span>
      <span class="c">/* header-color isn’t set,</span>
    <span class="c">     and so remains blue,</span>
    <span class="c">     the fallback value */</span>
    <span class="p">}</span>

This technique is especially useful for theming Web Components that use Shadow
DOM, as custom properties can traverse shadow boundaries. A Web Component author
can create an initial design using fallback values, and expose theming “hooks” 
in the form of custom properties.

    <span class="c"><!-- In the web component's definition: --></span>
    <span class="nt"><x-foo></span>
      #shadow
        <span class="nt"><style></span>
          <span class="nt">p</span> <span class="p">{</span>
            <span class="k">background-color</span><span class="o">:</span> <span class="n">var</span><span class="p">(</span><span class="o">--</span><span class="k">text</span><span class="o">-</span><span class="k">background</span><span class="o">,</span> <span class="nb">blue</span><span class="p">);</span>
          <span class="p">}</span>
        <span class="nt"></style></span>
        <span class="nt"><p></span>
          This text has a yellow background because the document styled me! Otherwise it
          would be blue.
        <span class="nt"></p></span>
    <span class="nt"></x-foo></span>

    <span class="c">/* In the larger application's style: */</span>
    <span class="nt">x-foo</span> <span class="p">{</span>
      <span class="o">--</span><span class="k">text</span><span class="o">-</span><span class="k">background</span><span class="o">:</span> <span class="nb">yellow</span><span class="p">;</span>
    <span class="p">}</span>

When using `var()` there are a few gotchas to watch out for. Variables cannot
be property names. For instance:

    <span class="nc">.foo</span> <span class="p">{</span>
      <span class="o">--</span><span class="n">side</span><span class="o">:</span> <span class="k">margin-top</span><span class="p">;</span>
      <span class="n">var</span><span class="p">(</span><span class="o">--</span><span class="n">side</span><span class="p">)</span><span class="o">:</span> <span class="m">20px</span><span class="p">;</span>
    <span class="p">}</span>

However this is not equivalent to setting `margin-top: 20px;`. Instead, the
second declaration is invalid and is thrown out as an error.

Similarly, you can’t (naively) build up a value where part of it is provided
by a variable:

    <span class="nc">.foo</span> <span class="p">{</span>
      <span class="o">--</span><span class="n">gap</span><span class="o">:</span> <span class="m">20</span><span class="p">;</span>
      <span class="k">margin-top</span><span class="o">:</span> <span class="n">var</span><span class="p">(</span><span class="o">--</span><span class="n">gap</span><span class="p">)</span><span class="k">px</span><span class="p">;</span>
    <span class="p">}</span>

Again, this is not equivalent to setting `margin-top: 20px;`. To build up a
value, you need something else: the`calc()` function.

## Building values with calc() {#building-values-with-calc}

If you’ve never worked with it before, the `calc()` function is a handly little
tool that lets you perform calculations to determine CSS values. It’s
[supported on all modern browsers][2], and can be combined with custom
properties to build up new values. For example:

    <span class="nc">.foo</span> <span class="p">{</span>
      <span class="o">--</span><span class="n">gap</span><span class="o">:</span> <span class="m">20</span><span class="p">;</span>
      <span class="k">margin-top</span><span class="o">:</span> <span class="n">calc</span><span class="p">(</span><span class="n">var</span><span class="p">(</span><span class="o">--</span><span class="n">gap</span><span class="p">)</span> <span class="o">*</span> <span class="m">1px</span><span class="p">);</span> <span class="c">/* niiiiice */</span>
    <span class="p">}</span>

## Working with custom properties in JavaScript {#working-with-custom-
properties-in-javascript
}

To get the value of a custom property at runtime, use the `getPropertyValue()`
method of the computed CSSStyleDeclaration object.

    <span class="c">/* CSS */</span>
    <span class="nd">:root</span> <span class="p">{</span>
      <span class="o">--</span><span class="n">primary</span><span class="o">-</span><span class="k">color</span><span class="o">:</span> <span class="nb">red</span><span class="p">;</span>
    <span class="p">}</span>
    
    <span class="nt">p</span> <span class="p">{</span>
      <span class="k">color</span><span class="o">:</span> <span class="n">var</span><span class="p">(</span><span class="o">--</span><span class="n">primary</span><span class="o">-</span><span class="k">color</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="c"><!-- HTML --></span>
    <span class="nt"><p></span>I’m a red paragraph!<span class="nt"></p></span>

    <span class="cm">/* JS */</span>
    <span class="kd">var</span> <span class="nx">styles</span> <span class="o">=</span> <span class="nx">getComputedStyle</span><span class="p">(</span><span class="nb">document</span><span class="p">.</span><span class="nx">documentElement</span><span class="p">);</span>
    <span class="kd">var</span> <span class="nx">value</span> <span class="o">=</span> <span class="nb">String</span><span class="p">(</span><span class="nx">styles</span><span class="p">.</span><span class="nx">getPropertyValue</span><span class="p">(</span><span class="s1">'--primary-color'</span><span class="p">)).</span><span class="nx">trim</span><span class="p">();</span>
    <span class="c1">// value = 'red'</span>

Similarly, to set the value of custom property at runtime, use the 
`setProperty()` method of the `CSSStyleDeclaration` object.

    <span class="c">/* CSS */</span>
    <span class="nd">:root</span> <span class="p">{</span>
      <span class="o">--</span><span class="n">primary</span><span class="o">-</span><span class="k">color</span><span class="o">:</span> <span class="nb">red</span><span class="p">;</span>
    <span class="p">}</span>
    
    <span class="nt">p</span> <span class="p">{</span>
      <span class="k">color</span><span class="o">:</span> <span class="n">var</span><span class="p">(</span><span class="o">--</span><span class="n">primary</span><span class="o">-</span><span class="k">color</span><span class="p">);</span>
    <span class="p">}</span>

    <span class="c"><!-- HTML --></span>
    <span class="nt"><p></span>Now I’m a green paragraph!<span class="nt"></p></span>

    <span class="cm">/* JS */</span>
    <span class="nb">document</span><span class="p">.</span><span class="nx">documentElement</span><span class="p">.</span><span class="nx">style</span><span class="p">.</span><span class="nx">setProperty</span><span class="p">(</span><span class="s1">'--primary-color'</span><span class="p">,</span> <span class="s1">'green'</span><span class="p">);</span>

You can also set the value of the custom property to refer to another custom
property at runtime by using the`var()` function in your call to 
`setProperty()`.

    <span class="c">/* CSS */</span>
    <span class="nd">:root</span> <span class="p">{</span>
      <span class="o">--</span><span class="n">primary</span><span class="o">-</span><span class="k">color</span><span class="o">:</span> <span class="nb">red</span><span class="p">;</span>
      <span class="o">--</span><span class="n">secondary</span><span class="o">-</span><span class="k">color</span><span class="o">:</span> <span class="nb">blue</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="c"><!-- HTML --></span>
    <span class="nt"><p></span>Sweet! I’m a blue paragraph!<span class="nt"></p></span>

    <span class="cm">/* JS */</span>
    <span class="nb">document</span><span class="p">.</span><span class="nx">documentElement</span><span class="p">.</span><span class="nx">style</span><span class="p">.</span><span class="nx">setProperty</span><span class="p">(</span><span class="s1">'--primary-color'</span><span class="p">,</span> <span class="s1">'var(--secondary-color)'</span><span class="p">);</span>

Because custom properties can refer to other custom properties in your
stylesheets, you could imagine how this could lead to all sorts of interesting 
runtime effects.

## Browser Support {#browser-support}

Currently Chrome 49, Firefox 42, Safari 9.1, and iOS Safari 9.3 support custom
properties.

## Demo {#demo}

Try out the [sample][3] for a glimpse at all of the interesting techniques you
can now leverage thanks to custom properties.

## Further Reading {#further-reading}

If you’re interested to learn more about custom properties, Philip Walton
from the Google Analytics team has written a primer on
[why he’s excited for custom properties][4] and you can keep tab on their
progress in other browsers over on[chromestatus.com][5].

Except as otherwise noted, the content of this page is licensed under the 
[Creative Commons Attribution 3.0 License][6], and code samples are licensed
under the[Apache 2.0 License][7]. For details, see our [Terms of Service][8].

 [1]: http://www.xanthir.com/blog/b4KT0
 [2]: http://caniuse.com/#search=calc
 [3]: https://googlechrome.github.io/samples/css-custom-properties/index.html

 [4]: http://philipwalton.com/articles/why-im-excited-about-native-css-variables/
 [5]: https://www.chromestatus.com/features/6401356696911872
 [6]: http://creativecommons.org/licenses/by/3.0/
 [7]: http://www.apache.org/licenses/LICENSE-2.0
 [8]: https://developers.google.com/site-terms
