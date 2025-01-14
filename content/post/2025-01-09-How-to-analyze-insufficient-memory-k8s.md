---
date: "2025-01-09T00:00:00Z"
title: How to analyze and fix a pod scheduling issue due to Insufficient Memory on Kubernetes or Openshift.
---

So it may happen that you have a Kubernetes or Openshift deployment and a pod does not want to start and remains in Pending state.

First, let's check the events that are causing this:
```
[root@server ~]# k get events -n <namespace>
LAST SEEN   TYPE      REASON             OBJECT            MESSAGE
31m         Warning   FailedScheduling   <namespace>/<pod_name>   0/13 nodes are available: 10 Insufficient memory, 3 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate, 4 Insufficient cpu.
```

As we can see, the reason is FailedScheduling and the message complains about insufficient memory and insufficient cpu.

Now let's check the pod definition to see how much resources it needs:

```
[root@server ~]# k get pods -n <namespace> <pod_name> -o yaml
apiVersion: v1
kind: Pod
metadata:
  <snip>
spec:
  containers:
    <snip>
    resources:
      limits:
        cpu: "8"
        memory: 40Gi
      requests:
        cpu: "2"
        memory: 20Gi
    <snip>
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2025-01-08T12:16:57Z"
    message: '0/13 nodes are available: 10 Insufficient memory, 3 node(s) had taint
      {node-role.kubernetes.io/master: }, that the pod didn''t tolerate, 4 Insufficient
      cpu.'
    reason: Unschedulable
    status: "False"
    type: PodScheduled
  phase: Pending
  qosClass: Burstable
```

As we can see, the pod requests a minimum of 20Gi memory.

Let's check the nodes's resources usage:
```
[root@server ~]# kubectl top node
NAME                           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
master0.ocp.cluster   2026m        27%    8307Mi          55%
master1.ocp.cluster   2779m        37%    14591Mi         50%
master2.ocp.cluster   1412m        18%    7704Mi          51%
worker0.ocp.cluster   1247m        8%     18348Mi         38%
worker1.ocp.cluster   1303m        8%     24336Mi         51%
worker2.ocp.cluster   2117m        13%    22087Mi         46%
worker3.ocp.cluster   2437m        15%    29750Mi         63%
worker4.ocp.cluster   1113m        7%     16064Mi         34%
worker5.ocp.cluster   854m         5%     15732Mi         38%
worker6.ocp.cluster   933m         6%     17811Mi         43%
worker7.ocp.cluster   1068m        6%     22274Mi         47%
worker8.ocp.cluster   608m         3%     18546Mi         39%
worker9.ocp.cluster   1789m        11%    28650Mi         60%
```

Even though the memory is at 38-39% for some workers (out of 48Gi), the pod still can't be scheduled.
Let's check the total available memory of the nodes:
```
[root@server ~]# kubectl describe nodes | grep -A 5 Allocatable
Allocatable:
  cpu:                7500m
  ephemeral-storage:  96143180846
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             15257520Ki
--
Allocatable:
  cpu:                7500m
  ephemeral-storage:  96143180846
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             29708204Ki
--
Allocatable:
  cpu:                7500m
  ephemeral-storage:  96143180846
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             15257512Ki
--
Allocatable:
  cpu:                15500m
  ephemeral-storage:  96143180846
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             48285912Ki
--
Allocatable:
  cpu:                15500m
  ephemeral-storage:  96143180846
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             48285912Ki
--
Allocatable:
  cpu:                15500m
  ephemeral-storage:  96143180846
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             48285912Ki
--
Allocatable:
  cpu:                15500m
  ephemeral-storage:  96143180846
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             48285912Ki
--
Allocatable:
  cpu:                15500m
  ephemeral-storage:  96143180846
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             48285912Ki
--
Allocatable:
  cpu:                15500m
  ephemeral-storage:  96143180846
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             42092768Ki
--
Allocatable:
  cpu:                15500m
  ephemeral-storage:  96143180846
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             42092768Ki
--
Allocatable:
  cpu:                15500m
  ephemeral-storage:  96143180846
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             48285912Ki
--
Allocatable:
  cpu:                15500m
  ephemeral-storage:  96143180846
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             48285912Ki
--
Allocatable:
  cpu:                15500m
  ephemeral-storage:  96143180846
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             48285904Ki
```

