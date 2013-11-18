---
layout: post
title: Implementing Hot Tracking in an ActiveX control
permalink: /blog/implementing-hot-tracking-in-an-activex-control
date: 2009-08-19 07:57:00
modified: 2009-08-25 23:43:02
author: Admin
id: 1e3eea8f-af6b-44f2-af32-8d23bf9b3195
---

### Introduction
Initially, this article was intended as a tutorial for the creation of ActiveX
controls. However, this subject is well covered by articles in magazines and on the
Internet. So, instead, I have decided to concentrate on one aspect of control
creation, which can make your control look and interact with the user better.

### What is Hot Tracking?

For the purposes of this article, hot tracking will be the process when a UI element
reacts to the cursor, entering or leaving its extent, by changing it appearance. Hot
tracking is not a new concept; it has been used in computer games for years. For
example, a game I made five years ago for Windows 3.1 had support for hot tracking
built-in as a standard feature. Recently, with help from Internet Explorer, MS Office
and other software, hot tracking is becoming a standard in user interfaces. In fact,
many UI and non-UI concepts made their first appearance in computer games, but this
is a separate discussion. Considering the abovementioned, supporting hot tracking in
your control may sound like a good idea. So, without further delay, let's get to the
point.

### Implementing Hot Tracking

We will start with a simple implementation of hot tracking. To implement simple hot
tracking, we will need to implement only two functions, OnMouseEnter and OnMouseLeave,
like this:

    void OnMouseEnter()
    {
        SetState(HOT);
        UpdateControl();
    }

    void OnMouseLeave()
    {
        SetState(NORMAL);
        UpdateControl();
    }

    HRESULT Draw(/*Parameters*/)
    {
        if (GetState() == HOT)
        {
            DrawHot(/*Parameters*/);
        }
        else if (GetState() == NORMAL)
        {
            DrawNormal(/*Parameters*/);
        }
    }

Now, you can see that every time the cursor enters the control area, the control will
change its appearance to indicate it's hot (active, selected, highlighted) state.
When cursor leaves the control, it will go back to its normal state. As we saw above,
basic support for hot tracking is quite simple. Of course, if you are an experienced
ATL developer, you would say, "But ATL doesn't support functions OnMouseEnter and
OnMouseLeave." Yes, that's true; unlike Java or DHTML, ATL leaves all of the
mouse-processing up to the programmer by providing only the WM_MOUSEMOVE message.
However, in ATL's defense, I shall say it inherited this shortcoming from Windows.

> Java and DHTML each have full support for mouse tracking. 
>
> * Java implements: mouseEnter, mouseExit, mouseMove and mouseDrag.
> * DHTML supports events: onmouseover, onmousemove, onmouseout.

To remedy the above problem, we should detect when the cursor enters and leaves our
control and call OnMouseEnter and OnMouseLeave accordingly. Seems simple, I know,
but where is the catch to all of this? The catch is this: because we only get
WM_MOUSEMOVE, we know when the cursor moves inside of our control, but once the
cursor leaves the control, we don't get WM_MOUSEMOVE anymore, and as result, we
cannot detect when the cursor left our control! There are four solutions known to me
to combat this problem, and I will discuss them in the sections below.

### Timer method

The first method that comes to mind is pretty simple. When the first mouse move is
detected, we should create a timer that will send us WM_TIMER message a few times
per second. Then, for every WM_TIMER message we receive, we should check if the
cursor is still inside of our control. If it is, then everything is OK. If not, we
should stop the timer and call OnMouseLeave. This method is pretty simple and
reliable, but it has some drawbacks. One -- it uses a timer. Two -- there's a delay
in detecting when the cursor leaves the control, up to the timer's interval, which
results in an unpleasant eye-lagging of the controls update. The Outlook Bar in 
MS Outlook uses this method. You can play around with it to see this lagging effect.

### TrackMouseEvent method

Windows 98 supports a function called TrackMouseEvent and two new mouse messages,
WM_MOUSEHOVER and WM_MOUSELEAVE. With the help of this new function, mouse tracking
becomes easy. Just call TrackMouseEvent with the window handle and flags set to
indicate that you want to receive WM_MOUSELEAVE for the specified window. Too bad,
however; this method has even more limitations then the previous one. First, it
won't work on Windows 95. Second, it requires a window and won't work with
windowless controls.

### Global Hook method

Setting a system-wide hook and monitoring of all mouse messages is the only
completely reliable way to detect when the mouse leaves a control. However, it is
hard to implement and it is performance-unfriendly.

### Mouse capture method

I am mentioning this method last, because this is my preferred method to deal with
the mouse-tracking problem. In fact, this method is almost the same as the OnTimer
method, but instead of using SetTimer/KillTimer it uses SetCapture/ReleaseCapture.
The advantages of this method are the following: it doesn't take up the timer
resource, it doesn't show a lagging effect, and it works with windowless controls.
This method is not ideal, but it works well in most cases. You can use any of the
methods I described depending on your situation. Below is a sample implementation
of OnMouseMove using SetCapture method.

    LRESULT OnMouseMove(UINT, WPARAM wParam, LPARAM lParam, BOOL& bHandled)
    {
        POINT pt;
        pt.x = LOWORD(lParam);
        pt.y = HIWORD(lParam);
        
        bMouseCapture = GetCapture(); //* see notes
        RECT rc = GetClientRect();
        if (!bMouseCapture && ::PtInRect(&rc, pt))
        {
            SetCapture(true);
            bMouseCapture = true;
            OnMouseEnter();
        }

        if (bMouseCapture && !::PtInRect(&rc, pt))
        {
            SetCapture(false);
            bMouseCapture = false;
            OnMouseLeave();
        }
    }

There are some considerations to be made regarding the above code sample. The mouse
is a shared system resource and its capture is subject to some regulations. The
system or another application can take capture away from you at any moment.
A windowed control should receive WM_CAPTURECHANGED when it loses capture. 
A windowless control depends on its container to pass the message to it.
Well-behaving containers will send a WM_CAPTURECHANGED to notify the control when
it loses mouse capture (IE does this). Not-so-well-behaving containers don't do
anything (that includes ATL Composite Control). If you are creating a windowless
control without prior knowledge of its host, or you want to be as safe as possible,
you should test to see if you still have capture every time you receive OnMouseMove.
It is still a good idea to process WM_CAPTURECHANGED, though.

### Wrapping it up

Now that we are done with the mouse tracking issue, let's take a look at the 
implementation of SetCapture and GetCapture functions:

    bool GetCapture()
    {
        if (m_hWnd != NULL)
            return ::GetCapture() == m_hWnd;
        if (m_spInPlaceSite != NULL)
            return m_spInPlaceSite->GetCapture() == S_OK;
        return false;
    }

    void SetCapture(bool bCapture)
    {
        if (m_hWnd != NULL)
        {
            if (bCapture)
                ::SetCapture(m_hWnd);
            else
                ::ReleaseCapture();
        }
        else if (m_spInPlaceSite != NULL)
            m_spInPlaceSite->SetCapture(bCapture);
    }

Acquiring mouse capture is done differently for windowed and windowless controls.
Functions shown above encapsulate these differences to simplify the development of
controls capable of working in both modes.

If you want to create control with some parts of it supporting hot tracking, you 
should do all that's been said above, plus, in your OnMouseMove code, you should 
check which part of the control is hot and change its state accordingly. I leave 
this part for you to figure out.

Now you know how to create you own control with support for hot tracking. Time to
make them look good!

February 21, 1999
