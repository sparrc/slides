<!--
theme: gaia
class: lead
-->

# cgroups v2

---

### Intro

- cgroups are a feature provided by the linux kernel to manage, restrict, and audit groups of processes.
- all linux containers are built on top of cgroups.
- `/sys/fs/cgroup` is usually the cgroup root.


<!--
Notes

-->

---

### Intro

```
$ docker run -d public.ecr.aws/docker/library/busybox:latest sleep 9999
7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3

$ find /sys/fs/cgroup | grep 7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/memory/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/memory/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3/cgroup.procs
/sys/fs/cgroup/memory/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3/memory.use_hierarchy
/sys/fs/cgroup/memory/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3/memory.kmem.tcp.usage_in_bytes
/sys/fs/cgroup/memory/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3/memory.soft_limit_in_bytes
/sys/fs/cgroup/memory/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3/memory.force_empty
/sys/fs/cgroup/memory/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3/memory.pressure_level
/sys/fs/cgroup/memory/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3/memory.move_charge_at_immigrate
/sys/fs/cgroup/memory/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3/memory.max_usage_in_bytes
...
```

<!--
Notes
- every launched docker container creates it's own cgroup
- note docker ID
-->

---

### Intro

- Amazon Linux 2022, Fedora, and Ubuntu all now use cgroups v2 by default.
- ECS agent added support for cgroups v2 in March 2022: https://github.com/aws/amazon-ecs-agent/pull/3127

---

### cgroups v1: structure

- cgroups v1 has a structure where a process's controls are split across many parent directories (memory, blkio, cpu, etc.):
```
/sys/fs/cgroup/devices/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/freezer/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/memory/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/hugetlb/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/blkio/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/perf_event/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/net_cls,net_prio/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/pids/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/cpu,cpuacct/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/cpuset/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/systemd/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
```

<!--
Notes
- cgroup root, then 11 parent directories, then docker container cgroup dirs
-->

---

### cgroups v2: unified structure

- cgroups v2 has a "unified" structure, in which all of the controls for a process are unified in a single directory:
```
/sys/fs/cgroup/system.slice/docker-e883b4e9fbc445677c65ce9325a2ec5548c30b2709c02141da64b7992c430afe.scope
```

<!--
Notes
- single directory contains control files for ALL resources!
-->

---

v1
```
/sys/fs/cgroup/devices/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/freezer/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/memory/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/hugetlb/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/blkio/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/perf_event/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/net_cls,net_prio/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/pids/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/cpu,cpuacct/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/cpuset/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
/sys/fs/cgroup/systemd/docker/7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
```

v2
```
/sys/fs/cgroup/system.slice/docker-e883b4e9fbc445677c65ce9325a2ec5548c30b2709c02141da64b7992c430afe.scope
```

<!--
Notes
- fewer directories makes it easier to grok, and is why cgroups v2 is also referred to as "unified cgroups"
-->

---

### cgroups v1: ECS task limits

- ECS tasks with resource limits have their own cgroup created, under which all containers of that task are created.
- task ID `e39cda25f9824fe9b035e2f57e81c793` has it's own directory within the `memory` control.
- this task has a 450MB limit (471859200 bytes)
```
$ cat /sys/fs/cgroup/memory/ecs/e39cda25f9824fe9b035e2f57e81c793/memory.limit_in_bytes
471859200
```

<!--
Notes
- v1
- memory/ecs/TASK_ID is the path for all memory limits
- cpu/ecs/TASK_ID is the path for all cpu limits
-->

---

### cgroups v2: ECS task limits

- task ID `e95a40efbde84cbc8600660a109ec194` has it's own systemd slice (more on this later).
- this task has a 450MB limit (471859200 bytes)
```
$ cat /sys/fs/cgroup/ecstasks.slice/ecstasks-e95a40efbde84cbc8600660a109ec194.slice/memory.max
471859200
```

<!--
Notes
- v2
- ecstasks.slice/ecstasks-TASK_ID.slice/memory.* for all memory limits
- ecstasks.slice/ecstasks-TASK_ID.slice/cpu.* for all cpu limits
-->

---

### v1 vs. v2 task limits

