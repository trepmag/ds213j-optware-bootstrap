ds213j-optware-bootstrap (manual)
========================

Synology ds213j optware manual bootstrap guide.

Introduction
------------

Actually there's no xsh bootstrap for the ds213j (Marvell Armada 370 ARMv7l) although the existing Marvell Kirkwood mv6281 binaries "are ~ compatible" (http://ipkg.nslu2-linux.org/feeds/optware/cs08q1armel/). So this is a small guide to setup manually the optware environment (ipkg, PATH and init scripts).

### Note on compatibility
It appear that binaries aren't that much/all compatible. The reason must be that the cs08q1armel toolchain doesn't have updated materials. 
If you encounter issues you should consider setup a chroot environment instead.

#### Note on Debian chroot alternative
You must use the --arch=armel deboostrap option and not armhf! This is because the ARMv7l doesn't have the thumb-2 feature which is required by the armhf architecture (reference: https://wiki.debian.org/ArmHardFloatPort#Minimum_CPU_.26_FPU)

Create optware root directory
------------
<pre>
$ mkdir /volume1/@optware
$ mkdir /opt
$ mount -o bind /volume1/@optware /opt
</pre>

Setup ipkg
------------
<pre>
$ feed=http://ipkg.nslu2-linux.org/feeds/optware/cs08q1armel/cross/unstable
$ ipk_name=`wget -qO- $feed/Packages | awk '/^Filename: ipkg-opt/ {print $2}'`
$ wget $feed/$ipk_name
$ tar -xOvzf $ipk_name ./data.tar.gz | tar -C / -xzvf -
$ mkdir -p /opt/etc/ipkg
$ echo "src cross $feed" > /opt/etc/ipkg/feeds.conf
</pre>
(source http://www.nslu2-linux.org/wiki/Optware/HomePage)

Set PATH
-------------------
Add the following line to /etc/profile:
<pre>
PATH=/opt/bin:/opt/sbin:$PATH
</pre>

Create init scripts
-------------------
The following steps will allow to automatically bind the /volume1/@optware directory to /opt and trigger the /opt/etc/init.d/* scripts.

Create the /etc/rc.local file (chmod 755) and insert:
<pre>
#!/bin/sh

# Optware setup
[ -x /etc/rc.optware ] && /etc/rc.optware start
</pre>

Create the /etc/rc.optware file (chmod 755) and insert:
<pre>
#! /bin/sh

if test -z "${REAL_OPT_DIR}"; then
# next line to be replaced according to OPTWARE_TARGET
REAL_OPT_DIR=/volume1/@optware
fi

case "$1" in
    start)
        echo "Starting Optware."
        if test -n "${REAL_OPT_DIR}"; then
            if ! grep ' /opt ' /proc/mounts >/dev/null 2>&1 ; then
                mkdir -p /opt
                mount -o bind ${REAL_OPT_DIR} /opt
            fi	
        fi
	[ -x /opt/etc/rc.optware ] && /opt/etc/rc.optware
    ;;
    reconfig)
	true
    ;;
    stop)
        echo "Shutting down Optware."
	true
    ;;
    *)
        echo "Usage: $0 {start|stop|reconfig}"
        exit 1
esac

exit 0
</pre>
(source: a working optware env)
