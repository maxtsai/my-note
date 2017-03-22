### Issue - WindowAnimation doesn't make effect on correct position in some cases such as WiFi password setting.
#
#### Android 6.x source trace
#### SurfaceInsets was assigned with non-zero value as below.

<pre>
//frameworks/base/core/java/android/view/ViewRootImpl.java
                // Compute surface insets required to draw at specified Z value.
                // TODO: Use real shadow insets for a constant max Z.
                if (!attrs.hasManualSurfaceInsets) {
                    final int surfaceInset = (int) Math.ceil(view.getZ() * 2);
                    attrs.surfaceInsets.set(surfaceInset, surfaceInset, surfaceInset, surfaceInset);
                }  
</pre>

#### WindowsStateAnimator.java gives (left,top) with offset from surfceInsets.

<pre>
//frameworks/base/services/core/java/com/android/server/wm/WindowManagerService.java :: relayoutWindow()
                try {
                    if (!win.mHasSurface) {
                        surfaceChanged = true;
                    }
                    SurfaceControl surfaceControl = winAnimator.createSurfaceLocked();

//frameworks/base/services/core/java/com/android/server/wm/WindowStateAnimator.java :: createSurfaceLocked()
            // Adjust for surface insets.
            width += attrs.surfaceInsets.left + attrs.surfaceInsets.right;
            height += attrs.surfaceInsets.top + attrs.surfaceInsets.bottom;
            left -= attrs.surfaceInsets.left;
            top -= attrs.surfaceInsets.top;
</pre>

#### WindowStateAnimation.java set position to SurfaceControl

<pre>
//frameworks/base/services/core/java/com/android/server/wm/WindowStateAnimator.java :: createSurfaceLocked()
            // Start a new transaction and apply position & offset.
            SurfaceControl.openTransaction();
            try {
                mSurfaceX = left;
                mSurfaceY = top;

                try {
                    mSurfaceControl.setPosition(left, top);                                  
</pre>

#### SurfaceControl call nativeSetPosition(), then SurfaceComposerClient.cpp::setPosition is executed
#### eventually, layer.cpp::SetPosition is executed.

<pre>
//frameworks/native/services/surfaceflinger/Layer.cpp
bool Layer::setPosition(float x, float y) {
    if (mCurrentState.transform.tx() == x && mCurrentState.transform.ty() == y)
        return false;
    mCurrentState.sequence++;
    mCurrentState.transform.set(x, y);
    setTransactionFlags(eTransactionNeeded);
    return true;
}
</pre>

#### At this moment, transform in layer is changed. Anything related with transform matrix will be changed.