v1
```
/sys/fs/cgroup/cpuset/ecs/c71f1a23ddec49b3b1921c06324a1a3a
/sys/fs/cgroup/cpuset/ecs/c71f1a23ddec49b3b1921c06324a1a3a/ba73fa85bf400117a7c47894d87b8982410cedd86bb39797a282b9571e8a0131
/sys/fs/cgroup/cpuset/ecs/c71f1a23ddec49b3b1921c06324a1a3a/2ed9902425c6cbc46c22c404645b8ab3e81842f99215f9e6d4f28461d4d4d93c
/sys/fs/cgroup/memory/ecs/c71f1a23ddec49b3b1921c06324a1a3a
/sys/fs/cgroup/memory/ecs/c71f1a23ddec49b3b1921c06324a1a3a/ba73fa85bf400117a7c47894d87b8982410cedd86bb39797a282b9571e8a0131
/sys/fs/cgroup/memory/ecs/c71f1a23ddec49b3b1921c06324a1a3a/2ed9902425c6cbc46c22c404645b8ab3e81842f99215f9e6d4f28461d4d4d93c
/sys/fs/cgroup/cpu,cpuacct/ecs/c71f1a23ddec49b3b1921c06324a1a3a
/sys/fs/cgroup/cpu,cpuacct/ecs/c71f1a23ddec49b3b1921c06324a1a3a/ba73fa85bf400117a7c47894d87b8982410cedd86bb39797a282b9571e8a0131
/sys/fs/cgroup/cpu,cpuacct/ecs/c71f1a23ddec49b3b1921c06324a1a3a/2ed9902425c6cbc46c22c404645b8ab3e81842f99215f9e6d4f28461d4d4d93c
```

v2
```
/sys/fs/cgroup/ecstasks.slice/ecstasks-2296a47d90b74991a428649ff2474cca.slice
/sys/fs/cgroup/ecstasks.slice/ecstasks-2296a47d90b74991a428649ff2474cca.slice/docker-6f5f11820ddac53cab1b097072e5bbf9a5ea147690531985799f9bf8f60ceee9.scope
/sys/fs/cgroup/ecstasks.slice/ecstasks-2296a47d90b74991a428649ff2474cca.slice/docker-34c7e3694bffd7543300456c77be7d4533414aef8ec685792b6ce65bfb899bcc.scope
```

<!--
Notes
-->

---

### cgroup drivers

- **cgroup drivers**: cgroupfs, systemd
- **cgroup versions**: v1, v2
- they are technically independent, but, in practice:
- cgroup v1 <=> cgroupfs
- cgroup v2 <=> systemd
- docker and kubernetes both recommend using the systemd driver. [4] [5]

<!--
Notes
- the two cgroup versions are indepedent of cgroup drivers, but in practice v2 is very much
  tied to the systemd driver because of docker and kubernetes.
-->

---

### cgroupfs driver

- the cgroupfs driver effectively allows users to create cgroups by creating files and directories in `/sys/fs/cgroup`

this creates a cgroup for ecs task `e39cda25f9824fe9b035e2f57e81c793`:
```
$ mkdir -p /sys/fs/cgroup/memory/ecs/e39cda25f9824fe9b035e2f57e81c793
```

<!--
Notes

-->

---

### systemd driver

- All containers and tasks are managed as a systemd unit. [1]
- **note:** they are systemd slices and scopes, not services. [2] [3]
- systemd slices can be created with files (like services can), but scopes cannot [7]

<!--
Notes
- The three systemd unit types to keep in mind: slice, service, scope.
- Slices are usually the parents with multiple services and scopes underneath them.
- Tasks are systemd slices.
- Containers are systemd scopes.
-->

---

### systemd driver

- **Slice** units are generally used as parents for managing scopes and services. **scope** and **service** units are assigned to slices. [2]
- **Service** units manage and _supervise_ processes. They can spawn, kill, and restart processes on their own. [8]
- **Scope** units manage processes but do _not_ supervise. Unlike service units, scope units manage externally created processes, and do not fork off processes on their own. [3]
```
/sys/fs/cgroup/ecstasks.slice                                                               <-- Parent slice for ECS tasks w/ resource limits
└── ecstasks-14e4b4156f6b47d18e1ff3cc55a47bc7.slice                                         <-- Task parent slice
    ├── docker-5a84a635371cf7f2255700c02a0dde027b62515db33cb14934ab62b7edfc27cf.scope       <-- Task container A scope
    └── docker-e883b4e9fbc445677c65ce9325a2ec5548c30b2709c02141da64b7992c430afe.scope       <-- Task container B scope
```

