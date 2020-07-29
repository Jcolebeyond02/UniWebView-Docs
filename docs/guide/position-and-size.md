# Position and Size

UniWebView has two ways to determine where the web view should be displayed on the screen.

## Setting Frame

You could set the position and size by using `Frame` property. 

### Origin and Size

You could define a rect in your screen and set it as the frame of the web view. 
The coordinate system used by UniWebView is a top-left based coordinate. The origin point (0, 0) is the most top-left point of your screen. And the screen size is defined by Unity `Screen` type as `Screen.width` and `Screen.height`:

![](/images/RectXY.svg)

In this coordinate system, you could set the position and size of the web view quite easy:

```csharp
// Make the web view full screen:
webView.Frame = new Rect(0, 0, Screen.width, Screen.height);

// Make the web view take the bottom half of screen:
webView.Frame = new Rect(0, Screen.height/2, Screen.width, Screen.height / 2);

// Make the web view insets from all sides by 10 units:
webView.Frame = new Rect(10, 10, Screen.width - 20, Screen.height - 20);
```

> Although you could set the absolute number of the unit like the final example above, it means little in a mobile device, since the screen sizes are varying from device to device. So a correct way to use the `Frame` property is set it with a relative value of current screen height and width. 

A fixed value is totally OK if you do not care about things like screen solution and different screen sizes, but it is quite hard to adapt to all your target devices accurately. So we suggest only use it when you are going to set the web view size to a relative size to the screen.

If you want more control on the position and size, UniWebView supports another way to get benefit from Resolution  & Device Independence concept from Unity UI and **Canvas Scaler**. Read the "Using Reference UI Element" below to know how to use it.

::: warning IMPORTANT
`Frame` property will be ignored if you use the method described in "Using Reference UI Element" (or say, if the `ReferenceRectTransform` property is not `null`).
:::

### Orientation Change

By setting the `Frame` property, you just set it based on the current situation and screen. The position and size will not get automatically updated when the screen size changes, which will happen on a mobile platform if the orientation is changed.

If you need to support auto orientation while displaying the web view, you may need to listen to the `OnOrientationChanged` event and take the response to update `Frame` property when this event raised. For example, the code below will set the web view full screen and also keep it full screen when the orientation changes:

```csharp
// Let's say we are setting the frame while the screen is in portrait with 320x640
// Now the Screen.width is 320 and Screen.height is 640. We set the web view full screen.
webView.Frame = new Rect(0, 0, Screen.width, Screen.height);

// Add a method which will be invoked when the orientation changes:
webView.OnOrientationChanged += (view, orientation) => {
    // For example it is from portrait to landscape, now it is 640x320 (width x height)
    // By setting again, we could keep the web view full screen.
    webView.Frame = new Rect(0, 0, Screen.width, Screen.height);
};
```

## Using Reference UI Element

`ReferenceRectTransform` is a property on `UniWebView` that you could set a reference `RectTransform` to. A `RectTransform` is a part of your UI element and it defines a `Rect` which attached to a transform. Unity UI could be designed to fit for multiple resolutions by using a canvas with Canvas Scaler. If you are not familiar with it, we recommend you to read the [Designing UI for Multiple Resolutions](https://docs.unity3d.com/Manual/HOWTO-UIMultiResolution.html) first.

It is possible for you to define the position and side of UniWebView to an existing `RectTransform` in your UI element. Normally, you could create a panel or any other UI element and use it as a reference rect transform. By setting `ReferenceRectTransform` to the reference rect transform, UniWebView will position and resize itself to match the position and size of the rect. So you could skip to calculate the `Frame` and adapt to devices with different sizes and resolutions.

Once you have a `RectTransform`, you could just set the property:

```csharp
RectTransform myUITransform = ...
webView.ReferenceRectTransform = myUITransform;
```

And everything should be done.

> The `RectTransform` way supports all three kinds render mode ("Screen Space - Overlay", "Screen Space - Camera" and "World Space") on the first `Canvas` from the transform above. However, if you are using a camera-depended space, please make sure you have set up the camera correctly for the render mode. Otherwise, `RectTransform` will fall back as there is no camera and the size/location might be incorrect.

### Rect Transform Change

Similar to the case for `Frame`, `ReferenceRectTransform` only consider the current state of the referenced transform when set. However, since we know the reference rect will be changed when the screen orientation changes, there is no need to update it again in `OnOrientationChanged` event. UniWebView will update its position and size automatically for you.

But besides of the orientation changing, there is a chance that the referred transform changed by you. You take the responsibility to update the web view position and size in this case. There is an `OnRectTransformDimensionsChange` in `UIBehavior`, which will be called by Unity when the attached rect transform changed. You could use this trick to update web view's frame:

```csharp
// In a UIBehavior script:
// Called when associated `rectTransform` is changed.
void OnRectTransformDimensionsChange() {
    // This will update web view's frame to match the reference rect transform if set.
    webView.UpdateFrame();
}
```

`UpdateFrame` will respect your settings and get the latest status from the `ReferenceRectTransform`, then update the frame to fit the rect transform.
