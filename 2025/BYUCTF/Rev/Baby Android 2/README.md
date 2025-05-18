## Baby Android 2  
Author: overllama  
Challenge prompt: If you've never reverse engineered an Android application, now is the time!! Get to it, already!! Learn more about how they work!!  

TO begin with, we decompile the apk in decompiler.com   
Following that, to look for challenge related clues, we navigate to sources/byuctf/babyandroid/FlagChecker.java  
From there, we're given an APK that asks for a flag input and checks it using a native method:  
```
public class FlagChecker {
    public static native boolean check(String str);
}
```
The actual flag checking logic is inside a native library:  

lib/arm64-v8a/libbabyandroid.so  

Step-by-Step Solution:  
1. Extract the Native Library   
```
unzip baby_android-2.apk -d baby2_unpacked   
ls baby2_unpacked/lib/arm64-v8a/libbabyandroid.so  
```
2. Load into Ghidra
-Import libbabyandroid.so  
-Run auto-analysis  
-Look for the native JNI function:  
    Java_byuctf_babyandroid_FlagChecker_check  

3. Analyze the Function
Decompiled logic looked like:
```
  char *pcVar1;
  long lVar2;
  int local_60;
  undefined1 local_34;
  basic_string<> abStack_30 [24];
  long local_18;
  
  lVar2 = tpidr_el0;
  local_18 = *(long *)(lVar2 + 0x28);
  pcVar1 = (char *)_JNIEnv::GetStringUTFChars(param_1,param_3,(uchar *)0x0);
  std::__ndk1::basic_string<>::basic_string<>(abStack_30,pcVar1);
  lVar2 = FUN_0011dde8(abStack_30);
  if (lVar2 == 0x17) {
    for (local_60 = 0; local_60 + -0x16 == 0 || local_60 < 0x16; local_60 = local_60 + 1) {
      pcVar1 = (char *)FUN_0011de0c(local_60 + -0x16,abStack_30,(long)local_60);
      if (*pcVar1 != "bycnu)_aacGly~}tt+?=<_ML?f^i_vETkG+b{nDJrVp6=)="[(local_60 * local_60) % 0x2f]
         ) {
        local_34 = 0;
        goto LAB_0011dcf4;
      }
    }
    local_34 = 1;
  }
  else {
    local_34 = 0;
  }
```
The key logic:  
-Flag length must be 23  
-Each character must match a character from a reference string: "bycnu)_aacGly~}tt+?=<_ML?f^i_vETkG+b{nDJrVp6=)="  
-Indexing rule: input[i] == ref[(i*i) % 47]  

# Python Solver
```
ref = "bycnu)_aacGly~}tt+?=<_ML?f^i_vETkG+b{nDJrVp6=)="
flag = ''.join(ref[(i * i) % 47] for i in range(23))
print(flag)
```
Final Flag: byuctf{c++_in_an_apk??}
