---
layout: post
title: Fun with TipBubble. Styling TabControl
permalink: /blog/fun-with-tipbubble
category: Blog
date: 2010-07-22 13:55:09
modified: 2010-08-07 07:31:39
author: admin
id: ab17324d-96ab-46f0-9f50-3dc24c12bda3
---

I was thinking of some fun ways to use the TipBubble control, aside from the obvious. Here’s an idea, how about
using TipBubble to style TabControl?

<img alt="TabStyle" src="/i/2010-7-22-fun-with-tipbubble/tabstyle.jpg" width="287" height="55" />

Achieving the result you see on the image above is quite simple. Just make a copy of the default TabItem style
and replace the TemplateTopSelected grid with the code below.

{% highlight xml %}
    <Grid x:Name="TemplateTopSelected" Canvas.ZIndex="1" Visibility="Collapsed">
        <tip:TipBubble
            BorderBrush="{TemplateBinding BorderBrush}"
            BorderThickness="1,1,1,0"
            CornerRadius="3,3,0,0"
            TipPosition="0.5"
            TipSide="Bottom"
            TipFill="#FcD400"
            TipSize="10"
            Margin="0 0 0 -8">
            <tip:TipBubble.Background>
                <LinearGradientBrush
                    EndPoint="0,0" StartPoint="0,1"
                    ColorInterpolationMode="ScRgbLinearInterpolation">
                    <GradientStopCollection>
                        <GradientStop Color="#FFE97F" Offset="1" />
                        <GradientStop Color="#FcD400" Offset="0" />
                    </GradientStopCollection>
                </LinearGradientBrush>
            </tip:TipBubble.Background>
            <ContentControl
                Cursor="{TemplateBinding Cursor}"
                Height="Auto" Width="Auto"
                x:Name="HeaderTopSelected"
                FontSize="{TemplateBinding FontSize}"
                FontWeight="Bold"
                Foreground="#333" IsTabStop="False"
                HorizontalContentAlignment="Center"
                VerticalContentAlignment="Center"
                Margin="6,4,6,4"/>
        </tip:TipBubble>
    </Grid>
{% endhighlight %}

Repeat this operation for TemplateBottomSelected, TemplateLeftSelected, and TemplateRightSelected to get
the complete style. Remember to modify TipSide, Margin BorderThickness and CornerRadius to correspond with
each template (i.e. tab on the bottom, tip on the top; tab on the left, tip on the right etc.)
