---
layout: post
title: Dynamic resizing of ActiveX controls hosted by ATL Composite Control
permalink: /blog/dynamic-resizing-of-activex-controls
date: 2009-08-19 12:10:00
modified: 2009-08-26 15:02:04
author: Admin
id: 9b7fa69a-c9d9-47c5-b3d7-d01c3c9feb2f
---

Did you cheer when you found out that ATL 3.0 would support ActiveX control hosting?
I did. This functionality was at the top of my wish list for ATL. ATL 3.0 has finally
arrived. What now? In one news group, I saw the following question: "How do I resize
ActiveX controls hosted by Composite Control?" The answer to this question is very
simple, but the reasoning behind it is not. Therefore, I decided to write this
article to share my knowledge of the inner workings of ATL's hosting of
ActiveX controls.

### How does ATL implement the hosting of ActiveX controls

If you will look in ATL code, you will notice that code for CComCompositeControl
takes less then two hundred lines. How is it possible to fit all hosting support
into two hundred lines of code? Well, the answer is quite simple -- 
CComCompositeControl does not have support for ActiveX hosting. The real hero behind
the scenes is CAxHostWindow.

CAxHostWindow implements necessary interfaces in order to support the hosting of
common ActiveX controls (including windowless) and Microsoft Web Browser control.
In case you do not need to host a Web Browser control, you can define
_ATL_NO_DOCHOSTUIHANDLER in your project and save about 4K in release dll.
CAxHostWindow derives from CWindowImpl, and this means it needs a window to operate.
CAxHostWindow can host only one control at a time, and as a result, one CAxHostWindow
is required for each hosted control. Without going into all the aspects of
implementation (look into atlhost.h for more details), I would like to concentrate
here on the way CAxHostWindow handles WM_SIZE and WM_PAINT.

When CAxHostWindow receives WM_SIZE, it checks if the hosted control supports
IOleObject and sets its new extent. In addition, if the hosted control is a
windowless control, then SetObjectRects() will be called with the new control size
equal to its window size. An important observation is that a control hosted by
CAxHostWindow always occupies the full size of the window. When handling WM_PAINT,
CAxHostWindow exhibits interesting behavior only for windowless objects. If it is
drawing a windowless control, it creates an off screen device context, paints it
with background color (which is by default COLOR_BTNFACE, if parent window is a
dialog, or otherwise COLOR_WINDOW), and calls Draw method of IViewObject. After
that, CAxHostWindow does BitBlt from off screen context to window context in order 
to prevent flickering while the control draws itself. Some controls employ a similar
flicker prevention technique inside their drawing code, which results in a slower
drawing of the control. Consequently, if you are creating a control that you will
only use with Composite Control, you can omit the flicker prevention code.

You do not need to create CAxHostWindow directly. ATL implements a helper function
AtlAxCreateControl(), which will create an instance of CAxHostWindow and will make
all necessary initializations for you. AtlAxCreateControl() has the following four
parameters: (1) the name of the control to be hosted; (2) the window handle to be
used by CAxHostWindow; (3) IStream, which can be used to initialize control if it
supports IPersistStream; and (4) pointer to IUnknown, for IUnknown of created
CAxHostWindow.

As you can see below, any application can support the hosting of ActiveX controls
with ease, using support provided by ATL.

    #include "stdafx.h"
    #include "resource.h"

    CComModule _Module;

    BEGIN_OBJECT_MAP(ObjectMap)
    END_OBJECT_MAP()

    int APIENTRY 
    WinMain(HINSTANCE hInst, HINSTANCE hPrevInst, LPSTR lpCmdLine, int nCmdShow)
    {
        _Module.Init(ObjectMap, hInst);

        //Register window class here

        HWND hWnd = CreateWindow( TEXT("ClassName"), TEXT("Title"),
                    WS_OVERLAPPEDWINDOW, CW_USEDEFAULT, CW_USEDEFAULT,
                    300, 300, 0L, 0L, hInst, 0L );

        USES_CONVERSION;
        CComPtr spUnk;
        HRESULT hRet = AtlAxCreateControl(T2COLE("Some.Control.1"), hWnd, NULL, &spUnk);
        if(FAILED(hRet))
            return -1;

        //Show window and go to GetMessage loop

        _Module.Terminate();
        return msg.wParam;
    }
	
### Composite Control

