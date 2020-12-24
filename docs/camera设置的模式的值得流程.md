

参考链接：[Camera HAL3的参数传递（CameraMetadata）](https://www.cnblogs.com/blogs-of-lxl/p/10981303.html)



例如HDR模式设置流程：

src\main\java\com\otaliastudios\cameraview\engine\options\Camera2Options.java



```java
// HDR
supportedHdr.add(Hdr.OFF);
int[] sceneModes = cameraCharacteristics.get(CONTROL_AVAILABLE_SCENE_MODES);
//noinspection ConstantConditions
for (int sceneMode : sceneModes) {
    Hdr value = mapper.unmapHdr(sceneMode);
    if (value != null) supportedHdr.add(value);
}
```



```java
public final class CameraCharacteristics extends CameraMetadata<CameraCharacteristics.Key<?>> {
```



```java
@PublicKey
@NonNull
public static final Key<int[]> CONTROL_AVAILABLE_SCENE_MODES =
        new Key<int[]>("android.control.availableSceneModes", int[].class);
```



```java
CameraCharacteristics extends CameraMetadata<CameraCharacteristics.Key<?>>
```

android\hardware\camera2\CameraMetadata.java



```java
public abstract class CameraMetadata<TKey> {
```

**Framework到HAL层的转换**

frameworks\base\core\java\android\hardware\camera2\impl\CameraMetadataNative.java



```java
private <T> T getBase(Key<T> key) {
        int tag = nativeGetTagFromKeyLocal(key.getName());
        byte[] values = readValues(tag);
        if (values == null) {
            // If the key returns null, use the fallback key if exists.
            // This is to support old key names for the newly published keys.
            if (key.mFallbackName == null) {
                return null;
            }
            tag = nativeGetTagFromKeyLocal(key.mFallbackName);
            values = readValues(tag);
            if (values == null) {
                return null;
            }
        }

        int nativeType = nativeGetTypeFromTagLocal(tag);
        Marshaler<T> marshaler = getMarshalerForKey(key, nativeType);
        ByteBuffer buffer = ByteBuffer.wrap(values).order(ByteOrder.nativeOrder());
        return marshaler.unmarshal(buffer);
    }
```



**Native层对应代码位置**：frameworks/base/core/jni/android_hardware_camera2_CameraMetadata.cpp



```cpp
static const JNINativeMethod gCameraMetadataMethods[] = {
// static methods
  { "nativeGetTagFromKey",
    "(Ljava/lang/String;J)I",
    (void *)CameraMetadata_getTagFromKey },
  { "nativeGetTypeFromTag",
    "(IJ)I",
    (void *)CameraMetadata_getTypeFromTag },
  { "nativeSetupGlobalVendorTagDescriptor",
    "()I",
    (void*)CameraMetadata_setupGlobalVendorTagDescriptor },
// instance methods
......
```

