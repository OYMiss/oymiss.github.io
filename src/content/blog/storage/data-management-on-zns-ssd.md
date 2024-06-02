---
author: Chongzhuo Yang
title: Designing a High-Performance Management Layer for ZNS SSDs
pubDatetime: 2024-05-31 15:25:07
tags:
  - storage
slug: "data-management-on-zns-ssd"
description: A guide to designing a high-performance management layer for data I/O, data mapping, and garbage collection on ZNS SSDs.
---

## Zone Interface

There are requirements of writing data to zones of ZNS SSDs.

1. Data can only be written at the write pointer.
2. Write pointers can be increased and reset by `append` and `reset`.
3. There is the limitation of maximum number of open and active zones.

> A closed zone is not an open zone but it is still an active zone. The `finish` can be used to make zones inactive.

## I/O Control

For data writes, we should ensure that: 1. One zone should be written only by one thread at the same time; 2. The number of active zones should be limited.

### Zone Group

We can use the status management to control the writing behavior and align with the limitation of max active zones. The zone status can be divided into four different groups: **free group**, **waiting group**, **writing group**, **finished group**.

<img class="d-block-light d-block-hidden" src="/assets/data-management-on-zns-ssd/group-state.png" style="width: 60%; height: 60%" alt="group-state"/>

<img class="d-block-dark d-block-hidden" src="/assets/data-management-on-zns-ssd/group-state-dark.png" style="width: 60%; height: 60%" alt="group-state"/>

Initially, all zones are assigned to the free group. If the storage engine cannot find a zone in writing group, it will transfer one zone from the free group to the waiting group (step 1). If a suitable zone is available, it will be moved to the writing group (step 2). Following the write operation, the zone can either remain in the waiting group (step 3) for additional writes or be transferred to the finished group (step 4) to reduce the number of active zones. When the storage nears capacity, the garbage collection (GC) can selectively reclaim zones from the finished group to provide more free zones (step 5).

### Data Mapping

Unlike regular SSDs, the data mapping between logical addresses and zone addresses should be maintained by the client. Therefore, applications can access data using logical addresses. The mapping can be designed in various ways to cater to different data layout requirements.

In this guide, we use the simplest data layout, where all data units have the same size.

### Parallel Control

One thread can allocate a zone in the waiting group for exclusive writing. After that, the zone will be transferred into the writing group and further writes of the zone do not need locks. The states of the zone group are controlled using a mutex lock (called `#group-lock`). The number of active/open zones can be controlled in steps 1 and 4 using a simple counter.

The mapping will be updated after writing data and accessed before reading data. Therefore, a mutex lock is also needed for mapping (called `#mapping-lock`).

To avoid deadlock, we should pay attention to the lock order of `#group-lock` and `#mapping-lock`.

## Garbage Collection

The zones in the finished group can be selected to reclaim invalid space resulting from data updates and deletions. The victim selection policy is an important design in ZNS SSDs. After selecting the victim zone, we will move valid data from the victim zone to the special GC zone reserved for the GC process.

The victim zone's mapping may also be updated when moving its data. We ignore these updates during data copying, which may move more data but results in a simpler and more efficient GC design. After moving the data, we will check and update the mapping correctly. If the data moved is marked as invalid (by updates or deletes) during moving, we just abort the mapping updates. Otherwise, we will update the mapping with the new data offset.

After moving all valid data, the victim zone should be reset and moved to the free group. However, since applications may still be reading from the victim zone, we need to introduce a new shared lock, `#read-lock`, to delay the reset operation.

For delete, after we moved the data and wanted to update the mapping.  we check the mapping whether the data is updated. If the data is updated, we abort the updating. If not, we update the moved offset.

## Control Flow

### Read

1. lock `#mapping-lock`.
2. get zone offset from mapping.
3. lock `#read-lock`.
4. unlock `#mapping-lock`.
5. read data from zone.
6. unlock `#read-lock`.

### Write

1. lock `#group-lock`.
2. allocate zone for writing.
3. unlock `#group-lock`.
4. write data to zone.
5. lock `#group-lock` and `#mapping-lock`.
6. update mapping and group states.
7. unlock `#group-lock` and `#mapping-lock`.

### Garbage Collection

1. lock `#group-lock`.
2. find a victim zone for reclaiming.
3. lock `#mapping-lock`.
4. read offsets of valid data.
5. unlock `#group-lock` and `#mapping-lock`.
6. move data in victim zone.
7. lock `#mapping-lock`.
8. update mapping.
9. unlock `#mapping-lock`.
10. repeat 6-9 until all data is moved.
11. lock `#group-lock` and `#mapping-lock`.
12. lock `#read-lock`.
13. reset victim zone.
14. unlock `#read-lock`.
15. update group states.
16. unlock `#group-lock` and `#mapping-lock`.