<!--
Notes
- Tasks are systemd slices.
- Containers are systemd scopes.
-->

---

### systemd driver

- A little more context on systemd slices/services/scopes, most systemd service units are part of the `system.slice` systemd slice.

```
$ sudo systemctl status ecs
● ecs.service - Amazon Elastic Container Service - container agent
     Loaded: loaded (/usr/lib/systemd/system/ecs.service; enabled; vendor preset: disabled)
     Active: active (running) since Mon 2022-04-18 20:38:24 UTC; 10s ago
       Docs: https://aws.amazon.com/documentation/ecs/
   Main PID: 6035 (amazon-ecs-init)
      Tasks: 6 (limit: 75910)
     Memory: 71.9M
        CPU: 87ms
     CGroup: /system.slice/ecs.service
             └─6035 /usr/libexec/amazon-ecs-init start                                                    .
```

<!--
Notes
- the ecs service runs in the system slice, along with most services on the instance.
-->

---

### systemd driver

- every docker container is it's own systemd scope `docker-$CONTAINER_ID.scope`
```
$ sudo systemctl status docker-e883b4e9fbc445677c65ce9325a2ec5548c30b2709c02141da64b7992c430afe.scope
● docker-e883b4e9fbc445677c65ce9325a2ec5548c30b2709c02141da64b7992c430afe.scope - libcontainer container e883b4e9fbc445677c65ce9325a2ec5548c30b2709c02141da64b7992c430afe
     Loaded: loaded (/run/systemd/transient/docker-e883b4e9fbc445677c65ce9325a2ec5548c30b2709c02141da64b7992c430afe.scope; transient)
  Transient: yes
    Drop-In: /run/systemd/transient/docker-e883b4e9fbc445677c65ce9325a2ec5548c30b2709c02141da64b7992c430afe.scope.d
             └─50-DevicePolicy.conf, 50-DeviceAllow.conf
     Active: active (running) since Fri 2022-04-15 17:33:31 UTC; 4h 1min ago
         IO: 0B read, 0B written
      Tasks: 1 (limit: 75910)
     Memory: 316.0K
        CPU: 20ms
     CGroup: /system.slice/docker-e883b4e9fbc445677c65ce9325a2ec5548c30b2709c02141da64b7992c430afe.scope
             └─8365 sleep 9999999

Apr 15 17:33:31 ip-10-0-0-142.us-west-2.compute.internal systemd[1]: Started libcontainer container e883b4e9fbc445677c65ce9325a2ec5548c30b2709c02141da64b7992c430afe.
```

<!--
Notes
- When using the systemd driver, you can use standard systemctl commands that you're
  familiar with to query for the status of docker containers.
-->

---

### systemd driver

- ECS tasks can be viewed under `ecstasks.slice` and `ecstasks-$TASK_ID.slice`:
```
$ sudo systemctl status ecstasks.slice
● ecstasks.slice - Slice /ecstasks
     Loaded: loaded
     Active: active since Fri 2022-04-15 19:05:21 UTC; 2h 32min ago
      Tasks: 2
     Memory: 796.0K
        CPU: 306ms
     CGroup: /ecstasks.slice
             └─ecstasks-e95a40efbde84cbc8600660a109ec194.slice
               └─docker-c7d42d8ffd0142ea1f7858cb04f05f19b913572a3dadb49ca9d27eb1705090a1.scope
                 ├─20648 bash /helloworld.sh
                 └─40352 sleep 20

Apr 15 19:05:21 ip-10-0-0-142.us-west-2.compute.internal systemd[1]: Created slice Slice /ecstasks.           .
```

<!--
Notes
- You can also use these familiar commands to query info about slices, namely the parent
  ecstasks slice.
-->

---

### systemd driver

when we create a task, ECS agent tells systemd to create a slice

```
$ tail -f /var/log/messages
systemd[1]: Created slice Slice /ecstasks.
systemd[1]: Created slice cgroup ecstasks-0a272520dcbb4340a96121fab054a479.slice                                                    .
```

and docker tells systemd to create a container (and scope)

