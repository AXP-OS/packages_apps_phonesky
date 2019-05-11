# microG Phonesky (Google Play) patches

While making (in-)app-purchases with microG works out-of-the-box with many Stock ROMs, on after-market ROMs like AOSP or LineageOS security restrictions prevent this (theese ROMs expect the signature of Google Play to be equal to the signature of the Play Services, but since microG has a different signature as Google Play they refuse to allow making purchases, more details at [microG #309](https://github.com/microg/android_packages_apps_GmsCore/issues/309)).

This patches allow the user to create a patched Google Play on their own which resolves that issue, additionally most recent patches also eliminate most update-triggers of Google Play, prevent most of the caused battery and data drain.

Base patches are created by @Nanolx with help by @Vavun for update-blocking hunks.

# Patching official Google Play builds

First get [apktool](https://ibotpeaches.github.io/Apktool/) (using the latest version to-date is recommended), then head over to [apkmirror](https://www.apkmirror.com/apk/google-inc/google-play-store/) to download the desired Google Play apk (again, using the latest supported version is recommended). Also get [Apk Sign](https://github.com/appium/sign/raw/master/dist/signapk.jar), along the required [testkey](https://github.com/appium/sign/raw/master/testkey.pk8), [testcertificate](https://raw.githubusercontent.com/appium/sign/master/testkey.x509.pem) to sign your custom apk with a test key.

For reference we assume you downloaded the Google Play apk for version 14.6.56 and named it Phonesky-14.6.56.apk, along the corresponding patch named Phonesky-14.6.56-microG.diff, commands provided are for unixoid operating systems, if you're using Windows or macOS you may need to adjust them.

* unpack the apk using apktool:

  `java -jar apktool.jar d Phonesky-14.6.56.apk`

* go into the unpacked apk:

  `cd Phonesky-14.6.56`

* clean-up possibly left-over `APKTOOL_DUMMY` entries:

  `find . -type f -name '*.xml' | xargs sed '/APKTOOL_DUMMY/d' -i`

* apply the patch:

  `patch -Np1 -i Phonesky-14.6.56-microG.diff`

  if all patches applied successfully, continue with the next steps. If not, check if you've got the correct patch version, if so and it still fails post a bug report. Future versions of Google Play will eventually be supported, older version won't.

* clean-up possibly left-over .rej or .orig files:

  `find . -type f -name '*.orig' -o -name '*.rej' | xargs rm -f`

* create the new, patched apk:

  `java -jar apktool.jar b .`

* sing the new apk using a test key:

  `java -jar signapk.jar testkey.x509.pem testkey.pk8 dist/Phonesky-14.6.56 dist/Phonesky-14.6.56-signed.apk`

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

Also grab the files from the `etc/` directory and push the in the same directories inside your `/system`, so you'll get the following files:

  * /system/etc/permissions/com.android.vending.xml
  * /system/etc/default-permissions/phonesky-permissions.xml

adjust permissions for those files aswell:

```
for file in /system/etc/permissions/com.android.vending.xml /system/etc/default-permissions/phonesky-permissions.xml; do
	chown 0:0 ${file}
	chmod 0644 ${file}
	chcon 'u:object_r:system_file:s0' ${file} 2>/dev/null
done
```

done (finally), reboot into your ROM, grant signature spoofing permission to Google Play and reboot the ROM again. Setup your Google account and feel free to make purchases.

# Patch compatibility

Patches can be used on other Google Play versions if only the micro version changed, for example you can use the patch for version 14.4.20 on version 14.4.22 aswell, but you can't use the patch for version 14.4.20 for version 14.5.52.
