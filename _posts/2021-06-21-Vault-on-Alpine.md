---
published: true
---
Vault exists in Alpine's apk repo (probably the community one), but with the aim of that version being small, like Alpine, it has no UI.  As I'm only just learning Vault and hoping to get the certification shortly, having and knowing the UI was non-negotiable, so I downloaded the latest x64 Vault binary and overwrote it.

THEN, it wouldn't start ... something about mlock.

I figured out how to disable mlock and it started fine, but then it was bugging me that that is a very not recommended configuration, so I started to do some digging.

Like with most things with Alpine, there is VERY little out there, and most of it has to do with people running it in containers ... fortunately that advice is often directly applicable to running it as a full VM OS.

This wonderful github issue comment here - https://github.com/hashicorp/docker-vault/issues/137#issuecomment-624440266 - explained the following -

---

* ### install libcap

    apk update

    apk install libcap


* ### Set the required capabilities on the binary

    setcap cap_ipc_lock=+ep /usr/sbin/vault
    
---

Then restart vault, and away you go, mlock, presuably, working!
