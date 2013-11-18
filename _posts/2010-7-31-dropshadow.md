---
layout: post
title: What you need to know about DropShadow to create great Silverlight applications
permalink: /blog/what-you-need-to-know-about-dropshadow
category: blog
date: 2010-07-31 14:42:39
modified: 2010-07-31 14:42:39
author: admin
id: ddebbb89-3ebb-4fee-b730-5db3a614a12d
---

Blur and DropShadow effects offer easy way to enhance your application’s visual appeal. But they come
with a big price tag, and if used incorrectly, could destroy user experience instead of improving it.
In order to make the best use of these effects in Silverlight, it is important to understand how they
work behind the curtain.

When Silverlight runtime renders an element with an effect (whether built-in or custom pixel shader),
it first renders that element and all of its children into an off-screen bitmap. Then, using that bitmap
as the input for the pixel shader effect, the output of the pixel shader is finally rendered to the screen.

What’s interesting about the Blur effect is that unlike custom effects that can only have one pass,
it executes in two passes. The first pass blurs the image horizontally, while second pass finishes with
a vertical blur. This is called separable blur, and it saves a significant amount of computation in exchange
for using extra memory and fill rate.

<img alt="blur breakdown" src="/i/2010-7-31-dropshadow/blur.jpg" width="420" />

The cost of separable blur with radius N (N tap filter) can be estimated as:
2 writes + 2 * N reads + 2 * N multiplies + 2 * (N – 1) additions per pixel. So, applying 9 tap blur effect
to a 400 x 400 pixel element will take 3,200,000 memory operations and 4,800,000 arithmetic operations. Now
you can see how applying the blur effect to a large element can choke even a high-end CPU. And yes, Silverlight 4
executes all effects, including blur and drop shadow, on the CPU. The Windows Phone 7 version of Silverlight is
an exception, because it doesn’t support custom pixel shaders, and executes Blur and Drop Shadow effects on the phone’s GPU.

The DropShadow effect is a little more complicated then the Blur effect. It starts in the same way as any other
effect by rendering the element into a memory bitmap. Then an alpha mask is separated from the bitmap and blurred
using the exact same two-pass method I described for blur. Finally, the bitmap and the blurred alpha mask combined
to create an element with drop shadow.

<img alt="shadow breakdown" src="/i/2010-7-31-dropshadow/shadow.jpg" width="583" />

Because DropShadow uses a blur filter, it will suffer the same performance problems as the Blur effect. Furthermore,
DropShadow uses even more memory and more fill rate, making it very taxing on both the CPU and memory bandwidth.
It is important to understand that the cost of applying the DropShadow effect depends entirely on the size of
the element, regardless of how much or how little of the actual shadow can be seen.

As I mentioned before, applying an effect to an element with children will cause the complete visual sub-tree
to be rendered into the bitmap and processed by pixel shader. This has an important implication – the easiest
way to drive CPU utilization to 100% is to add a drop shadow to the large form with one tiny animated control.

This last bit of information about effects is not performance related, but still important to know. Applying
an effect to a text element or its parent forces Silverlight to use its standard anti-aliased text rendering
instead of ClearType. This is the second reason why you should not add effects to the top level controls.

### Things you can do to improve performance

The recommendations I provide below hold true for all effects, but they are especially important for computationaly
expensive effects such as DropShadow.

 1. Use effects sparingly. This will not only improve your application’s performance, but it will also
    increase the impact from using them.
 2. Avoid adding drop shadow to elements with children, especially if children are interactive or animated.
    In most cases, it should be possible to redesign the visual hierarchy in such a way that drop shadow is
    applied to an element without children.
 3. Use alternative methods to achieve similar visual result. For example, drop shadow can be often
    approximated with the help of linear gradients. Pre-rendering the effect for static elements can
    offer big performance savings as well.
 4. If you need to animate large element with a drop shadow, turn effect off before starting animation
    and turn it back on at the end. You will be surprised how much smoother animation will feel.
 5. Consider creating a “fast” theme for your application with all nonessential effects removed. Netbook
    owners and mobile users will love you for this.
