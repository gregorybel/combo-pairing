# Keeping a Backup of the Pairing Data

While switching between branches and upgrading, it may help to keep a backup of the pairing data.
1.  retrieve a list of the installed user-level apks: 
    ```
    $ adb shell pm list packages -3 | grep ruffy
    package:org.monkey.d.ruffy.ruffy
    ```
1. Backup of an apk's data - this does not include the apk itself. 
   (Filename according to personal preference - in this example depending on version)
   ```
   $ adb backup -f monkey.d.ruffy-1.ab org.monkey.d.ruffy.ruffy
   $ adb backup -f monkey.d.ruffy-2.ab org.monkey.d.ruffy.ruffy-2
   ```
   To restore a backup: `adb restore monkey.d.ruffy-1.ab`
1. That `ab` archive can be checked/unpacked using either specialized tools, or the generic commandline:
   ```
   $ dd if=monkey.d.ruffy-1.ab  bs=24 skip=1 | unpigz | tar -tvf -
   36+1 records in
   36+1 records out
   877 bytes transferred in 0.000095 secs (9242223 bytes/sec)
   -rw-------  0 1000   1000     1000 Jan  1  1970 apps/org.monkey.d.ruffy.ruffy/_manifest
   -rw-rw----  0 10096  10096     441 Feb 11 16:41 apps/org.monkey.d.ruffy.ruffy/sp/pumpdata.xml 
   ```
     1. The first 24 bytes of the file are skipped
     2. Zlib is applied to uncompress the content. If `pigz` isn't installed, there are a few generic alternative ways to achieve this task - just slot into the second part of the pipe (in place of `unpigz`):
       * `python -c "import zlib,sys;sys.stdout.write(zlib.decompress(sys.stdin.read()))"`
       * `perl -MCompress::Zlib -e 'undef $/; print uncompress(<>)'`
       * if `pigz` is installed, `unpigz` (same as `pigz -d`)
       * any other ideas? 
     3. The resulting tar archive can be examined or unpacked with `tar` (use `-x` to unpack)
     
 1. The Bluetooth pairing information can be fetched for safekeeping off-phone. Example from Lineageos 14.1:
    ```
    adb pull /data/misc/bluedroid/bt_config.conf
    ```
1. Same with *AndroidAPS* configuration data 
   * `Export Settings` from within AndroidAPS
   * Fetch the file (I prefer adding yyyy-mm-dd to the filename to keep track of versions. YMMV :) 
     ```
     adb pull /storage/emulated/0/AndroidAPSPreferences AndroidAPSPreferences-2018-02-13
     ```
