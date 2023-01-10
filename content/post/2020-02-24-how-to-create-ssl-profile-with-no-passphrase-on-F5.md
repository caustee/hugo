---
date: "2020-02-24T00:00:00Z"
title: How to create a SSL profile when you don't know the key passphrase
---
On F5 BigIP LTM you can create a SSL profile even if you don't know the private key passphrase

This workaround can be applied only if you've used the private key on the same F5 BigIP node already. In my case, I had to reuse the same cert+key from a server-ssl profile on a client-ssl profile.
And because business was in a hurry and they didn't have the private key anymore, I had to find alternative ways of making it work.

This can be achieavable by copying the config snippet from `bigip.conf`:
```
ltm profile server-ssl /Common/<name> {
    app-service none
    ca-file /Common/XXXXXXcertXXXXX.crt
    cert /Common/XXXXXXcertXXXXX.crt
    cert-key-chain {
        XXXXXXkeyXXXXX_privkey {
            cert /Common/XXXXXXcertXXXXX.crt
            key /Common/XXXXXXkeyXXXXX_privkey.key
            passphrase <random hash value>
        }
    }
    chain none
    defaults-from /Common/serverssl
    inherit-certkeychain false
    key /Common/XXXXXXkeyXXXXX_privkey.key
    passphrase <different hash value>
    peer-cert-mode require
}
```
And re-added in `bigip.conf` as a client-ssl profile.
```
ltm profile client-ssl /Common/<name> {
    app-service none
    ca-file /Common/XXXXXXcertXXXXX.crt
    cert /Common/XXXXXXcertXXXXX.crt
    cert-key-chain {
        XXXXXXkeyXXXXX_privkey {
            cert /Common/XXXXXXcertXXXXX.crt
            key /Common/XXXXXXkeyXXXXX_privkey.key
            passphrase <random hash value>
        }
    }
    chain none
    defaults-from /Common/clientssl
    inherit-certkeychain false
    key /Common/XXXXXXkeyXXXXX_privkey.key
    passphrase <different hash value>
    peer-cert-mode require
}
```


then verify  the config: `tmsh load sys config verify`. If you don't get errors just drop the `verify` part in order to load the new config: `tmsh load sys config`
These commands work only from linux shell, if you're already in tmsh  just ignore the tmsh part of the command.

And that's it. Make sure to have the same passphrase and to do this on the Standby node. If everything is ok and you see the change in the config, just sync it to the Active peer node.
I've tried to add the lines to a new file and just merge the config but it didn't work as it couldn't get the passphrase to read the key. This was the only option I could find.
Good luck!

