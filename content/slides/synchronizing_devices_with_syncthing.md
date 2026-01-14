---
title: "Synchronizing Devices with Syncthing"
date: 2025-11-15T00:00:00+01:00
lastmod: 2026-01-13T22:21:50+01:00
draft: false
---

## Synchronizing devices with Syncthing {#synchronizing-devices-with-syncthing}

{{< figure src="/ox-hugo/syncthing.png" >}}


### About me {#about-me}

-   Hi, my name is Marcel Patzwahl
-   I'm a Platform Engineer at MOIA by day
-   I love:
    -   Open Source
    -   Configuration Management &amp; Tinkering
    -   Everything Linux &amp; Emacs specifically


### What is Syncthing {#what-is-syncthing}


#### Peer to Peer {#peer-to-peer}

-   Peer to Peer synchronization software
-   No central service
-   Sync across multiple devices
    -   If one device becomes unavailable, the rest can still talk to each other


#### Open Source {#open-source}

-   [Hosted on Github](https://github.com/syncthing/syncthing)
-   Binaries available for:
    -   MacOS
    -   Linux
    -   Windows
-   Third party packages available for mobile:
    -   Android: syncthing-fork
    -   iOS: Möbius Sync


#### Encryption <span class="org-target" id="org-target--Encrypted"></span> {#encryption}

-   End to End encrypted
    -   Gets encrypted on the sender side and decrypted on the receiver side


### Demo {#demo}

-   Setting up Syncthing on a new Device
-   Pair 2 devices
-   Add a folder to sync
-   Explore more things


### Syncthing Deep Dive {#syncthing-deep-dive}


#### Synchronisation across Networks {#synchronisation-across-networks}

-   Both devices don't need to be on the same network
-   They can also find each other when both are connected to the internet
-   2 technologies make this possible:
    -   Discovery Server
    -   Relaying

<!--list-separator-->

-  Discovery Server

    -   Used to find peers on the internet
    -   Similar to DNS in that you don't need IPs to find each other
    -   A global cluster in part operated by the Syncthing project
    -   Everyone can run a discovery server

<!--list-separator-->

-  Relaying

    -   Syncthing bounces traffic via a `relay` when it's not possible to establish a direct connection
    -   Many public `relays` serve this purpose
    -   Transfer rate much lower than direct connection
    -   Enabled by default, but direct connection is favored
        -   Can be turned off

    <!--list-separator-->

    -  Security Concerns

        -   Connection between the two communicating devices is still
        -   `relays` only transmit encrypted data, like a router
        -   When device registers with a `relay`, the following things are known to the `relay`:
            -   Your IP and Device ID
            -   The amount of data you sync
            -   But not it's content
        -   Everyone can run a `relay` server


#### How synchronisation actually works {#how-synchronisation-actually-works}

<!--list-separator-->

-  Blocks

    -   Files are devided into blocks
    -   Blocks have the same size except the last one
    -   Block size depends on file size
        -   Can be 128KiB to 16MiB
    -   A [SHA256](https://en.wikipedia.org/wiki/SHA-2) hash is computed for each block
    -   The result over all blocks is called the `block list`
        -   Contains the offset, size and hash of all blocks in a file

<!--list-separator-->

-  Updating a File

    -   Syncthing compares the `block list` of the current version of a file to the `block list` of the desired version
    -   Tries to find a source for each block that differs
        -   Locally if another file already has a block with the same hash =&gt; copied over
        -   Or from another device in the cluster =&gt; requested over the network from the other device
    -   When a block is copied or received from another device, it's [SHA256](https://en.wikipedia.org/wiki/SHA-2) is computed and compared with the expected value
        -   If it matches, the block is written to a temporary copy of the file
        -   otherwise it's discarded and Syncthing tries to find another source for the block

<!--list-separator-->

-  Scanning

    -   2 ways for Syncthing to detect changes:
        -   regular full scan (once per hour)
        -   notifications received from the filesystem watcher
    -   Configurable per folder

<!--list-separator-->

-  Syncing

    -   Syncthing keeps track of several version of each file:
        -   the version that it currently has on disk (local version)
        -   the versions announced by all other connected devices
        -   the "best" (usually most recent) version of the file
            -   also called `global` version
            -   every device strives to be up to date with the `global` version
    -   The version information is kept in an `index database`
    -   It's stored in the configuration or data directory called `index-<version-number>`
    -   When new index data is received from other devices Syncthing recalculates which version for each file should be the `global` version and compares it to the local version
        -   When the 2 differ, Syncthing synchronizes the file
            -   `block lists` are compared to build a list of needed blocks
            -   The blocks are requested from the network or copied locally

<!--list-separator-->

-  Conflicts

    -   Syncthing does recognize conflicts
    -   When a file has been modified on 2 devices simultaneously and the content differs
    -   One of the files will be renamed like this: `<filename>.sync-conflict-<date>-<time>-<modifiedBy>.<ext>`
    -   The file with the older modification time will be marked as the conflicting file
    -   When time is exactly the same, the file with the higher value in the first 63 bits in it's device ID will marked as the conflict
    -   The conflicting file is synced to every device

<!--list-separator-->

-  Filename case sensitivity

    -   In principle Syncthing works with case-sensitive paths
    -   However some Operating Systems (Windows) see them as similar and therefore as conflicts

<!--list-separator-->

-  Temporary Files

    -   Syncthing never writes into destination files directly
    -   Instead it creates a temporary copy which is then moved in place over the old version
    -   If an error occurs the temporary copy stays there up to a day
    -   Temporary files look like this `.syncthing.<original-filename>.<ext>.tmp`


### Questions? {#questions}
