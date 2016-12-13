---
layout: post
title: 'Disk image file (.dmg) from command line'
tags:
- dmg
- cli
- mac
- security
- hdiutil
- keychain
---

I prefer .dmg instaed of zip for archiving project data, etc. `.dmg` is handy for refering files, modify contents without extract files to somewhere.

`.dmg` can usable as like USB drive. Disk Utility tool can create/update `.dmg` from folder with various options. Options are like encryption, readonly, compression, etc.

But if you have tens of folders to archive, it's better to use command line tools.

## Create encrypted `.dmg` file

`hdiutil` is command line version of Disk Utility app. This command can mount/unmount/create/update disk image files. Please see `man hdiutil` for more detail.

Below script is part of my workflow of archiving project files. I'm using encrypted `.dmg` for archive. The script require prepare password file under `$HOME/.dmg-password`. Please create and store password for `.dmg` without LF.

And update permission like `chmod 600 $HOME/.dmg-password` to prevent read from other users. This sequence using password and encryption. But it's not strong enough, reason described below.

```sh
#!/bin/sh

if [ $# -lt 2 ]; then
  echo $0 SRC_DIR DEST_DIR
  exit 1
fi

SRC=$1
DST=$2
PWD="$HOME/.dmg-password"

if [ ! -e $PWD ]; then
  echo Disk Image password not found
  exit 2
fi

for t in "$SRC"/*; do
  if [ -d "$t" ]; then
    echo Creating: $t
    n=$(basename $t)
    cat $PWD |                \
      hdiutil create          \
        -srcfolder "$t"       \
        -fs HFS+              \
        -encryption AES-128   \
        -format UDBZ          \
        -stdinpass            \
        "$DST/$n.dmg"
  fi
done
```

## Preset password for `.dmg` in Key Chain

It's kind of pain in neck entering password for opening `.dmg` everytime. If you open `.dmg` from Finder.app, the password dialog refuse copy & paste operation.

![Password dialog](/images/2016-12-13-dmg1.png)

There is option "remember password in my keychain". Concept is similar to this.

The password for disk image is stored in keychain which identified by UUID of `.dmg`. The UUID is referable by command like below.

```sh
$ hdiutil isencrypted YOURDISKIMAGE.dmg
```

Now you can store password through `security` command.

```sh
$ security add-generic-password -a (UUID above) -D "disk image password" -s (YOUR DISK IMAGE).dmg -w (PASSWORD OF DISK IMAGE)
```

Unfortunatelly, there are no option like `-stdinpass`. So the password must be passed through command line argument. This mean optential leak through `ps` command or shell history.

By the way, I'm using below script for preset password to disk images.

```
#!/bin/sh

if [ $# -lt 1 ]; then
  echo $0 [dmg file]...
  exit 1
fi

PWD=$HOME/.dmg-password

for FILE in "$@"; do
  UUID=$(hdiutil isencrypted "$FILE" 2>&1 | grep uuid | awk '{print $2}')
  BASE=$(basename $FILE)

  echo File: $BASE
  echo UUID: $UUID

  security add-generic-password -a $UUID -D "disk image password" -s $BASE -w $(cat $PWD)
done
```

When opening `.dmg` from Finder, operating system ask authorisation of using password by `diskimages-helper`.

![Confirmation dialog](/images/2016-12-13-dmg2.png)

You can skip this dialog by `-A` option of `security` command, but this option authorise for all applications. It's better not use this `-A` option for better security.
