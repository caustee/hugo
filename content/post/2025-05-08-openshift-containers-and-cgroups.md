---
date: "2025-05-08T00:00:00Z"
title: "RedHat Openshift, container IDs and cgroups"
---
---

So I had a really interesting issue recently where one of our app devs was complaining that he can't get the container ID from within the container in a new environment.

For context, there's a setup with 3 identical environments on the customer side and a local lab that was recently replaced. I've deployed a new Openshift cluster (version 4.14. will become important later) to replace another lab environment in a different location.

Now, the app admin complained that in all the other envs he was able to run a script to find out the container ID:
```
"$(cat /proc/1/cpuset)" | sed -e 's/^.*-//' | cut -c1-12)
```

Normally this would have returned something like this:
```
sh-4.4$ cat /proc/1/cpuset
/kubepods.slice/kubepods-podd5fb9541_11a3_403a_8dfc_ce3cfdf06023.slice/crio-66d462830ca23be608eb8c8f377485f09126df694f00803396d0c44f03c4bdea.scope
```

This would result into:
```
costin@hp:~$ echo "/kubepods.slice/kubepods-podd5fb9541_11a3_403a_8dfc_ce3cfdf06023.slice/crio-66d462830ca23be608eb8c8f377485f09126df694f00803396d0c44f03c4bdea.scope" | sed -e 's/^.*-//' | cut -c1-12
66d462830ca2
costin@hp:~$
```

Now, in this new setup the result was different:
```
/ # cat /proc/1/cpuset
/
```

After losing many hours trying to understand why cpuset shows a different value using various online sources (blog posts, LLMs) I had to get back to the old fashion way.

It was time to understand what's up with this cpuset, and this started with learning about cgroup and namespaces.

A great resource on this is [this video ](https://www.youtube.com/watch?v=x1npPrzyKfs) from Samuel Karp.

Armed with this I checked [this amazing presentation](https://www.youtube.com/watch?v=sK5i-N34im8) from Jerome Petazzoni.


Then I checked the official kernel documentation for cgroup:
[cgroup v1](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v1/cgroups.html)
[cgroup v2](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html)

After this I was able to understand that my issue comes from using different cgroup versions. The working ones were using cgroup v1 and the non working ones v2.

![why?](/images/ytho.jpg)

And this had to do with the Openshift version. I mentioned earlier that I installed v4.14. This new env was a fresh install, while all previous ones were upgraded from earlier versions to 4.14 at some time.

Checking [official documentatio from RedHat](https://docs.redhat.com/en/documentation/openshift_container_platform/4.14/html/installation_configuration/enabling-cgroup-v1) we see that:
```
As of OpenShift Container Platform 4.14, OpenShift Container Platform uses Linux control group version 2 (cgroup v2) in your cluster. If you are using cgroup v1 on OpenShift Container Platform 4.13 or earlier, migrating to OpenShift Container Platform 4.14 will not automatically update your cgroup configuration to version 2. A fresh installation of OpenShift Container Platform 4.14 will use cgroup v2 by default.
```

Exactly my version. My deployment started with v2 while the old ones had v1 as default and were not upgraded automatically.

So this explains the **WHY**, now all I had to do was find out **HOW**.

And this was as simple as: `oc edit node.config/cluster`

And just populate the spec config with correct version:
```
spec:
  cgroupMode: "v1"
```

After this Openshift started its magic and upgraded all the nodes one by one (an amazing process, btw. Huge props to RedHat team for creating this).

Of course, this was more of a workaround, because downgrading the cgroup version is not a valid option in a fresh install.
Especially since cgroup v1 will be retired soon from Openshift:
```
Important

cgroup v1 is a deprecated feature. Deprecated functionality is still included in OpenShift Container Platform and continues to be supported; however, it will be removed in a future release of this product and is not recommended for new deployments.

For the most recent list of major functionality that has been deprecated or removed within OpenShift Container Platform, refer to the Deprecated and removed features section of the OpenShift Container Platform release notes.
```


But sometimes, in some organizations, you have to take this kind of decisions to unblock the application team while R&D starts working on a permament fix for the application.


