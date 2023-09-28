---
date: "2023-09-28T00:00:00Z"
title: How to fix "Could not connect to one or more vCenter Server"
---

Recently I had to fix a vCenter Server that was stopped working. The webpage displayed the following error, and the automated backup jobs stopped working.


`Could not connect to one or more vCenter Server systems: https://$VCENTER_FQDN:443/sdk`

The version is rather old and end of life (6.7), so VMware support was out of the question. Which means I had to do it all by myself. Great...

After checking the logs for obvious errors, I've noticed that most services were stopped. And when I started them manually, the command hanged for a while and it returned this error after timeout expired:

```
root@VCENTER [ ~ ]# service-control --start vmware-vpostgres
Operation not cancellable. Please wait for it to finish...
Performing start operation on service vmware-vpostgres...
 
Error executing start on service vmware-vpostgres. Details {
    "detail": [
        {
            "id": "install.ciscommon.service.failstart",
            "translatable": "An error occurred while starting service '%(0)s'",
            "args": [
                "vmware-vpostgres"
            ],
            "localized": "An error occurred while starting service 'vmware-vpostgres'"
        }
    ],
    "componentKey": null,
    "resolution": null,
    "problemId": null
}
Service-control failed. Error: {
    "detail": [
        {
            "id": "install.ciscommon.service.failstart",
            "translatable": "An error occurred while starting service '%(0)s'",
            "args": [
                "vmware-vpostgres"
            ],
            "localized": "An error occurred while starting service 'vmware-vpostgres'"
        }
    ],
    "componentKey": null,
    "resolution": null,
    "problemId": null
}
```

Ok, so the problem came from `vmware-vpostgres` service. I issued again the same command and in a separate shell I ran `ps faux` to see the processes. This showed me the hanging command:

```
vpostgr+ 35914  0.0  0.0  21492  3408 ?        S    09:10   0:00 /bin/bash /opt/vmware/vpostgres/current/scripts/pg_refresh_certs
vpostgr+ 35962  0.0  0.0  73328  6268 ?        S    09:10   0:00  \_ /usr/lib/vmware-vmafd/bin/vecs-cli entry list --store TRUSTED_ROOT_CRLS
```

Looks like part of the starting process for vpostgres is this `pg_refresh_certs` script. I ran this one manually with debug on:

