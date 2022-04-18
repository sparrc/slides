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

---

### cgroups v1: structure

- cgroups v1 has a structure where a process is split across many sub-directories (memory, blkio, cpu, etc.):
```
$ find /sys/fs/cgroup -type d | grep 7ff2ace8e15097939aa2ca98f2179fc216aa9faca0eafdcf92dd9c755bd7c4f3
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

---

### cgroups v2: unified structure

- cgroups v2 has a "unified" structure, in which all of the controls for a process are unified in a single directory:
```
$ find /sys/fs/cgroup -type d | grep e883b4e9fbc445677c65ce9325a2ec5548c30b2709c02141da64b7992c430afe
/sys/fs/cgroup/system.slice/docker-e883b4e9fbc445677c65ce9325a2ec5548c30b2709c02141da64b7992c430afe.scope
```

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

---

### cgroups v1: ECS task limits

- ECS tasks with resource limits have their own cgroup created, under which all containers of that task are created.
- task ID `e39cda25f9824fe9b035e2f57e81c793` has it's own directory within the `memory` control.
- this task has a 450MB limit (471859200 bytes)
```
$ cat /sys/fs/cgroup/memory/ecs/e39cda25f9824fe9b035e2f57e81c793/memory.limit_in_bytes
471859200
```

---

### cgroups v2: ECS task limits

- task ID `e95a40efbde84cbc8600660a109ec194` has it's own systemd slice (more on this later).
- this task has a 450MB limit (471859200 bytes)
```
$ cat /sys/fs/cgroup/ecstasks.slice/ecstasks-e95a40efbde84cbc8600660a109ec194.slice/memory.max
471859200
```

---

### cgroup drivers

- **cgroup drivers**: cgroupfs, systemd
- **cgroup versions**: v1, v2
- they are technically independent, but, in practice:
- cgroup v1 <=> cgroupfs
- cgroup v2 <=> systemd
- docker and kubernetes both recommend using the systemd driver. [4] [5]

---

### cgroupfs driver

- the cgroupfs driver effectively allows users to create cgroups by creating files and directories in `/sys/fs/cgroup`

this creates a cgroup for ecs task `e39cda25f9824fe9b035e2f57e81c793`:
```
$ mkdir -p /sys/fs/cgroup/memory/ecs/e39cda25f9824fe9b035e2f57e81c793
```

---

### systemd driver

- All containers and tasks are managed as a systemd unit. [1]
- **note:** they are systemd slices and scopes, not services. [2] [3]
- systemd slices can be created with files (like services can), but scopes cannot [7]

---

### systemd driver

- A slice unit is a concept for hierarchically managing resources of a group of processes. **scope** and **service** units are assigned to a specific slice. [2]
- A scope unit manages a set of system processes. Unlike service units, scope units manage externally created processes, and do not fork off processes on its own. [3]
```
$ tree -d /sys/fs/cgroup/ecstasks.slice
/sys/fs/cgroup/ecstasks.slice                                                               <-- Parent slice for ECS tasks w/ resource limits
├── ecstasks-14e4b4156f6b47d18e1ff3cc55a47bc7.slice                                         <-- Task 1 parent slice
│   ├── docker-5a84a635371cf7f2255700c02a0dde027b62515db33cb14934ab62b7edfc27cf.scope       <-- Task 1 container 1 scope
│   └── docker-e883b4e9fbc445677c65ce9325a2ec5548c30b2709c02141da64b7992c430afe.scope       <-- Task 1 container 2 scope
└── ecstasks-ef1d696601704c3ebc321f0932e8a0bf.slice                                         <-- Task 2 parent slice
    └── docker-16b283f08cc44cc0eb7d6b055a2d42bbc38f53bf32d9f05daa15e7199ff6f9da.scope       <-- Task 2 container 1 scope                      .
```

---

### systemd driver

- A little more context on systemd slices/services/scopes, most systemd service units are part of the `system.slice` systemd slice.
- TLDR; a systemd *scope* is similar to a systemd service, but with far fewer features.

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

---

### systemd driver

- when we create a task, systemd logs it in `/var/log/messages`

```
$ tail -f /var/log/messages
systemd[1]: Created slice Slice /ecstasks.
systemd[1]: Created slice cgroup ecstasks-0a272520dcbb4340a96121fab054a479.slice.
systemd[1]: Starting Set up policy routes for ens5...
systemd[1]: refresh-policy-routes@ens5.service: Deactivated successfully.
systemd[1]: Finished Set up policy routes for ens5.
systemd[1]: var-lib-docker-overlay2-7d2c540c1a633c2349c3f2ef2a7445dc1800e8bae257c26bf6e9b50ec6d37406\x2dinit-merged.mount: Deactivated successfully.
dockerd[3]: time="2022-04-18T20:00:33Z" level=info msg="Configured log driver..." container=579341a0143cb9256598d0d7c94fcd7fd8947d51c9fccf5a9e1739bc10580cbb driver=awslogs
containerd[2]: time="2022-04-18T20:00:33Z" level=info msg="starting signal loop" namespace=moby path=/run/.../moby/579341a0143cb9256598d0d7c94fcd7fd8947d51c9fccf5a9e1739bc10580cbb
systemd[1]: Started libcontainer container 579341a0143cb9256598d0d7c94fcd7fd8947d51c9fccf5a9e1739bc10580cbb.
```

---

### cgroups v1: OOM-kill behavior

Kernel handles the OOM-kill and docker (tries) to report it to ecs agent through a container event:

TODO see if docker logs OOM-kill with debug logs turned on

```
$ sudo cat /var/log/ecs/ecs-agent.log | grep OOM
level=info time=2022-04-15T23:21:53Z msg="DockerGoClient: process within container 8268d4612267114894856a5d12d316fa1bf65da2045e3481a0b8ffe8f4c6262b \
    (name: "ecs-stress-ng-mem-1-main-f0fb97ba8ef6b98be201") died due to OOM" \
    module=docker_client.go
```

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
