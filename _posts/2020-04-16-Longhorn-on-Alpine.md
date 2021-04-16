Alpine is not officially supported for Longhorn, but after digging through some old issues on GitHub and getting pointers to some others from the awesome folks on the official Slack, it wasn't hard to get working at all.

I figured I'd document it since, in all my searching at least, this information does not seem to be all in one place anywhere.

---
enable rshared -

mount --make-rshared /

Not entirely clear what it is, or why it's needed, but if you try to install without it, it will tell you it needs it.

---
enable local (or you can write a proper init script if you'd prefer) -

rc-update add local default

---
put script in local with .start extension

k3snode-a1:~# cat /etc/local.d/rshared.start
#!/bin/bash
mount --make-rshared /
k3snode-a1:~#

The .start extension is the important part here.  That was fun to troubleshoot.
Obbiously if you're using a bash script like this, you need bash - apk update && apk install bash

---
install iscsi tools -

The next thing it will complain about if you get this far is the lack of iscsi support and an iscsiadm binary.

apk update && apk add open-iscsi

---
install the other stuff! -

Even then it doesn't work, and it's a whole lot less clear why!  There is where one of the kind members of the Longhorn team helped me out and pointed me to an issue that covered all the remaining problems -

apk add util-linux && rc-update add iscsid boot && /etc/init.d/iscsid start

another one explained - "Longhorn relies on command lsblk in util-linux to collect the disk info on each node."

and the last two parts of that should be obvious - installing open-iscsi doesn't actually *start* or *enable* iscsid ... have to do that manually

---
(optional) mount a dedicated volume to /var/lib/longhorn

If you want to use dedicated space for longhorn, mount the volume to /var/lib/longhorn

---
(optional) if using a dedicate volume, redo rshared part

If you do that, don't forget to enable rshared on that mount as well -

mount --make-rshared /var/lib/longhorn/

and add it to your start script -

echo "mount --make-rshared /var/lib/longhorn/" >> /etc/local.d/rshared.start

---
Then follow the instructions to install and test Longhorn, and you should be good to go!

(I used these, but it looks like there are actually better ones out there that may have saved me some of my digging ... c'est la vie)

https://rancher.com/docs/k3s/latest/en/storage/

("Setting up Longhorn", about half way down the page)


Happy trails!