And let's check the allocated resources:
```
[root@server ~]# kubectl describe nodes | grep -A 8 "Allocated resources"
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests      Limits
  --------           --------      ------
  cpu                1589m (21%)   0 (0%)
  memory             6542Mi (43%)  0 (0%)
  ephemeral-storage  0 (0%)        0 (0%)
  hugepages-1Gi      0 (0%)        0 (0%)
  hugepages-2Mi      0 (0%)        0 (0%)
--
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests      Limits
  --------           --------      ------
  cpu                2106m (28%)   0 (0%)
  memory             9467Mi (32%)  0 (0%)
  ephemeral-storage  0 (0%)        0 (0%)
  hugepages-1Gi      0 (0%)        0 (0%)
  hugepages-2Mi      0 (0%)        0 (0%)
--
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests      Limits
  --------           --------      ------
  cpu                1549m (20%)   0 (0%)
  memory             6322Mi (42%)  0 (0%)
  ephemeral-storage  0 (0%)        0 (0%)
  hugepages-1Gi      0 (0%)        0 (0%)
  hugepages-2Mi      0 (0%)        0 (0%)
--
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests       Limits
  --------           --------       ------
  cpu                14588m (94%)   22748m (146%)
  memory             39863Mi (84%)  58048Mi (123%)
  ephemeral-storage  0 (0%)         0 (0%)
  hugepages-1Gi      0 (0%)         0 (0%)
  hugepages-2Mi      0 (0%)         0 (0%)
--
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests       Limits
  --------           --------       ------
  cpu                14494m (93%)   23125m (149%)
  memory             45686Mi (96%)  58712Mi (124%)
  ephemeral-storage  0 (0%)         0 (0%)
  hugepages-1Gi      0 (0%)         0 (0%)
  hugepages-2Mi      0 (0%)         0 (0%)
--
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests       Limits
  --------           --------       ------
  cpu                14678m (94%)   29300m (189%)
  memory             45826Mi (97%)  55412Mi (117%)
  ephemeral-storage  0 (0%)         0 (0%)
  hugepages-1Gi      0 (0%)         0 (0%)
  hugepages-2Mi      0 (0%)         0 (0%)
--
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests       Limits
  --------           --------       ------
  cpu                13125m (84%)   22650m (146%)
  memory             44494Mi (94%)  58228Mi (123%)
  ephemeral-storage  0 (0%)         0 (0%)
  hugepages-1Gi      0 (0%)         0 (0%)
  hugepages-2Mi      0 (0%)         0 (0%)
--
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests       Limits
  --------           --------       ------
  cpu                9644m (62%)    13 (83%)
  memory             42188Mi (89%)  43Gi (93%)
  ephemeral-storage  0 (0%)         0 (0%)
  hugepages-1Gi      0 (0%)         0 (0%)
  hugepages-2Mi      0 (0%)         0 (0%)
--
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests           Limits
  --------           --------           ------
  cpu                10194m (65%)       20025m (129%)
  memory             37941332480 (88%)  46284133Ki (109%)
  ephemeral-storage  0 (0%)             0 (0%)
  hugepages-1Gi      0 (0%)             0 (0%)
  hugepages-2Mi      0 (0%)             0 (0%)
--
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests       Limits
  --------           --------       ------
  cpu                13664m (88%)   23600m (152%)
  memory             37099Mi (90%)  51379540992 (119%)
  ephemeral-storage  0 (0%)         0 (0%)
  hugepages-1Gi      0 (0%)         0 (0%)
  hugepages-2Mi      0 (0%)         0 (0%)
--
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests       Limits
  --------           --------       ------
  cpu                12619m (81%)   28575m (184%)
  memory             41204Mi (87%)  52079648256 (105%)
  ephemeral-storage  0 (0%)         0 (0%)
  hugepages-1Gi      0 (0%)         0 (0%)
  hugepages-2Mi      0 (0%)         0 (0%)
--
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests       Limits
  --------           --------       ------
  cpu                12704m (81%)   21450m (138%)
  memory             46078Mi (97%)  46848Mi (99%)
  ephemeral-storage  0 (0%)         0 (0%)
  hugepages-1Gi      0 (0%)         0 (0%)
  hugepages-2Mi      0 (0%)         0 (0%)
--
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests       Limits
  --------           --------       ------
  cpu                12804m (82%)   25300m (163%)
  memory             46334Mi (98%)  58880Mi (124%)
  ephemeral-storage  0 (0%)         0 (0%)
  hugepages-1Gi      0 (0%)         0 (0%)
  hugepages-2Mi      0 (0%)         0 (0%)
```

Now we can see clearly that the requests percentage used is at least 84% of the 48Gi (total available memory for worker nodes), which is less than 20Gi requested by the pod. Sure, we also have one master node that has enough memory to accomodate this, but because of the taint it is not allowed to be scheduled on it. And we'd like to keep it that way.

There are 2 fixes here. The best one is to increase the physical memory available on the cluster. Either by adding another worker or increasing the available memory on the current ones. 
And the short-term workaround, which is to enable overcommitment on the cluster. This, of course, should not be done in Production environment in order to avoid OOM issues.

If you want to set overcommitment on Openshift you can use [this guide](https://docs.openshift.com/container-platform/4.17/nodes/clusters/nodes-cluster-overcommit.html).



