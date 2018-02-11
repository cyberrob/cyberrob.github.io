---
layout: post
title:  "使用 shell scripts 增加版號並安裝"
date:   2018-02-11
categories: shell script adb gradle
---

最近遇到一個詭異的bug是因為NotificationListenerService在某些Oppo手機上無法正常截收到通知，並顯示到連線的藍芽裝置上。

特別是在App升版之後，原本已經Attach上的Listener可能就收不到通知了！

為了創造出可以在升版後deploy的行為，我從原本的git repo做了另一個branch出來，再加上一個簡單的shell script，讓script來完成後續deploy的動作。

首先是一個 git command:
```shell
$ git worktree add -b test_notification ../test_notification master
```

這個指令用意是要從 master 複製出一份一樣的branch 到另一個位置，當作後續修改設定並打包安裝的專案。

接下來就是兜出一份可以完成撈出目前版號/輸入新版號/gradle打包/安裝/啟動App的script：

```shell
#! /bin/sh
txtrst=$(tput sgr0) # Text reset
txtylw=$(tput setaf 3) # Yellow
txtgrn=$(tput setaf 2) # Green
txtbold=$(tput bold) # Text Bold
path=$1

echo \#\#\#\#\# Found match at: \#\#\#\#\#
cd "$1"/Sunray/app
# print out line contains defined versionCode but not contains "apk"
cat build.gradle | grep -n --color=always "versionCode" | grep -v "apk"
# Ask user to enter new version as input
read -p 'Enter new version (ENTER): ' versionCode

# Check if input is valid integer
if ! [[ "$versionCode" =~ ^[0-9]+$ ]]
then
echo "Sorry! verion must be integer!"
else
echo \#\#\#\#\# As you wish it will be ${txtbold}${txtgrn}$versionCode${txtrst} \#\#\#\#\#
# Replace line contains "versionCode" but not "apk" with new version
sed -i.bak -e "/apk/!s/versionCode.*/versionCode $versionCode/g" build.gradle
cat build.gradle | grep -n --color=always "versionCode" | grep -v "apk"
echo \#\#\#\#\# Version changed to ${txtbold}${txtgrn}"$versionCode"${txtrst}! \#\#\#\#\#
fi

cd "$1"/Sunray
# Build & install alpha debug flavor onto device if there's any
./gradlew installAlphaDebug
# Start activity
adb shell am start com.noodoe.sunray.alpha/com.noodoe.sunray.main.SunrayActivity
# Grant permissions
adb shell pm grant com.noodoe.sunray.alpha android.permission.ACCESS_FINE_LOCATION
adb shell pm grant com.noodoe.sunray.alpha android.permission.WRITE_EXTERNAL_STORAGE
adb shell pm grant com.noodoe.sunray.alpha android.permission.READ_EXTERNAL_STORAGE
adb shell pm grant com.noodoe.sunray.alpha android.permission.CAMERA
adb shell pm grant com.noodoe.sunray.alpha android.permission.READ_SMS
adb shell pm grant com.noodoe.sunray.alpha android.permission.RECEIVE_SMS
adb shell pm grant com.noodoe.sunray.alpha android.permission.READ_PHONE_STATE
adb shell pm grant com.noodoe.sunray.alpha android.permission.CHANGE_CONFIGURATION
adb shell pm grant com.noodoe.sunray.alpha android.permission.READ_CONTACTS

echo \#\#\#\#\# HAPPY TESTING ON VERSION $versionCode \#\#\#\#\#
```

其實對script並不是很熟，所以花了一些時間 調整 grep 跟 sed 指令跟參數。

sed 是整個script中最關鍵的角色，他可以直接替換掉檔案中指定的字串，可說是狠角色！

```shell
// 在 build.gradle 裡 找到含 "versionCode" 那行，且那行不包含 "apk” ，把 $versionCode 替換上去
sed -i.bak -e "/apk/!s/versionCode.*/versionCode $versionCode/g" build.gradle
```

測試過程其實很多都在搜尋正確的寫法，分段後測試後再組合起來跑！

完成時真是通體舒暢！建議大家都來試試！當然你如果跟我一樣也有這種奇怪的測試需求...LOL