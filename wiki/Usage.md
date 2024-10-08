Important for 32-bit mobile users: Please remove arm64-v8a from application.mk

#### **modmenu/Preferences.java**

Saving the menu feature preferences and calling changes via JNI

#### **modmenu/FloatingModMenuService.java**

Main codes of floating mod menu design

You don't need to change unless you want to redesign it. The codes are explained in the comments

- `GradientDrawable`: A code for setting corner and stroke/inner border. Works for any View Components

```java
GradientDrawable gradientdrawable = new GradientDrawable();
gradientdrawable.setCornerRadius(20); //Set corner
gradientdrawable.setColor(Color.parseColor("#1C2A35")); //Set background color
gradientdrawable.setStroke(1, Color.parseColor("#32cb00")); //Set border
```

Set the gradient drawable to the view component

```java
[name of your view component].setBackground(gradientdrawable);
```

- Resizing menu box

I've added variables so you can find it easly to resize

```java
private final int MENU_WIDTH = 290;
private final int MENU_HEIGHT = 200;
```

Note: You may need to implement auto sizing due to many types of phone with different DPIs and resolutions

- Color Animation: The codes can be seen in `startAnimation()`

- Adding new view

Normally the Android development documentation does not explain the code in java. If you read the Android development documentation and you see an example like TextView

```java
TextView textView = (TextView) findViewById(R.id.textView);
textView.setFontVariationSettings("'wdth' 150");
```

This is for xml. Instead, create an instance for java and add view to your Layout

```java
TextView textView = new TextView(this);
textView.setFontVariationSettings("'wdth' 150");
LinearLayoutExample.addView(textView);
```

While we can't explain much here, you can use Google. Search like `create a textview programmatically android`, `create a button programmatically android` etc. for more infomation

#### **MainActivity.java**

The Main Activity. Checks if device running Android 6.0 or above and if have overlay permission enabled before starting menu service.

You pretty don't need to work with it unless you are implementing something like login layout.

#### **jni/Menu/Menu.h**

Menu related with JNI calls

- `Title`: Big text

- `Heading`: Little text. Semi HTML is supported. Text will scroll if the text is too long

- `Icon`: Compressed image that is encoded to base64

You can pretty much use any tools for base64 encoding.

We use a simple website https://www.base64encode.org/

Scroll down till you see `Encode files into Base64 format`. Click or tap on the box to select a file

Click on `ENCODE` button and click on `CLICK OR TAP HERE` to download your encoded file. Now you can paste it in cpp code

- `IconWebViewData`: Use icon in Web view with GIF animation support. URL requires internet permission `android.permission.INTERNET`

Examples

```cpp
//From internet: (Requires android.permission.INTERNET)
return env->NewStringUTF("https://i.imgur.com/SujJ85j.gif"); 

//From assets folder: (Requires android.permission.INTERNET)
return env->NewStringUTF("file:///android_asset/example.gif"); 

//Base64 html:
return env->NewStringUTF("data:image/png;base64, <encoded base64 here>");

//Nothing:
return NULL
```

- `settingsList`: Feature list for settings

#### **jni/Main.cpp**

In this file, you will work with your mods. Below `hack_thread`, you write your code to patch with KittyMemory or hook with MShook. You must have learned it already

It has a macro to detect if the ARM architecture is 32-bit or 64-bit on compile-time, it's to avoid using wrong offsets, like using ARMv7 offsets on an ARM64 lib. Check the game's APK what libs it contains before you proceed. If you want to target armeabi-v7a lib, write the code below `#else`. If you want to target arm64-v8a libs, write the code below `#if defined(__aarch64__)`. If the game has both armeabi-v7a and arm64-v8a, save your time and delete arm64-v8a folder. Armv7 will work universally

We know we could do `#if defined(__arm__)` for ARMv7 and `#if defined(__i386__)` for x86, but we will leaving `#else`, so Android Studio doesn't make that part greyed out. We will still using ARMv7 as a primary target

- `Changes`: Get values to apply mods. BE CAREFUL NOT TO ACCIDENTLY REMOVE break;

- `settingsList`: Settings assigned in negative numbers, we keep the positive numbers for mods. Works same as mod features but the call must be implemented in `localChanges(int featureNum, boolean toggle)` in `FloatingModMenuService.java`

- `getFeatureList`: Mod features

Assigning feature numbers is optional. Without it, it will automatically count for you, starting from 0

Assigned feature numbers can be like any numbers 1,3,200,10... instead in order 0,1,2,3,4,5...

Do not change or translate the first text unless you know what you are doing

Toggle, ButtonOnOff and Checkbox can be switched on by default, if you add `True_`. Example: `CheckBox_True_The Check Box`

To learn HTML, go to this page: https://www.w3schools.com/

Usage:

```cpp
(Optional feature number)_Toggle_(feature name)
(Optional feature number)_True_Toggle_(feature name)
(Optional feature number)_SeekBar_(feature name)_(min value)_(max value)
(Optional feature number)_Spinner_(feature name)_(Items e.g. item1,item2,item3)
(Optional feature number)_Button_(feature name)
(Optional feature number)_ButtonOnOff_(feature name)
(Optional feature number)_InputValue_(feature name)
(Optional feature number)_CheckBox_(feature name)
(Optional feature number)_RadioButton_(feature name)_(Items e.g. radio1,radio2,radio3)
RichTextView_(Text with limited HTML support)
RichWebView_(Full HTML support)
ButtonLink_(feature name)_(URL/Link here)
Category_(text)
```

To add a collapse, create a new instance
```cpp
Collapse_The collapse 1
```

