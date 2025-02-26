# CI/CD

(requires login at https://code.binbash.rocks to see states)

![Gitea build](https://code.binbash.rocks/AXP.OS/packages_apps_phonesky/actions/workflows/build.yml/badge.svg)
![Codeberg sync](https://code.binbash.rocks/AXP.OS/packages_apps_phonesky/actions/workflows/mirror.yaml/badge.svg)
![Github sync](https://code.binbash.rocks/AXP.OS/packages_apps_phonesky/actions/workflows/mirror-github.yaml/badge.svg)

# microG Phonesky (Google Play) patches

While making (in-)app-purchases with microG works out-of-the-box with many Stock ROMs, on after-market ROMs like AOSP or LineageOS security restrictions prevent this (theese ROMs expect the signature of Google Play to be equal to the signature of the Play Services, but since microG has a different signature as Google Play they refuse to allow making purchases, more details at [microG #309](https://github.com/microg/android_packages_apps_GmsCore/issues/309)).

This patches allow the user to create a patched Google Play on their own which resolves that issue, additionally most recent patches also eliminate most update-triggers of Google Play, prevent most of the caused battery and data drain.

Base patches are created by @Nanolx with help by @Vavun for update-blocking hunks.

The patcher itself has been heavily re-worked by steadfasterX to fix several FC's, adding more features, staying compatbible with current play store releases and last but not least adding a ci/cd workflow to automate the whole process.

# Releases

The CI/CD automation process creates [releases](https://code.binbash.rocks/AXP.OS/packages_apps_phonesky/releases) which (when automated) are flagged as "pre-release".

Those builds are set to **pre-release/untested!** by default.

After a manual test process pre-releases may become productive and so the pre-release flag will be removed then.

Most issues which can be caused by the whole process are FC's (force closes) of the play store during usage. When this happens a [logcat](https://forum.xda-developers.com/t/guide-providing-a-good-logcat.3814517/) is needed at the time when the FC happens.

# Patching official Google Play builds

1. get [apktool](https://ibotpeaches.github.io/Apktool/) (using the latest version to-date is recommended)
1. either use [apkmd](https://github.com/tanishqmanuja/apkmirror-downloader/) to download the latest play store or get it manually from [apkmirror](https://www.apkmirror.com/apk/google-inc/google-play-store/) (again, using the latest supported version is recommended)
1. optional: get [Apk Sign](https://github.com/appium/sign/raw/master/dist/signapk.jar), if you want to sign the patched apk (recommended)


## Usage

Use the `patch-playstore` Script provided here.

If you want to sign you can change the

  `keystore=""`

value to the path of your java keystore.

regular usage:

```
./patch-playstore com.android.vending_37.9.18-21_0_PR_571399392-83791810.apk
```

debugging output & options:

```
DEBUG=yes APKTOOL_FLAGS=" --use-aapt2" ./patch-playstore com.android.vending_37.9.18-21_0_PR_571399392-83791810.apk
```

do nothing, just print the resulting apk filename, full playstore version and current flags:
 
```
./patch-playstore printapk com.android.vending_37.9.18-21_0_PR_571399392-83791810.apk
```


# Installing patched Google Play

For Google Play to properly work, it needs to be a system app, prefered way of doing this is from TWRP. This is only required for first time installations, updates of the patched Google Play can be installed by pushing the apk to the device and opening it or via `adb install`.

The typical path for Google Play is:

  `/system/priv-app/Phonesky/Phonesky.apk`

be aware that you'll also need to unpack the libraries matching your device's architecture to:

  `/system/priv-app/Phonesky/lib/${arch}/`

where ${arch} is something like `arm`, or `x86`, you can open apk files like zip files to get the libraries. Once the apk and libraries are in place adjust the owner and file permissions:

```
find /system/priv-app/Phonesky/ -type d 2>/dev/null | while read dir; do
	chown 0:0 ${dir}
	chmod 0755 ${dir}
	chcon 'u:object_r:system_file:s0' ${dir} 2>/dev/null
done

find /system/priv-app/Phonesky/ -type f 2>/dev/null | while read file; do
	chown 0:0 ${file}
	chmod 0644 ${file}
	chcon 'u:object_r:system_file:s0' ${file} 2>/dev/null
done
```
> BEFORE continuing, check your `/system/etc/permissions` folder on your device. If it contains a file named `com.android.vending.xml` then use the respective `com.android.vending.xml` file from `etc/` in this repository. If it instead contains a file named `privapps-permissions-com.android.vending.xml`, then use the respective file located in `etc/` in this repository.

Also grab the files from the `etc/` folder **of this repository** and push them to their respective directories in `/system` so you'll get the following files:

  * /system/etc/permissions/**com.android.vending.xml** or **privapps-permissions-com.android.vending.xml**
  * /system/etc/default-permissions/**phonesky-permissions.xml**

adjust permissions for those files aswell:

```
for file in /system/etc/permissions/com.android.vending.xml /system/etc/default-permissions/phonesky-permissions.xml; do
	chown 0:0 ${file}
	chmod 0644 ${file}
	chcon 'u:object_r:system_file:s0' ${file} 2>/dev/null
done
```

done (finally), reboot into your ROM, grant signature spoofing permission to Google Play and reboot the ROM again. Setup your Google account and feel free to make purchases.

# Trouble Shooting

* some recent builds of Google Play need `--use-aapt2` flag to be passed to `apktool`
  * when using `patch-playstore` script use: `DEBUG=yes APKTOOL_FLAGS=--use-aapt2 patch-playstore ...`
* bootloop after installing patched Google Play: [see this post](https://gitlab.com/Nanolx/microg-phonesky-iap-support/issues/3#note_268785229)
