---
layout : post
title : Working around slow deletes of components in Sonatype Nexus Repository Manager 3 OSS
tags:
 - nexus
 - sonatype
 - performance
 - investigation
---

I spent a while trying to reclaim disk from an on-premise Nexus Repository Manager instance, and got stuck when delete requests would take a surprisingly long time.

The Nexus instance is an old version, running on an old OS, old hardware and an old JVM, simply because we haven't got round to upgrading it yet. So this is not me complaining about Sonatype -- instead, I want to leave a search-engine-friendly explanation of the problem I found, in case anyone else experiences it. It's been raised as a [bug](https://issues.sonatype.org/browse/NEXUS-21764) in Sonatype's JIRA, but that isn't publicly accessible (yet, I'm not sure if it will be) but I don't think there's any security concerns around the bug, so I've repeated here verbatim in the hopes it might help someone else.


tl;dr: deleting any component takes > 15minutes, deleting all the assets belonging to it takes <1 seconds.

We run Nexus Repository Manager OSS on-premise, with local disk storage. On finding disk was about to run out, I set about deleting unused artifacts. We do not use SNAPSHOT vs RELEASE artifacts, because we are (mostly) continuously deploying new versions, so I didn't feel existing cleanup tasks fit the bill. I also was not able to confirm it is safe to delete all artifacts with a published-at or last-accessed-at policy – the fear is that some projects may not have built in a while, but still need that library available. I opted for removing versions of artifacts that I knew that were safe to delete (via special knowledge, nothing systematic) .

I tried issuing HTTP DELETE requests with a component ID, and also clicking "Delete Component" in the Web UI. In both cases the response would not return for many minutes (if at all) and the Nexus instance would experience high sustained CPU load, sometimes to the point where it could not serve artifacts. In some cases, shortly after issuing the DELETE, if I did a GET with the same ID, it would return nothing. Which suggests there is some lingering task in the delete that blocks the HTTP response, but still actually gets the job done. In some cases the server would not recover except with a service restart.

During delete component requests, there would be a series of active threads, named like: "Thread-10723 <command>sql.select from component where (group = :p0 AND name = :p1) and (bucket=#15:4)</command>" #11152 prio=5 os_prio=0 tid=0x00007f02c4037800 nid=0xc10 runnable [0x00007f017cbb1000]" that would quickly trace back into org.sonatype.nexus.repository.storage, or into OrientDb code. It seemed to be that there were many threads spawned of a similar name, for a single request, and they never appeared to be deadlocked or waiting for too long.

We use one default blobstore for all repositories, proxy several public repositories (central, jcenter, npm) one private docker repository with Amazon ECR, and several private repositories which are of type either maven2 or npm. I only attempted to delete components from a maven2 repository.  The blobstore was approx 180G, and the same behaviour would occur on components with a very small and a very large number of versions (from 1 to 4k). Components usually had around 8-10 assets attached to them (jars, checksums, etc) and no individual asset was larger than 10MB. I wasn't able to update to the latest version to find if the problem still exists on recent versions.

Unlike deleting components, deleting the assets directly is very fast. Since I could traverse search results to access the asset IDs via the components I wanted to delete, it wasn't difficult to script, and I have been able to reclaim space.

Because the setup uses nothing up-to-date (hardware, OS, JVM, Nexus version) I'm not sure how much value this bug report is. My motivation for raising this bug is to leave a breadcrumb for other poor souls who find themselves needing to clear space fast from an opaque blobstore, and are unsure how to workaround the problem that deleting a single component takes > 15 minutes. The answer appears to be to delete the underlying assets.

Environment: Nexus version 3.16.2, Linux (Ubuntu Trusty 14.04) virtual machine, ext4 filesystem, spinning disk. 2x Intel Westmere CPU, 2.6GHz. 2Gb Java heap, 4G MaxDirectMemorySize, 12G of addressable memory.