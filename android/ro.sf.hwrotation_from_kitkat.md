

This commit backport ro.sf.hwrotation to Android 5.x

<pre>
	commit fdba9e2ffd27db20a48b99a488eacfb6d1c30408
	Author: Liu Xiaowen <b37945@freescale.com>
	Date:   Mon Jan 27 10:38:40 2014 +0800

	    ENGR44307472 add ro.sf.hwrotation back to kitkat.
	    
	    add ro.sf.hwrotation back to kitkat.
	    ro.sf.hwrotation should be set in init.rc.
	    
	    Signed-off-by: Liu Xiaowen <b37945@freescale.com>
	    
	    Conflicts:
		services/surfaceflinger/SurfaceFlinger.cpp

	<b>diff --git a/services/surfaceflinger/DisplayDevice.cpp b/services/surfaceflinger/DisplayDevice.cpp</b>
	index fde20e5..a2acb18 100644
	--- a/services/surfaceflinger/DisplayDevice.cpp
	+++ b/services/surfaceflinger/DisplayDevice.cpp
	@@ -443,6 +443,22 @@ void DisplayDevice::setProjection(int orientation,
	     Rect viewport(newViewport);
	     Rect frame(newFrame);
	 
	+    int displayOrientation = DisplayState::eOrientationDefault;
	+    char property[PROPERTY_VALUE_MAX];
	+    if (mType == DISPLAY_PRIMARY) {
	+        if (property_get("ro.sf.hwrotation", property, NULL) > 0) {
	+            switch (atoi(property)) {
	+                case 90:
	+                    displayOrientation = DisplayState::eOrientation90;
	+                    break;
	+                case 270:
	+                    displayOrientation = DisplayState::eOrientation270;
	+                    break;
	+            }
	+        }
	+    }
	+
	     const int w = mDisplayWidth;
	     const int h = mDisplayHeight;
	 
	@@ -453,6 +469,11 @@ void DisplayDevice::setProjection(int orientation,
		 // the destination frame can be invalid if it has never been set,
		 // in that case we assume the whole display frame.
		 frame = Rect(w, h);
	+        if (R.getOrientation() & Transform::ROT_90) {
	+            // frame is always specified in the logical orientation
	+            // of the display (ie: post-rotation).
	+            swap(frame.right, frame.bottom);
	+        }
	     }
	 
	     if (viewport.isEmpty()) {

	<b>diff --git a/services/surfaceflinger/SurfaceFlinger.cpp b/services/surfaceflinger/SurfaceFlinger.cpp</b>
	index 6acc2d0..f86a483 100644
	--- a/services/surfaceflinger/SurfaceFlinger.cpp
	+++ b/services/surfaceflinger/SurfaceFlinger.cpp
	@@ -14,7 +14,7 @@
	  * limitations under the License.
	  */
	 
	-/* Copyright (C) 2013 Freescale Semiconductor, Inc. */ +/* Copyright (C) 2013-2014 Freescale Semiconductor, Inc. */
	 
	 #define ATRACE_TAG ATRACE_TAG_GRAPHICS
	 
	@@ -584,8 +584,30 @@ status_t SurfaceFlinger::getDisplayConfigs(const sp<IBinder>& display,
		     info.orientation = 0;
		 }
	 
	-        info.w = hwConfig.width;
	-        info.h = hwConfig.height;
	+        int width = hwConfig.width;
	+        int height = hwConfig.height;
	+
	+        int displayOrientation = DisplayState::eOrientationDefault;
	+        char property[PROPERTY_VALUE_MAX];
	+        if (type == DisplayDevice::DISPLAY_PRIMARY) {
	+            if (property_get("ro.sf.hwrotation", property, NULL) > 0) {
	+                switch (atoi(property)) {
	+                    case 90:
	+                        displayOrientation = DisplayState::eOrientation90;
	+                        break;
	+                    case 270:
	+                        displayOrientation = DisplayState::eOrientation270;
	+                        break;
	+                }
	+            }
	+        }
	+
	+        if (displayOrientation & DisplayState::eOrientationSwapMask) {
	+            swap(width, height);
	+        }
	+
	+        info->w = width;
	+        info->h = height;
		 info.xdpi = xdpi;
		 info.ydpi = ydpi;
		 info.fps = float(1e9 / hwConfig.refresh);


	<b>diff --git a/services/inputflinger/InputReader.cpp b/services/inputflinger/InputReader.cpp</b>
	index 8634e42..41fab0d 100644
	--- a/services/inputflinger/InputReader.cpp
	+++ b/services/inputflinger/InputReader.cpp
	@@ -42,6 +42,7 @@
	#include "InputReader.h"

	#include <cutils/log.h>
	+#include <cutils/properties.h>
	#include <input/Keyboard.h>
	#include <input/VirtualKeyMap.h>

	@@ -2969,6 +2970,19 @@ void TouchInputMapper::configureSurface(nsecs_t when, bool* outResetNeeded) {
	int32_t rawWidth = mRawPointerAxes.x.maxValue - mRawPointerAxes.x.minValue + 1;
	int32_t rawHeight = mRawPointerAxes.y.maxValue - mRawPointerAxes.y.minValue + 1;

	+    char hwrotBuf[PROPERTY_VALUE_MAX];
	+    int32_t hwrotation = DISPLAY_ORIENTATION_0;
	+    if (property_get("ro.sf.hwrotation", hwrotBuf, NULL) > 0) {
	+        switch (atoi(hwrotBuf)) {
	+            case 90:
	+                hwrotation = DISPLAY_ORIENTATION_90;
	+                break;
	+            case 270:
	+                hwrotation = DISPLAY_ORIENTATION_270;
	+                break;
	+        }
	+    }
	+
	// Get associated display dimensions.
	DisplayViewport newViewport;
	if (mParameters.hasAssociatedDisplay) {
	@@ -2980,8 +2994,16 @@ void TouchInputMapper::configureSurface(nsecs_t when, bool* outResetNeeded) {
	     mDeviceMode = DEVICE_MODE_DISABLED;
	     return;
	 }
	+               newViewport.orientation = (newViewport.orientation + hwrotation) % 4;
	} else {
	+        if ((hwrotation == DISPLAY_ORIENTATION_90 ||
	+                hwrotation == DISPLAY_ORIENTATION_270)) {
	+            int tmp = rawWidth;
	+            rawWidth = rawHeight;
	+            rawHeight = tmp;
	+        }
	 newViewport.setNonDisplayViewport(rawWidth, rawHeight);
	+               newViewport.orientation = hwrotation;
	}
	bool viewportChanged = mViewport != newViewport;
	if (viewportChanged) {
</pre>


But there is problem when new configuration changed and suspend, such as USB keyboard plugged. Need add hwrotation check in ColorFade and ScreenRotationAnimation. Since hwrotation is not Android original configuration already, it's better to check any function referring to DisplayInfo::getDisplayInfo.

<pre>
    // "frameworks/base/services/core/java/com/android/server/display/ColorFade.java"
	public void onDisplayTransaction() {
	synchronized (this) {
	    if (mSurfaceControl == null) {
		return;
	    }

	    DisplayInfo displayInfo = mDisplayManagerInternal.getDisplayInfo(mDisplayId);
	    if (MYDEBUG) {
		    Slog.w(TAG, "displayInfo is " + displayInfo + " force to " + Surface.ROTATION_270);
		    <b> displayInfo.rotation = Surface.ROTATION_270; </b>
	    }

	   switch (displayInfo.rotation) {
		case Surface.ROTATION_0:
		    mSurfaceControl.setPosition(0, 0);
		    mSurfaceControl.setMatrix(1, 0, 0, 1);
		    break;
		case Surface.ROTATION_90:
		    mSurfaceControl.setPosition(0, displayInfo.logicalHeight);
</pre>

<pre>
	diff --git a/services/core/java/com/android/server/wm/ScreenRotationAnimation.java b/services/core/java/com/android/server/wm/ScreenRotationAnimation.java
	index f79896b..ce1f2b0 100644
	--- a/services/core/java/com/android/server/wm/ScreenRotationAnimation.java
	+++ b/services/core/java/com/android/server/wm/ScreenRotationAnimation.java

	@@ -210,6 +210,9 @@ class ScreenRotationAnimation {
	     int originalRotation = display.getRotation();
	     final int originalWidth;
	     final int originalHeight;
	+
	+       <b> originalRotation += 3; </b>
	+

	     DisplayInfo displayInfo = displayContent.getDisplayInfo();
	     if (forceDefaultOrientation) {
		 // Emulated orientation.
</pre>