Then you can add component views to collapse like

```cpp
CollapseAdd_Toggle_The toggle
123_CollapseAdd_Toggle_The toggle
CollapseAdd_Button_The button
```

#### KittyMemory patching usage:
```cpp
MemoryPatch::createWithHex([Lib Name], [offset], "[hex. With or without spaces]");
[Struct].get_CurrBytes().Modify();
[Struct].get_CurrBytes().Restore();

[Struct].get_TargetAddress();
[Struct].get_PatchSize();
[Struct].get_CurrBytes().c_str();

//Example
hexPatches.GodMode = MemoryPatch::createWithHex(targetLibName, string2Offset(OBFUSCATE("0x123456")), OBFUSCATE("00 00 80 D2 C0 03 5F D6"));
hexPatches.GodMode.Modify();
hexPatches.GodMode.Restore();
```

```cpp
// Patching offsets directly. Strings are automatically obfuscated!
PATCH("0x20D3A8", "00 00 A0 E3 1E FF 2F E1");
PATCH_LIB("libFileB.so", "0x20D3A8", "00 00 A0 E3 1E FF 2F E1");
PATCH_SYM("_SymbolExample", "00 00 A0 E3 1E FF 2F E1");
PATCH_LIB_SYM("_SymbolExample", "00 00 A0 E3 1E FF 2F E1");

//Switchable patches
PATCH_SWITCH("0x400000", "00 00 A0 E3 1E FF 2F E1", boolean);
PATCH_LIB_SWITCH("libil2cpp.so", "0x200000", "00 00 A0 E3 1E FF 2F E1", boolean);
PATCH_SYM_SWITCH("_SymbolExample", "00 00 A0 E3 1E FF 2F E1", boolean);
PATCH_LIB_SYM_SWITCH("libNativeGame.so", "_SymbolExample", "00 00 A0 E3 1E FF 2F E1", boolean);

//Restore patched offset to original
RESTORE("0x400000");
RESTORE_LIB("libil2cpp.so", "0x400000");
RESTORE_SYM("_SymbolExample");
RESTORE_LIB_SYM("libil2cpp.so", "_SymbolExample");
```

Example: https://github.com/MJx0/KittyMemory/blob/master/Android/test/src/main.cpp

Use an online ARM assembly converter like ARMConverter to convert ARM to HEX: https://armconverter.com/

#### Hook usage:
This macro works for both ARMv7 and ARM64. Make sure to use predefined macro `defined(__aarch64__)` and `defined(__arm__)` if you are targeting both archs

Strings for macros are automatically obfuscated. No need to obfuscate!
```cpp
HOOK("0x123456", FunctionExample, old_FunctionExample);
HOOK_LIB("libFileB.so", "0x123456", FunctionExample, old_FunctionExample);
HOOK_NO_ORIG("0x123456", FunctionExample);
HOOK_LIB_NO_ORIG("libFileC.so", "0x123456", FunctionExample);

HOOKSYM("__SymbolNameExample", FunctionExample, old_FunctionExample);
HOOKSYM_LIB("libFileB.so", "__SymbolNameExample", FunctionExample, old_FunctionExample);
HOOKSYM_NO_ORIG("__SymbolNameExample", FunctionExample);
HOOKSYM_LIB_NO_ORIG("libFileB.so", "__SymbolNameExample", FunctionExample);
```

#### JNI register usage:

We register JNI method calls in order to hide function names from disassembler such as IDA Pro

JNI signatures. Can be obtained from smali file

- Void: `V`
- Boolean: `Z`
- Float: `F`
- Integer: `I`
- String: `Ljava/lang/String;`
  
- Void method with int parameter: `(I)V` (`void methodExample(int i)`)
- Void method with int and bool parameters: `(IZ)V` (`void methodExample(int i, boolean b)`)
- Void method with 2 int parameters: `(II)V` (`void methodExample(int i, int i2)`)
- Void method with int and float parameters: `(IF)V` (`void methodExample(int i, float f)`)
- void method with int and String parameters: `(ILjava/lang/String;)V` (`void methodExample(int i, String s)`)
- String array: `()[Ljava/lang/String;` (`String[] strArrayExample()`
- String method with string parameter`(Ljava/lang/String;)Ljava/lang/String;` (`String stringExample(String s)`)

An example of finding class `com/android/support/Preferences` and register a void method `public static native void Changes(...)` with parameters `(Context con, int fNum, String fName, int i, boolean bool, String str)`

```cpp
jclass p = globalEnv->FindClass( OBFUSCATE("com/android/support/Preferences"));
if (p != nullptr){
    static const JNINativeMethod prefmethods[] = {
            { OBFUSCATE("Changes"), OBFUSCATE("(Landroid/content/Context;ILjava/lang/String;IZLjava/lang/String;)V"), reinterpret_cast<void *>(Changes)},
    };

    int pm = globalEnv->RegisterNatives(p, prefmethods, sizeof(prefmethods) / sizeof(JNINativeMethod));
    if (pm != JNI_OK){
        LOGE(OBFUSCATE("Preferences methods error"));
        return pm;
    }
}
else{
    LOGE(OBFUSCATE("JNI error"));
    return JNI_ERR;
}
```

See more about JNI: https://developer.android.com/training/articles/perf-jni#native-libraries

#### **Android.mk**

The make file for the c++ compiler. In that file, you can change the lib name on the `LOCAL_MODULE` line
When you change the lib name, change also on `System.loadLibrary("")` under OnCreate method on `MainActivity.java`
Both must have same name