```
root@VCENTER [ ~ ]# /bin/bash  -x /opt/vmware/vpostgres/current/scripts/pg_refresh_certs
+ '[' -z /opt/vmware/vpostgres/current ']'
+ SANITY_FILE=/opt/vmware/vpostgres/current/scripts/vpostgres_sanity_checks
+ '[' -f /opt/vmware/vpostgres/current/scripts/vpostgres_sanity_checks ']'
+ source /opt/vmware/vpostgres/current/scripts/vpostgres_sanity_checks
++ CHECK_STATUS=0
++ '[' -z /opt/vmware/vpostgres/current ']'
++ '[' -z /opt/vmware/vpostgres/current/bin ']'
++ '[' -z /opt/vmware/vpostgres/current/etc ']'
++ '[' -z /opt/vmware/vpostgres/current/scripts ']'
++ '[' -z postgres ']'
++ '[' -z vpostgres ']'
++ '[' -z users ']'
++ '[' -z /storage/db/vpostgres ']'
++ '[' -z /storage/dblog ']'
++ '[' -z /storage/db ']'
++ '[' -z /storage/archive ']'
++ '[' -z /storage/dblog/vpostgres/pg_xlog ']'
++ '[' -z /storage/archive/vpostgres ']'
++ '[' -z /storage/archive/backup_data ']'
++ '[' -z /var/log/vmware/vpostgres ']'
++ '[' 0 == 1 ']'
+ '[' -z /usr/lib ']'
+ EXPECTED_ARGS=0
+ '[' 0 -ne 0 ']'
+ '[' '!' -d /storage/db/vpostgres_ssl ']'
+ chmod 700 /storage/db/vpostgres_ssl
+ chown vpostgres:users /storage/db/vpostgres_ssl
+ CERT_FILE=/storage/db/vpostgres_ssl/server.crt
+ KEY_FILE=/storage/db/vpostgres_ssl/server.key
+ /usr/lib/vmware-vmafd/bin/vecs-cli entry getkey --store MACHINE_SSL_CERT --alias __MACHINE_CERT --output /storage/db/vpostgres_ssl/server.key
+ ERRNUM=0
+ '[' 0 '!=' 0 ']'
+ /usr/lib/vmware-vmafd/bin/vecs-cli entry getcert --store MACHINE_SSL_CERT --alias __MACHINE_CERT --output /storage/db/vpostgres_ssl/server.crt
+ ERRNUM=0
+ '[' 0 '!=' 0 ']'
+ chmod 600 /storage/db/vpostgres_ssl/server.key /storage/db/vpostgres_ssl/server.crt
+ chown vpostgres:users /storage/db/vpostgres_ssl/server.key /storage/db/vpostgres_ssl/server.crt
+ CA_FILE=/storage/db/vpostgres_ssl/root_ca.pem
+ CRL_FILE=/storage/db/vpostgres_ssl/root_crl.pem
+ print_full_store TRUSTED_ROOTS /storage/db/vpostgres_ssl/root_ca.pem
+ STORE_NAME=TRUSTED_ROOTS
+ OUTPUT_FILE=/storage/db/vpostgres_ssl/root_ca.pem
+ OUTPUT_TMP=/storage/db/vpostgres_ssl/root_ca.pem.tmp
+ ALIAS_TMP=/storage/db/vpostgres_ssl/root_ca.pem.alias
+ /usr/lib/vmware-vmafd/bin/vecs-cli entry list --store TRUSTED_ROOTS
+ grep Alias
+ '[' 0 '!=' 0 ']'
+ IFS=
+ read -r entry
+ alias=38c489f6004a9b188aa691e70728fbf8bbbf8e7e
+ /usr/lib/vmware-vmafd/bin/vecs-cli entry getcert --store TRUSTED_ROOTS --alias 38c489f6004a9b188aa691e70728fbf8bbbf8e7e
+ '[' 0 '!=' 0 ']'
+ IFS=
+ read -r entry
+ alias=b73f46482e7a7deb8ad16bcc24a9a7354b60efdc
+ /usr/lib/vmware-vmafd/bin/vecs-cli entry getcert --store TRUSTED_ROOTS --alias b73f46482e7a7deb8ad16bcc24a9a7354b60efdc
+ '[' 0 '!=' 0 ']'
+ IFS=
+ read -r entry
+ alias=f6434495e555a68eb7a142ed847206cf82b93efe
+ /usr/lib/vmware-vmafd/bin/vecs-cli entry getcert --store TRUSTED_ROOTS --alias f6434495e555a68eb7a142ed847206cf82b93efe
+ '[' 0 '!=' 0 ']'
+ IFS=
+ read -r entry
+ alias=079432ecb4450b2b9a9bdd01faf1d4ddc2b71dad
+ /usr/lib/vmware-vmafd/bin/vecs-cli entry getcert --store TRUSTED_ROOTS --alias 079432ecb4450b2b9a9bdd01faf1d4ddc2b71dad
+ '[' 0 '!=' 0 ']'
+ IFS=
+ read -r entry
+ mv /storage/db/vpostgres_ssl/root_ca.pem.tmp /storage/db/vpostgres_ssl/root_ca.pem
+ rm /storage/db/vpostgres_ssl/root_ca.pem.alias
+ chmod 600 /storage/db/vpostgres_ssl/root_ca.pem
+ chown vpostgres:users /storage/db/vpostgres_ssl/root_ca.pem
+ print_full_store TRUSTED_ROOT_CRLS /storage/db/vpostgres_ssl/root_crl.pem
+ STORE_NAME=TRUSTED_ROOT_CRLS
+ OUTPUT_FILE=/storage/db/vpostgres_ssl/root_crl.pem
+ OUTPUT_TMP=/storage/db/vpostgres_ssl/root_crl.pem.tmp
+ ALIAS_TMP=/storage/db/vpostgres_ssl/root_crl.pem.alias
+ grep Alias
+ /usr/lib/vmware-vmafd/bin/vecs-cli entry list --store TRUSTED_ROOT_CRLS
 
 
```


