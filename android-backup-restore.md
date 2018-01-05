Android Phone Backup/Restore
============================

### adb backup

    adb backup -apk -shared -all -f backup-file.adb
    adb backup -apk -shared -all -f backup.ab
    adb install /path/to/file/app.apk
    adb restore backup-file.adb


### FDroid Fetcher

    function f_droid() {
      curl -s https://f-droid.org/packages/$1/ | grep "package-version-download" -A 1 | head -n 2 | tail -n 1 | sed -n 's/.*href="\(.*\)".*/\1/p'
    }

