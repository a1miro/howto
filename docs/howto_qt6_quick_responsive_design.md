It's common to see Qt6 Quick (Qt Quick/QML) applications built around a fixed-size window (especially for desktop or kiosk apps), relying on fixed sizes can be quite limiting, especially if your application might run on a variety of displays or be ported to devices with different resolutions and aspect ratios.

**Best Practice: Responsive Layouts**

**Advantages of calculating sizes/margins based on screen size:**

- **Scalability:** Your UI will look good on any screen size (desktop, tablet, 4K monitor, etc.).
- **Flexibility:** You can support both landscape and portrait orientations, and adapt to different aspect ratios.
- **Accessibility:** Larger elements on bigger screens may help users with accessibility needs.
- **Future-proofing:** As new devices/screens emerge, your application will be ready.

**How to do this in Qt Quick/QML:**

- **Use Anchors and Layouts:**  
  Instead of absolute `x`, `y`, `width`, and `height`, use anchors (`anchors.fill`, `anchors.margins`) and layouts (`RowLayout`, `ColumnLayout`, `GridLayout`).
- **Reference Parent Size:**  
  Bind sizes to parent properties:  
  ```qml
  Rectangle {
      width: parent.width * 0.5
      height: parent.height * 0.3
  }
  ```
- **Use Screen/Window Size:**  
  Use the root `Window`’s `width` and `height`, or the `Screen` object to access DPI and pixel density for even more nuanced scaling.
  ```qml
  import QtQuick.Window 2.15
  Window {
      width: Screen.width
      height: Screen.height
      // ...
  }
  ```
- **Responsive Units:**  
  You can define “responsive units” (like `dp` in Android) for consistent scaling.
  ```qml
  property real dp: Screen.pixelDensity / 160
  Rectangle { width: 48 * dp; height: 48 * dp }
  ```
the number 160 represents the baseline pixel density of a “standard” screen, measured in dots per inch (dpi). This value comes from Android’s definition of a “medium density” (mdpi) screen, which is 160 dpi. It is used as a reference point for density-independent pixels (dp or dip) in responsive UI design.

Screen.pixelDensity gives you the actual screen’s pixel density in dpi.
By dividing by 160, you get a scaling factor that tells you how much bigger/smaller UI elements should be, compared to the reference “standard” screen.
If you multiply your dimensions by this scaling factor, your UI will be similarly sized on all screens, regardless of their actual pixel density.
Example:
----
On a 320 dpi screen: dp = 320 / 160 = 2. So an element that is 48*dp will be 96 pixels.
On a 160 dpi screen: dp = 160 / 160 = 1. So 48*dp is 48 pixels. 

**When Fixed Size Might Be Justified:**

- If your application is *truly* only ever used in a single screen size or for tightly controlled environments (like a kiosk).
- For prototyping, where speed is more important than flexibility (but try to refactor later!).

**Conclusion:**  
For most modern applications, especially those expected to run on more than one device or screen type, it is *much* wiser to make your UI responsive to the screen size. Qt Quick’s layout system and property binding make this relatively straightforward.

If you want, I can show an example of a responsive Qt Quick layout!
