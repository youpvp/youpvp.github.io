---
layout: post
title: How to extend CAxHostWindow to support new functionality
permalink: /blog/how-to-extend-caxhostwindow-to-support-new-functionality
date: 2009-08-19 11:36:00
modified: 2009-08-25 23:39:13
author: Admin
id: 90f7385e-1f35-4170-b02d-64e25bae2b98
---

There are two major methods that you can use to support control containment in your
control using ATL 3.0. One method is to use the support provided by ATL, namely
CAxHostWindow. The other method is to write everything from scratch. By far, using
ATL is easier then writing tons of code yourself, but what if CAxHostWindow doesn't
support the functionality that you need? Is following the second method your only 
option? Most likely, the answer is no. In most cases, the result can be achieved 
just by extending the functionality of CAxHostWindow, which I will be discussing 
as the main topic in this article.

The most obvious way to extend the functionality of CAxHostWindow is to modify its
code in atlhost.h, but it should be done with care because changes you make will 
reflect on every user of CAxHostWindow, such as Composite Control. An alternative 
solution is to derive a new class from CAxHostWindow and override its methods and 
message handlers. Below, I will demonstrate in more detail how to modify 
CAxHostWindow to support new functionality.

The first thing you should do after deriving from CAxHostWindow is to declare class
factory, since, without doing so, CreateInstance will always create CAxHostWindow
which has a class factory declared. You don&rsquo;t have to use the same class
factory as CAxHostWindow. In the code below, I declared my class as not aggregatable,
even if CAxHostWindow uses DECLARE_POLY_AGGREGATABLE.

    class CAxHostWindowEx : public CAxHostWindow
    {
    public:
        DECLARE_NOT_AGGREGATABLE(CAxHostWindowEx);

        CAxHostWindowEx()
        {
            m_bTrack = false;
        }
        ...
    }

You may already know that CAxHostWindow is responsible for forwarding window
messages to windowless controls. Because messages sent to windows is the primary
method of communication in the Windows environment, it is important to know which
messages are forwarded to hosted control. In my control, I wanted to forward the
following messages: WM_CAPTURECHANGED, WM_MOUSEWHEEL and WM_MOUSELEAVE. In the code
fragment below, you can see how forwarding works. If you need to forward a message,
you should map it to OnWindowlessMouseMessage if it is a mouse message or to
OnWindowMessage for the rest.

    BEGIN_MSG_MAP(CAxHostWindowEx)
        if (m_bWindowless)
        {
            MESSAGE_HANDLER(WM_MOUSEMOVE, OnMouseMove)
            MESSAGE_HANDLER(WM_MOUSELEAVE, OnMouseLeave)
            MESSAGE_HANDLER(WM_CAPTURECHANGED, OnWindowMessage)

            // Mouse messages handled when a windowless control has captured 
            // the cursor or if the cursor is over the control
            
            DWORD dwHitResult = m_bCapture ? HITRESULT_HIT : HITRESULT_OUTSIDE;
            if (dwHitResult == HITRESULT_OUTSIDE && m_spViewObject != NULL)
            {
                POINT ptMouse = { LOWORD(lParam), HIWORD(lParam) };
                m_spViewObject->QueryHitPoint(DVASPECT_CONTENT, 
                               &m_rcPos, ptMouse, 0, &dwHitResult);
            }
            if (dwHitResult == HITRESULT_HIT)
            {
                MESSAGE_HANDLER(WM_MOUSEWHEEL, OnWindowlessMouseMessage)
            }
        }
        CHAIN_MSG_MAP(CAxHostWindow)
    END_MSG_MAP()

Overriding methods of CAxHostWindow to change its behavior are fairly straightforward.

    STDMETHOD(OnPosRectChange)(LPCRECT lprc)
    {
        SetWindowPos(NULL, lprc.left , lprc.top,
                     lprc.righ - lprc.left, lprc.bottom - lprc.top, 
                     SWP_NOACTIVATE | SWP_NOZORDER);
        return S_OK;
    }

In the example above, method OnPosRectChange, when called, will change the position
and size of its host window. CAxHostWindow leaves many methods unimplemented or with
default implementation. Here are some of the methods you might consider for
overriding:

* IOleClientSite 
    - SaveObject
    - GetMoniker
    - RequestNewObjectLayout
* IOleInPlaceSite 
    - OnPosRectChange
* IOleInPlaceSiteWindowless 
    - GetFocus
    - SetFocus
* IOleControlSite 
    - TransformCoords
    - TranslateAccelerator
    - OnFocus
    - ShowPropertyFrame

After implementing all modifications in our derived class, the next question is
this: how are we going to use it? Normally CAxHostWindow stays invisible, and never
needs to be used directly. In the diagram below, you can see the relations between 
different components of ATL and CAxHostWindow.

<img src="http://youpvp.com/blog/image.axd?picture=scheme.png" alt="" />

From the diagram above, it becomes clear that by creating alternative to
CAxHostWindow, we've made only the first step. To make it usable, alternatives
to AtlAxCreateControl, AtlAxWin, CAxWindow, and Composite Control should be created.
However, creating an alternative AtlAxCreateControl and AtlAxAttachControl should
satisfy the bare minimum.

By applying the techniques described above, you can enjoy the conveniences of using
ATL's support for control containment and the freedom to do what you want.

In conclusion, I must say that the methods provided above will not be useful in all
situations. One serious limitation of CAxHostWindow, the inability to host more then
one control, cannot be changed. If you need to host multiple windowless controls, you
should consider a different solution.

June 6, 1999