So this is the command that hangs! I ran it manually and timed it:

```
root@VCENTER [ /storage/db/vpostgres_ssl ]# time /usr/lib/vmware-vmafd/bin/vecs-cli entry list --store TRUSTED_ROOT_CRLS
real    56m52.011s
user    0m0.444s
sys     0m0.600s
```

And here we go. The TRUSTED_ROOT_CRLS store was so long that the entry list took 56 minutes. Just to list them! For fun I've checked the number of lines, and /storage/db/vpostgres_ssl/root_crl.pem had more than 100.000 lines.

Luckily I'm not a special case, and [VMware already had a resource online](https://kb.vmware.com/s/article/59555) which included a fix. Granted, the name and service were different, but the root case was the same.

So I did the same. Created the `crl-fix.sh` script:
```
#!/bin/bash
cd /etc/ssl/certs
mkdir /tmp/pems
mkdir /tmp/OLD-CRLS-CAs
mv *.pem /tmp/pems && mv *.* /tmp/OLD-CRLS-CAs
h=$(/usr/lib/vmware-vmafd/bin/vecs-cli entry list --store TRUSTED_ROOT_CRLS --text | grep Alias | cut -d : -f 2)
for hh in "echo "${h[@]}"";do echo "Y" | /usr/lib/vmware-vmafd/bin/vecs-cli entry delete --store TRUSTED_ROOT_CRLS --alias $hh;done
mv /tmp/pems/* .
for l in `ls *.pem`;do ln -s $l ${l/pem/0};done
service-control --stop vmafdd && service-control --start vmafdd
```

And ran it. Now, because we know the list entries command takes almost 1 hour, we should be patient. After around 1 hour the script finished successfully. I ran the store entry list command again and this time there were only 2 certs in TRUSTED_ROOT_CRLS store. So the script was successful.

I've rebooted the VCSA and then checked the services again. Most of them were stopped. I've started them manually:

```
root@VCENTER [ ~ ]# service-control --start --all
Operation not cancellable. Please wait for it to finish...
Performing start operation on service lwsmd...
Successfully started service lwsmd
Performing start operation on service vmafdd...
Successfully started service vmafdd
Performing start operation on service vmdird...
Successfully started service vmdird
Performing start operation on service vmcad...
Successfully started service vmcad
Performing start operation on service vmware-sts-idmd...
Successfully started service vmware-sts-idmd
Performing start operation on service vmware-stsd...
Successfully started service vmware-stsd
Performing start operation on service vmdnsd...
Successfully started service vmdnsd
Performing start operation on profile: ALL...
Service-control failed. Error: Failed to start services in profile ALL. RC=5, stderr=Failed to start eam, vsphere-client, vsphere-ui, vpxd-svcs services. Error: Operation not allowed in current service state
```

![wtf](https://media.tenor.com/Yy8d1d0oUAEAAAAC/wtf.gif)

After I questioned the script and my life decisions, I've checked again the status of services.

```
root@VCENTER [ ~ ]# service-control --status --all
StartPending:
vmware-content-library vmware-perfcharts
Stopped:
vmcam vmware-imagebuilder vmware-mbcs vmware-netdumper vmware-rbd-watchdog vmware-vcha vsan-dps
Running:
applmgmt lwsmd pschealth vmafdd vmcad vmdird vmdnsd vmonapi vmware-analytics vmware-certificatemanagement vmware-cis-license vmware-cm vmware-eam vmware-pod vmware-postgres-archiver vmware-rhttpproxy vmware-sca vmware-sps vmware-statsmonitor vmware-sts-idmd vmware-stsd vmware-topologysvc vmware-updatemgr vmware-vapi-endpoint vmware-vmon vmware-vpostgres vmware-vpxd vmware-vpxd-svcs vmware-vsan-health vmware-vsm vsphere-client vsphere-ui
root@VCENTER [ ~ ]#
```

![surprised pikachu](https://en.meming.world/images/en/thumb/2/2c/Surprised_Pikachu_HD.jpg/300px-Surprised_Pikachu_HD.jpg)

And here it was. All services started automatically, I just had to be patient and wait :)

The web interface was back now. Anyway, thanks a bunch unnamed article author for this script.

