---
title: "Synchronizing Devices with Syncthing"
date: 2025-11-16T00:00:00+01:00
lastmod: 2025-11-16T00:00:00+01:00
draft: false
---
![img](syncthing.png)

---
## Table of Contents
1.  [[#What is Syncthing?]]
    1.  [[#Peer to Peer]]
    2.  [[#Open Source]]
    3.  [[#Encryption]]
2.  [[#Demo]]
3.  [[#Syncthing Deep Dive]]
    1.  [[#Synchronisation across Networks]]
    2.  [[#How synchronisation actually works]]
---
## What is Syncthing?
---
### Peer to Peer
- Peer to Peer synchronization software
- No central service
- Sync across multiple devices
    - If one device becomes unavailable, the rest can still talk to each other
---
### Open Source
- Hosted on GitHub
- Binaries available for:
    - MacOS
    - Linux
    - Windows
- Third party packages available for mobile:
    - Android: syncthingfork
    - iOS: Möbius Sync
---
### Encryption
- End to End encrypted
    - Gets encrypted on the sender side and decrypted on the receiver side
---
## Demo
- Setting up Syncthing on a new Device
- Pair two devices
- Add a folder to sync
---
## Syncthing Deep Dive
---
### Synchronisation across Networks
- Both devices don't need to be on the same network
- They can also find each other when both are connected to the internet
- 2 technologies make this possible:
    - Discovery Server
    - Relaying
---
#### Discovery Server
- Used to find peers on the internet
- Similar to DNS in that you don't need IPs to find each other
- A global cluster in part operated by the Syncthing project
- Everyone can run a discovery server
---
##### Relaying
- Syncthing bounces traffic via a _relay_ when it's not possible to establish a direct connection
- Many public _relays_ serve this purpose
- Transfer rate much lower than direct connection
- Enabled by default, but direct connection is favored
  - Can be turned off
---
##### Security Concerns
- Connection between the two communicating devices is still [[#Encryption|encrypted]]
- relays only transmit encrypted data, like a router
- When device registers with a _relays_, the following things are known to the relay:
  - Your IP and Device ID 
  - The amount of data you sync
  - But not it's content
- Everyone can run a _relay_ server 
---
### How synchronisation actually works
---
#### Blocks
- Files are devided into blocks
- Blocks have the same size except the last one
- Block size depends on file size
    - Can be 128KiB to 16MiB
- A [SHA256](https://en.wikipedia.org/wiki/SH2) hash is computed for each block
- The result over all blocks is called the `block list`
    - Contains the offset, size and hash of all blocks in a file
---
#### Scanning
- 2 ways for Syncthing to detect changes:
    - regular full scan (once per hour)
    - notifications received from the filesystem watcher
- Configurable per folder
---
#### Syncing
- Syncthing keeps track of several version of each file:
    - the version that it currently has on disk (local version)
    - the versions announced by all other connected devices
    - the "best" (usually most recent) version of the file
        - also called `global` version and every device strives to be up to date with that version
---
- The version information is kept in an `index database`
- It's stored in the configuration or data directory called `index<versionumber>`
---
##### When new index data is received from other devices
- Syncthing recalculates which version for each file should be the `global` version and compares it to the local version
- When the 2 differ, Syncthing synchronizes the file
    - `block lists` are compared to build a list of needed blocks
    - The blocks are requested from the network or copied locally
---
#### Conflicts
- Syncthing does recognize conflicts
- When a file has been modified on 2 devices simultaneously and the content differs
- One of the files will be renamed: `<filename>.synconflic<date<time<modifiedBy>.<ext>`
- file with the older modification time will be marked as the conflicting file
- When time is the same, the file with the higher value in the first 63 bits of it's device ID will marked as the conflict
- The conflicting file is synced to every device
---
#### Filename case sensitivity
- In principle Syncthing works with case sensitive paths
- However some Operating Systems (Windows) see them as similar and therefore as conflicts
---
#### Temporary Files
- Syncthing never writes into destination files directly
- Instead it creates a temporary copy which is then moved in place over the old version
- If an error occurs the temporary copy stays there up to a day
- Temporary files look like this `.syncthing.originafilename.ext.tmp`
---
## Questions?
- You can find the slides under [snow-dev.com/talks/Synchronizing-Devices-with-Syncthing](https://snow-dev.com/talks/Synchronizing-Devices-with-Syncthing)
