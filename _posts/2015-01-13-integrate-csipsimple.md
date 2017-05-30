---
layout: post
title:  "整合csipsimple"
date:   2015-01-13
categories: 職場 integration sip wireshark Android
---

最近公司案子要用到sip（原本搞WebRTC了三個月才又改成這個）自己架了signalling server之後又說要用pjsua當libarary做，一看下去才知道pjsua現在[Android平台上根本還不支援影像](http://trac.pjsip.org/repos/milestone/release-2.4)！

搜尋了一下發現[csipsimple](https://code.google.com/p/csipsimple/)根本就是**終極神器**：一個人自幹了四年，功能多到爆：

- 支援多國sip服務（根本都沒聽過）
- 多國語言
- 整合Android內容撥號功能，甚至可以加入系統聯絡人跟撥號紀錄！
- 整合pjsua, webrtc (from google)以及多種影音codec
- 重點是現在已經有影像功能的測試版apk！

不是它還有誰可以當我的救贖呢？

一切都從**SipService**開始，它啟動後會開始監聽裝置狀態跟Sip帳號的更動，讓它可以在使用者設定完成後就開始依序的相關動作。Register receiver有兩個 method，其中一個是監視 service的狀態。

要整合 csipsimple到自己的專案，你需要做的是：

- 複製所有 so檔到jniLibs裡（略過重新編譯 jni裡所有C/C++原始碼的麻煩）
- 複製 com.csipsimple下除了 com.csipsimple.ui 之外的所有 package（因為我只用到 Basic wizard所以只複製了相關的物件，其他類型的 wizard不用理）
- 將 csipsimple AndroidManifest.xml 裡宣告的 Service, Receiver 複製到 自己的  AndroidManifest.xml。
> 先（最好）不要修改 csipsimple 原本的 package名稱，因為 csipsimple 內部元件互動幾乎都是靠 定義在SipConfigManager裡面的intent action，若更動可能得花很多時間測試才知道正確的組合！我在自己專案的 start up activity裡要啟動 SipService 就卡關很久！才發現它一直隻找不到要啟動的 Service。

- 另外就是啟動 Service的 intent.setPackage()可以註解掉。
> Android 5.0之後不允許implicit intent，所以又會碰到無法找到SipService的狀況。解法可參考下面這段code(忘記在哪找到的..)

```language-java
/***
     * Android L (lollipop, API 21) introduced a new problem when trying to invoke implicit intent,
     * "java.lang.IllegalArgumentException: Service Intent must be explicit"
     *
     * If you are using an implicit intent, and know only 1 target would answer this intent,This method will help you turn the implicit intent into the explicit form.
     *
     * Inspired from SO answer: http://stackoverflow.com/a/26318757/1446466
     * @param context
     * @param implicitIntent - The original implicit intent
     * @return Explicit Intent created from the implicit original intent
     */
    public static Intent createExplicitFromImplicitIntent(Context context, Intent implicitIntent) {
        // Retrieve all services that can match the given intent
        PackageManager pm = context.getPackageManager();
        List<ResolveInfo> resolveInfo = pm.queryIntentServices(implicitIntent, 0);

        // Make sure only one match was found
        if (resolveInfo == null || resolveInfo.size() != 1) {
            return null;
        }

        // Get component info and create ComponentName
        ResolveInfo serviceInfo = resolveInfo.get(0);
        String packageName = serviceInfo.serviceInfo.packageName;
        String className = serviceInfo.serviceInfo.name;
        ComponentName component = new ComponentName(packageName, className);

        // Create a new intent. Use the old one for extras and such reuse
        Intent explicitIntent = new Intent(implicitIntent);

        // Set the component to be explicit
        explicitIntent.setComponent(component);

        return explicitIntent;
    }
```

- 參考 #SipHome 的 startSipService()
```language-java
// Check if returned compName is null
ComponentName compName = startService(intent);
```
- AIDL檔的位置記得在 build.gradle 裡特別指定，因為 csipsimple 所使用的 AIDL檔放在 com.csipsimple.api 裡，而不是 Android Studio 預設的 /java/main/aidl。在build.gradle裡指定AIDL路徑的語法是：
```xml
android {
    compileSdkVersion 21
    buildToolsVersion "21.1.0"
    ...
	sourceSets {
        main {
            aidl.srcDirs = ['src/main/java']
        }
    }
}
```
- 新增一筆 Sip 帳號
> 用 BasePrefsWizard 來新增一個 Basic 帳號（帳號密碼，伺服器url，給一個帳號別名）

- 播出電話
> 參考 DialerFragment 的 placeCall(); 裡面有如何從 fragment 連上 Service 的做法：bindService();
下面是我自己改的撥出語音電話跟影像電話
```language-java
private void placeAudioCall() {
	dialFeedback.giveFeedback(3001);
	placeCallWithOption(null);
}

private void placeVideoCall() {
	dialFeedback.giveFeedback(3001);
	Bundle b = new Bundle();
	b.putBoolean(SipCallSession.OPT_CALL_VIDEO, true);
	placeCallWithOption(b);
}
```

本篇文章未完成，更新會持續補上。
