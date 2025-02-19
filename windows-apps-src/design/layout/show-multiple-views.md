---
Description: View multiple parts of your app in separate windows.
title: Show multiple views for an app
ms.date: 05/19/2017
ms.topic: article
keywords: windows 10, uwp
ms.localizationpriority: medium
---
# Show multiple views for an app

![Wireframe showing an app with multiple windows](images/multi-view.gif)

Help your users be more productive by letting them view independent parts of your app in separate windows. When you create multiple windows for an app, each window behaves independently. The taskbar shows each window separately. Users can move, resize, show, and hide app windows independently and can switch between app windows as if they were separate apps. Each window operates in its own thread.

> **Important APIs**: [**ApplicationViewSwitcher**](https://docs.microsoft.com/uwp/api/Windows.UI.ViewManagement.ApplicationViewSwitcher), [**CreateNewView**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.core.coreapplication.createnewview)

## When should an app use multiple views?
There's a variety of scenarios that can benefit from multiple views. Here are a few examples:
 - An email app that lets users view a list of received messages while composing a new email
 - An address book app that lets users compare contact info for multiple people side-by-side
 - A music player app that lets users see what's playing while browsing through a list of other available music
 - A note-taking app that lets users copy information from one page of notes to another
 - A reading app that lets users open several articles for reading later, after an opportunity to peruse all high-level headlines

To create separate instances of your app, see [Create a multi-instance UWP app](../../launch-resume/multi-instance-uwp.md).

## What is a view?

An app view is the 1:1 pairing of a thread and a window that the app uses to display content. It's represented by a [**Windows.ApplicationModel.Core.CoreApplicationView**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.Core.CoreApplicationView) object.

Views are managed by the [**CoreApplication**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.Core.CoreApplication) object. You call [**CoreApplication.CreateNewView**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.core.coreapplication.createnewview) to create a [**CoreApplicationView**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.Core.CoreApplicationView) object. The **CoreApplicationView** brings together a [**CoreWindow**](https://docs.microsoft.com/uwp/api/Windows.UI.Core.CoreWindow) and a [**CoreDispatcher**](https://docs.microsoft.com/uwp/api/Windows.UI.Core.CoreDispatcher) (stored in the [**CoreWindow**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.core.coreapplicationview.corewindow) and [**Dispatcher**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.core.coreapplicationview.dispatcher) properties). You can think of the **CoreApplicationView** as the object that the Windows Runtime uses to interact with the core Windows system.

You typically don’t work directly with the [**CoreApplicationView**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.Core.CoreApplicationView). Instead, the Windows Runtime provides the [**ApplicationView**](https://docs.microsoft.com/uwp/api/Windows.UI.ViewManagement.ApplicationView) class in the [**Windows.UI.ViewManagement**](https://docs.microsoft.com/uwp/api/Windows.UI.ViewManagement) namespace. This class provides properties, methods, and events that you use when your app interacts with the windowing system. To work with an **ApplicationView**, call the static [**ApplicationView.GetForCurrentView**](https://docs.microsoft.com/uwp/api/windows.ui.viewmanagement.applicationview.getforcurrentview) method, which gets an **ApplicationView** instance tied to the current **CoreApplicationView**’s thread.

Likewise, the XAML framework wraps the [**CoreWindow**](https://docs.microsoft.com/uwp/api/Windows.UI.Core.CoreWindow) object in a [**Windows.UI.XAML.Window**](https://docs.microsoft.com/uwp/api/Windows.UI.Xaml.Window) object. In a XAML app, you typically interact with the **Window** object rather than working directly with the **CoreWindow**.

## Show a new view

While each app layout is unique, we recommend including a "new window" button in a predictable location, such as the top right corner of the content that can be opened in a new window. Also consider including a context menu option to "Open in a new window".

Let's look at the steps to create a new view. Here, the new view is launched in response to a button click.

```csharp
private async void Button_Click(object sender, RoutedEventArgs e)
{
    CoreApplicationView newView = CoreApplication.CreateNewView();
    int newViewId = 0;
    await newView.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
    {
        Frame frame = new Frame();
        frame.Navigate(typeof(SecondaryPage), null);   
        Window.Current.Content = frame;
        // You have to activate the window in order to show it later.
        Window.Current.Activate();

        newViewId = ApplicationView.GetForCurrentView().Id;
    });
    bool viewShown = await ApplicationViewSwitcher.TryShowAsStandaloneAsync(newViewId);
}
```

**To show a new view**

1.  Call [**CoreApplication.CreateNewView**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.core.coreapplication.createnewview) to create a new window and thread for the view content.

    ```csharp
    CoreApplicationView newView = CoreApplication.CreateNewView();
    ```

2.  Track the [**Id**](https://docs.microsoft.com/uwp/api/windows.ui.viewmanagement.applicationview.id) of the new view. You use this to show the view later.

    You might want to consider building some infrastructure into your app to help with tracking the views you create. See the `ViewLifetimeControl` class in the [MultipleViews sample](https://go.microsoft.com/fwlink/p/?LinkId=620574) for an example.

    ```csharp
    int newViewId = 0;
    ```

3.  On the new thread, populate the window.

    You use the [**CoreDispatcher.RunAsync**](https://docs.microsoft.com/uwp/api/windows.ui.core.coredispatcher.windows) method to schedule work on the UI thread for the new view. You use a [lambda expression](https://go.microsoft.com/fwlink/p/?LinkId=389615) to pass a function as an argument to the **RunAsync** method. The work you do in the lambda function happens on the new view's thread.

    In XAML, you typically add a [**Frame**](https://docs.microsoft.com/uwp/api/Windows.UI.Xaml.Controls.Frame) to the [**Window**](https://docs.microsoft.com/uwp/api/Windows.UI.Xaml.Window)'s [**Content**](https://docs.microsoft.com/uwp/api/windows.ui.xaml.window.content) property, then navigate the **Frame** to a XAML [**Page**](https://docs.microsoft.com/uwp/api/Windows.UI.Xaml.Controls.Page) where you've defined your app content. For more info, see [Peer-to-peer navigation between two pages](../basics/navigate-between-two-pages.md).

    After the new [**Window**](https://docs.microsoft.com/uwp/api/Windows.UI.Xaml.Window) is populated, you must call the **Window**'s [**Activate**](https://docs.microsoft.com/uwp/api/windows.ui.xaml.window.activate) method in order to show the **Window** later. This work happens on the new view's thread, so the new **Window** is activated.

    Finally, get the new view's [**Id**](https://docs.microsoft.com/uwp/api/windows.ui.viewmanagement.applicationview.id) that you use to show the view later. Again, this work is on the new view's thread, so [**ApplicationView.GetForCurrentView**](https://docs.microsoft.com/uwp/api/windows.ui.viewmanagement.applicationview.getforcurrentview) gets the **Id** of the new view.

    ```csharp
    await newView.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
    {
        Frame frame = new Frame();
        frame.Navigate(typeof(SecondaryPage), null);   
        Window.Current.Content = frame;
        // You have to activate the window in order to show it later.
        Window.Current.Activate();

        newViewId = ApplicationView.GetForCurrentView().Id;
    });
    ```

4.  Show the new view by calling [**ApplicationViewSwitcher.TryShowAsStandaloneAsync**](https://docs.microsoft.com/uwp/api/windows.ui.viewmanagement.applicationviewswitcher.tryshowasstandaloneasync).

    After you create a new view, you can show it in a new window by calling the [**ApplicationViewSwitcher.TryShowAsStandaloneAsync**](https://docs.microsoft.com/uwp/api/windows.ui.viewmanagement.applicationviewswitcher.tryshowasstandaloneasync) method. The *viewId* parameter for this method is an integer that uniquely identifies each of the views in your app. You retrieve the view [**Id**](https://docs.microsoft.com/uwp/api/windows.ui.viewmanagement.applicationview.id) by using either the **ApplicationView.Id** property or the [**ApplicationView.GetApplicationViewIdForWindow**](https://docs.microsoft.com/uwp/api/windows.ui.viewmanagement.applicationview.getapplicationviewidforwindow) method.

    ```csharp
    bool viewShown = await ApplicationViewSwitcher.TryShowAsStandaloneAsync(newViewId);
    ```

## The main view


The first view that’s created when your app starts is called the *main view*. This view is stored in the [**CoreApplication.MainView**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.core.coreapplication.mainview) property, and its [**IsMain**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.core.coreapplicationview.ismain) property is true. You don’t create this view; it’s created by the app. The main view's thread serves as the manager for the app, and all app activation events are delivered on this thread.

If secondary views are open, the main view’s window can be hidden – for example, by clicking the close (x) button in the window title bar - but its thread remains active. Calling [**Close**](https://docs.microsoft.com/uwp/api/windows.ui.xaml.window.close) on the main view’s [**Window**](https://docs.microsoft.com/uwp/api/Windows.UI.Xaml.Window) causes an **InvalidOperationException** to occur. (Use [**Application.Exit**](https://docs.microsoft.com/uwp/api/windows.ui.xaml.application.exit) to close your app.) If the main view’s thread is terminated, the app closes.

## Secondary views


Other views, including all views that you create by calling [**CreateNewView**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.core.coreapplication.createnewview) in your app code, are secondary views. Both the main view and secondary views are stored in the [**CoreApplication.Views**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.core.coreapplication.views) collection. Typically, you create secondary views in response to a user action. In some instances, the system creates secondary views for your app.

> [!NOTE]
> You can use the Windows *assigned access* feature to run an app in [kiosk mode](https://technet.microsoft.com/library/mt219050.aspx). When you do this, the system creates a secondary view to present your app UI above the lock screen. App-created secondary views are not allowed, so if you try to show your own secondary view in kiosk mode, an exception is thrown.

## Switch from one view to another

Consider providing a way for the user to navigate from a secondary window back to its parent window. To do this, use the [**ApplicationViewSwitcher.SwitchAsync**](https://docs.microsoft.com/uwp/api/windows.ui.viewmanagement.applicationviewswitcher.switchasync) method. You call this method from the thread of the window you're switching from and pass the view ID of the window you're switching to.

```csharp
await ApplicationViewSwitcher.SwitchAsync(viewIdToShow);
```

When you use [**SwitchAsync**](https://docs.microsoft.com/uwp/api/windows.ui.viewmanagement.applicationviewswitcher.switchasync), you can choose if you want to close the initial window and remove it from the taskbar by specifying the value of [**ApplicationViewSwitchingOptions**](https://docs.microsoft.com/uwp/api/Windows.UI.ViewManagement.ApplicationViewSwitchingOptions).

## Do's and don'ts

* Do provide a clear entry point to the secondary view by utilizing the "open new window" glyph.
* Do communicate the purpose of the secondary view to users.
* Do ensure that your app works is fully functional in a single view and users will open a secondary view only for convenience.
* Don't rely on the secondary view to provide notifications or other transient visuals.

## Related topics

* [ApplicationViewSwitcher](https://docs.microsoft.com/uwp/api/Windows.UI.ViewManagement.ApplicationViewSwitcher)
* [CreateNewView](https://docs.microsoft.com/uwp/api/windows.applicationmodel.core.coreapplication.createnewview)
 
