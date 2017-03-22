### Issue - WindowAnimation doesn't make effect on correct position in some cases such as WiFi password setting.

Tracing History
<code>
                // Compute surface insets required to draw at specified Z value.
                // TODO: Use real shadow insets for a constant max Z.
                if (!attrs.hasManualSurfaceInsets) {
                    final int surfaceInset = (int) Math.ceil(view.getZ() * 2);
</code>
