---
name: rea-android-apk-analysis
description: Reverse engineer Android applications (APK, AAB, DEX) using REA. Decompile classes with headless JADX, audit AndroidManifest permissions and exported components, trace method references, and perform dynamic ADB/Frida instrumentation.
---

# Android Application Analysis with REA

This skill provides comprehensive procedures for static and dynamic reverse engineering of Android package files (APK/AAB), Dalvik/ART DEX bytecode, and associated native JNI libraries.

---

## 1. Static Package & Manifest Audit

Begin analysis directly on the target APK without unzipping manually:

```json
{
  "path": "/path/to/target.apk"
}
```
- Tool: `inspect_android_package`
- **Emits**:
  - Package ID and version code/name.
  - Min SDK, Target SDK, and Compile SDK versions.
  - Declared Android permissions (`uses-permission`).
  - Component declarations:
    - **Activities** (UI screens).
    - **Services** (Background workers).
    - **Broadcast Receivers** (Event listeners).
    - **Content Providers** (Data storage interfaces).

### Key Security Vulnerability Checklist
- [ ] **Exported Components**: Identify any component with `android:exported="true"` lacking permission checks (`android:permission`).
- [ ] **Dangerous Permissions**: Audit requests for `READ_EXTERNAL_STORAGE`, `ACCESS_FINE_LOCATION`, `RECORD_AUDIO`, `READ_SMS`, `CAMERA`.
- [ ] **Deep Links & Schemes**: Examine `<intent-filter>` declarations with `<data android:scheme="..." android:host="...">` for client-side injection or unauthorized invocation.
- [ ] **Debuggable Flag**: Check whether `android:debuggable="true"` is enabled in production releases.

---

## 2. Headless JADX Decompilation Workflow

REA integrates with headless JADX to decompile Dalvik bytecode into readable Java/Kotlin source without launching external GUIs.

### Step 1: Search Classes
Locate classes associated with authentication, networking, cryptography, or proprietary logic:
```json
{
  "path": "/path/to/target.apk",
  "query": "Auth|Token|Login|ApiClient|Security",
  "limit": 30
}
```
- Tool: `search_android_classes`

### Step 2: Inspect Class Members
Inspect methods, fields, and constructors of a selected class:
```json
{
  "path": "/path/to/target.apk",
  "class_name": "com.example.app.network.ApiClient"
}
```
- Tool: `inspect_android_class`

### Step 3: Decompile Method to Java
Decompile a specific method to inspect its implementation:
```json
{
  "path": "/path/to/target.apk",
  "class_name": "com.example.app.network.ApiClient",
  "method_name": "signRequest"
}
```
- Tool: `inspect_android_method`
- If overloaded methods exist, provide the `overload_index` reported by `inspect_android_class`.

### Step 4: Trace Static References
Trace incoming callers across the entire application bytecode:
```json
{
  "path": "/path/to/target.apk",
  "class_name": "com.example.app.crypto.KeyManager",
  "method_name": "getSecretKey"
}
```
- Tool: `trace_android_references`
- Identifies all call sites within activities, services, or helper classes.

---

## 3. Native JNI Libraries (`.so` Files)

Many modern Android applications offload sensitive logic (integrity checks, cryptographic keys, license validation) to native C/C++ libraries stored in `lib/arm64-v8a/` or `lib/x86_64/`:

1. **Extract Native Libraries**:
   - Use `open_binary` on the APK to inspect archive members (`inspect_artifact`).
   - Extract `.so` files of interest (e.g., `libnative-lib.so`).
2. **Analyze via Native Workflow**:
   - Dispatch to `rea-native-binary-analysis` skill.
   - Decompile native entry points such as `Java_com_example_app_NativeLib_stringFromJNI`.

---

## 4. Dynamic Instrumentation with ADB & Frida

When static analysis is obscured by obfuscators (ProGuard, R8, DexGuard):

```json
// Start application via ADB on connected emulator or device
start_adb_app({
  "package_name": "com.example.app",
  "activity_name": ".ui.MainActivity"
})

// Attach Frida session for runtime hooking
start_frida_session({
  "target": "com.example.app"
})

// Inject custom hook script to monitor function calls or bypass SSL pinning
load_frida_script({
  "script": "Java.perform(function() { var Api = Java.use('com.example.app.network.ApiClient'); Api.signRequest.implementation = function(req) { console.log('Request payload: ' + req); return this.signRequest(req); }; });"
})
```
- Conclude dynamic tasks with `close_frida_session` and `stop_adb_app`.
