# R8 Test

This is a sample project to show R8 ignoring the Proguard configuration line to retain line numbers as described in the official [DOCS](https://developer.android.com/studio/build/shrink-code#decode-stack-trace).

## Issue

R8 ignores the proguard configuration needed to retain line numbers in stack traces in release builds.

```
-keepattributes LineNumberTable,SourceFile
```

This project showcases this bug. There is a single screen with a "Next" button. Tapping on the "Next" button creates a `java.lang.Throwable` instance and logs it to the console.

Here is a sample of the stack trace when R8 is enabled and the `debuggable` flag is set to `true`.

```
2021-05-28 12:13:31.519 14034-14034/in.obvious.r8test I/R8Test: Error
    java.lang.Throwable
        at in.obvious.r8test.FirstFragment.onViewCreated$lambda-0(FirstFragment.kt:37)
        at in.obvious.r8test.FirstFragment.lambda$1w_HuEsUHmOLKQ58J-F6IcSYm_I(Unknown Source:0)
        at in.obvious.r8test.-$$Lambda$FirstFragment$1w_HuEsUHmOLKQ58J-F6IcSYm_I.onClick(Unknown Source:0)
        at android.view.View.performClick(View.java:7869)
        at android.widget.TextView.performClick(TextView.java:14958)
        at com.google.android.material.button.MaterialButton.performClick(MaterialButton.java:1119)
        at android.view.View.performClickInternal(View.java:7838)
        at android.view.View.access$3600(View.java:886)
        at android.view.View$PerformClick.run(View.java:29362)
        at android.os.Handler.handleCallback(Handler.java:883)
        at android.os.Handler.dispatchMessage(Handler.java:100)
        at android.os.Looper.loop(Looper.java:237)
        at android.app.ActivityThread.main(ActivityThread.java:8107)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:496)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1100)
```

However, setting the `debuggable` flag to `false` and logging the error now removes the line numbers (See `FirstFragment.kt`).

```
2021-05-28 12:16:20.191 14461-14461/? I/R8Test: Error
    java.lang.Throwable
        at in.obvious.r8test.-$$Lambda$FirstFragment$1w_HuEsUHmOLKQ58J-F6IcSYm_I.onClick(lambda:2)
        at android.view.View.performClick(View.java:7869)
        at android.widget.TextView.performClick(TextView.java:14958)
        at com.google.android.material.button.MaterialButton.performClick(MaterialButton.java:2)
        at android.view.View.performClickInternal(View.java:7838)
        at android.view.View.access$3600(View.java:886)
        at android.view.View$PerformClick.run(View.java:29362)
        at android.os.Handler.handleCallback(Handler.java:883)
        at android.os.Handler.dispatchMessage(Handler.java:100)
        at android.os.Looper.loop(Looper.java:237)
        at android.app.ActivityThread.main(ActivityThread.java:8107)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:496)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1100)
```

Disabling R8 and using Proguard instead retains the line numbers as expected in FirstFragment.

```
2021-05-28 12:21:17.915 14786-14786/? I/R8Test: Error
    java.lang.Throwable
        at in.obvious.r8test.FirstFragment.onViewCreated$lambda-0(FirstFragment.kt:37)
        at in.obvious.r8test.FirstFragment.lambda$1w_HuEsUHmOLKQ58J-F6IcSYm_I(Unknown Source:0)
        at in.obvious.r8test.-$$Lambda$FirstFragment$1w_HuEsUHmOLKQ58J-F6IcSYm_I.onClick(Unknown Source:0)
        at android.view.View.performClick(View.java:7869)
        at android.widget.TextView.performClick(TextView.java:14958)
        at com.google.android.material.button.MaterialButton.performClick(MaterialButton.java:1119)
        at android.view.View.performClickInternal(View.java:7838)
        at android.view.View.access$3600(View.java:886)
        at android.view.View$PerformClick.run(View.java:29362)
        at android.os.Handler.handleCallback(Handler.java:883)
        at android.os.Handler.dispatchMessage(Handler.java:100)
        at android.os.Looper.loop(Looper.java:237)
        at android.app.ActivityThread.main(ActivityThread.java:8107)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:496)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1100)
```

To try this with Proguard, do the following steps:

- Change `android.enableR8` property in `gradle.properties` to `false`.
- Uncomment the "Only needed for using Proguard" in `proguard-rules.pro` section.