Now that we know how hosting works in ATL, we can speculate on how Composite Control
works. One possible scenario could go like this: When Composite Control is created,
it parses its dialog template; for every ActiveX control description it finds, it
creates a child window and then passes it and the control&rsquo;s name to
AtlAxCreateControl(). However, the developers of ATL found a simpler and more 
elegant solution. In order to understand their solution, we should first look at
the helper window class provided by ATL. ATL implements a function AtlAxWinInit(),
which registers window class "AtlAxWin". When the window of this class is created,
the handler for WM_CREATE uses the window&rsquo;s caption as the control's name and
creates the control hosted in this window. It also converts CreateParams into IStream
and passes it to the control. The handler for WM_DESTROY releases the host control 
interface to guarantee its destruction. Now, we can create the control hosted in the
window with even less code.

    AtlAxWinInit();
    CreateWindow( TEXT("AtlAxWin"), TEXT("Some.control.1"),
        WS_CHILD | WS_VISIBLE, x, y, cx, cy, hWndParent, NULL, hInst, NULL);
		
Now, we're ready to investigate how Composite Control creates hosted controls.
Composite Control indirectly derives from CAxDialgImpl. When CaxDialogImpl is
created, it calls the helper function AtlAxCreateDialog(). This function initializes
"AtlAxWin" by calling AtlAxWinInit() and then makes a call to a function mysteriously
named _DialogSplitHelper::SplitDialogTemplate(). What this particular function does
is essentially parse through the dialog template and finds all CONTROL entries with
class names starting with "{". When Visual Studio saves the dialog resource
containing ActiveX controls, it saves information about each control in a form that
look like a windows control, but instead of a window class, it writes the CLSID of
the ActiveX control.

    IDD_DIALOG DIALOGEX 0, 0, 286, 177
    STYLE WS_CHILD
    EXSTYLE WS_EX_CONTROLPARENT
    FONT 8, "MS Sans Serif"
    BEGIN
        CONTROL    "",IDC_CONTROL,"{3343C18D-A997-11D2-B348-00A0244AE119}",
                   WS_TABSTOP,3,3,133,171
        CONTROL    "Progress1",IDC_PROGRESS1,"msctls_progress32",WS_BORDER,
                   87,95,80,14
    END

For each ActiveX control found, SplitDialogTemplate() replaces it with record that
looks like the following below:

    CONTROL     "{3343C18D-A997-11D2-B348-00A0244AE119}",IDC_CONTROL,"AtlAxWin",
                WS_BORDER, 3,3,133,171
            
As you can see this record look exactly like the record of a normal windows control,
and it can be created by a standard windows function like
::CreateDialogIndirectParam() or ::DialogBoxIndirectParam().

### Summary

Finally, we get to the bottom of our investigation, and I can review it in short. 
Composite Control does not have any hosting capabilities, and relies completely on
support provided by CAxDialogImpl. CAxDialogImpl uses the ability of "AtlAxWin" to 
make ActiveX controls look like windows controls. CAxHostWindow provides support for
the hosting of ActiveX controls, but it requires a window to do so.

Now, we can go back to the question, "How do I resize ActiveX controls hosted by
Composite Control?" We know now that the answer is simple - "In the same way
you would resize any windows control in dialog, because Composite Control doesn't
know the difference!" Let's illustrate it with the following code sample:

    LRESULT OnSize(UINT, WPARAM wParam, LPARAM lParam, BOOL& bHandled)
    {
        int cx = LOWORD(lParam);
        int cy = HIWORD(lParam);
        
        HWND h = GetDlgItem(IDC_CONTROL);
        ::SetWindowPos(h, NULL, 3, 3, cx - 6, cy - 27, SWP_NOZORDER);

        h = GetDlgItem(IDC_PROGRESS1);
        ::SetWindowPos(h, NULL, 3, cy - 21, cx - 6, 18, SWP_NOZORDER);
    }

The implementation of hosting for ActiveX controls in ATL is simple and lightweight,
but it obviously has limitations. All advantages of windowless controls are
effectively eliminated by one control per window restriction. Methods
OnPosRectChange() and RequestNewObjectLayout() are not implemented. This means you
cannot implement resizing per the control&rsquo;s request.

ATL's "AtlAxWin" shows us a very interesting possibility. We can create a window 
class that will wrap around some specific ActiveX control, include mapping controls
methods into messages, and make this control accessible for non-COM environments!
The creation process of a wrapper window class is even possible to automate. This 
is a very interesting opportunity, indeed!

February 1, 1999