```
dockerd[3]: level=info msg="Configured log driver..." container=579341a0143cb9256598d0d7c94fcd7fd8947d51c9fccf5a9e1739bc10580cbb driver=awslogs
containerd[2]: level=info msg="starting" namespace=moby path=/.../moby/579341a0143cb9256598d0d7c94fcd7fd8947d51c9fccf5a9e1739bc10580cbb
systemd[1]: Started libcontainer container 579341a0143cb9256598d0d7c94fcd7fd8947d51c9fccf5a9e1739bc10580cbb.
```

<!--
Notes
- With the systemd driver, all cgroup operations are logged by systemd in the system logs.
-->

---

### cgroups v1: OOM-kill behavior

Kernel handles the OOM-kill and docker (tries) to report it to ecs agent through a container event:

```
$ sudo cat /var/log/ecs/ecs-agent.log | grep OOM
level=info time=2022-04-15T23:21:53Z msg="DockerGoClient: process within container 8268d4612267114894856a5d12d316fa1bf65da2045e3481a0b8ffe8f4c6262b \
    (name: "ecs-stress-ng-mem-1-main-f0fb97ba8ef6b98be201") died due to OOM" \
    module=docker_client.go
```

<!--
Notes
- TODO
-->

---

### cgroups v2: OOM-kill behavior

Kernel handles OOM-kill and systemd logs it in `/var/log/messages`:
```
$ sudo cat /var/log/messages | grep -iI oom
Apr 15 23:04:52 localhost kernel: head invoked oom-killer: gfp_mask=0x500cc2(GFP_HIGHUSER|__GFP_ACCOUNT), order=0, oom_score_adj=0
Apr 15 23:04:52 localhost kernel: oom-kill:constraint=CONSTRAINT_MEMCG,nodemask=(null),\
    cpuset=docker-60ef727c9751aac9084702bca274544f94f1bb070ace95e8a79911c22b01a91d.scope,mems_allowed=0,\
    oom_memcg=/ecstasks.slice/ecstasks-488fc5b8b77f4fb7984bdc4aba948f9f.slice,\
    task_memcg=/ecstasks.slice/ecstasks-488fc5b8b77f4fb7984bdc4aba948f9f.slice/docker-60ef727c9751aac9084702bca274544f94f1bb070ace95e8a79911c22b01a91d.scope,\
    task=tail,pid=51947,uid=0
Apr 15 23:04:52 localhost kernel: Memory cgroup out of memory: Killed process 51947 (tail) total-vm:459896kB, anon-rss:458176kB, file-rss:772kB, shmem-rss:0kB, UID:0 pgtables:948kB oom_score_adj:0
Apr 15 23:04:52 localhost systemd[1]: docker-60ef727c9751aac9084702bca274544f94f1bb070ace95e8a79911c22b01a91d.scope: A process of this unit has been killed by the OOM killer.
Apr 15 23:04:52 localhost kernel: oom_reaper: reaped process 51947 (tail), now anon-rss:0kB, file-rss:0kB, shmem-rss:0kB
```
and the parent slice remembers! [6]
```
$ cat /sys/fs/cgroup/ecstasks.slice/memory.events
low 0
high 0
max 21
oom 1
oom_kill 1                                                                                                                      .
```

<!--
Notes
- TODO
-->

---

# Thank you
Cam Sparr
cssparr@
github.com/sparrc
slides: [github/sparrc/slides/cgroups-v2/slides.md](https://github.com/sparrc/slides/blob/main/cgroups-v2/slides.md)
![](https://avatars.githubusercontent.com/u/7155926?s=300&u=6d5636aa89f316288ec27afe89fc4ab6f20fa566)


<!-- references -->

[1]: https://www.freedesktop.org/wiki/Software/systemd/ControlGroupInterface/ "The New Control Group Interfaces"
[2]: https://www.freedesktop.org/software/systemd/man/systemd.slice.html "systemd.slice"
[3]: https://www.freedesktop.org/software/systemd/man/systemd.scope.html "systemd.scope"
[4]: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/ "kubernetes cgroup driver"
[5]: https://github.com/moby/moby/pull/40846 "docker cgroup driver"
[6]: https://github.com/aws/amazon-ecs-logs-collector/pull/68 "collect cgroup v2 events in ecs log collector"
[7]: https://baykara.medium.com/docker-resource-management-via-cgroups-and-systemd-633b093a835c "creating systemd slices"
[8]: https://www.freedesktop.org/software/systemd/man/systemd.service.html "systemd.service"
