### JDK 1.6.0_45

<pre>
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update && sudo apt-get install oracle-java6-installer
java -version
</pre>

<pre>
sudo update-alternatives --config java
sudo update-alternatives --config javac
sudo update-alternatives --config javaws
sudo update-alternatives --config jar
sudo update-alternatives --config javadoc
sudo update-alternatives --config javap
</pre>

Or change java setting by update-java-alternatives

<pre>
sudo update-java-alternatives -l
sudo update-java-alternatives -s java-1.7.0-openjdk-amd64
</pre>

### Problems of building Android 6.x

<pre>
clang: error: linker command failed with exit code 1 (use -v to see invocation)
build/core/host_shared_library_internal.mk:51: recipe for target 'out/host/linux-x86/obj/lib/libart.so' failed
</pre>

#### Disable clang for art build

<pre>
diff --git a/build/Android.common_build.mk b/build/Android.common_build.mk
index b84154b..8cf41c0 100644
--- a/build/Android.common_build.mk
+++ b/build/Android.common_build.mk
@@ -74,7 +74,7 @@ ART_TARGET_CFLAGS :=
 ART_HOST_CLANG := false
 ifneq ($(WITHOUT_HOST_CLANG),true)
   # By default, host builds use clang for better warnings.
-  ART_HOST_CLANG := true
+  ART_HOST_CLANG := false
 endif
 
 # Clang on the target. Target builds use GCC by default.
</pre>
