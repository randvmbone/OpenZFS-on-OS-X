# [OpenZFS on OS X](https://openzfsonosx.org/)

## Identify Disks

```
randvmb@RANDVMBs-Server ~ % diskutil list
...
/dev/disk1 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *3.0 TB     disk1
   1:                        EFI ⁨EFI⁩                     209.7 MB   disk1s1
   2:                  Apple_HFS ⁨Untitled                3.0 TB     disk1s2

/dev/disk2 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *3.0 TB     disk2
   1:                        EFI ⁨EFI⁩                     209.7 MB   disk2s1
   2:                  Apple_HFS ⁨Untitled⁩                3.0 TB     disk2s2

/dev/disk3 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *3.0 TB     disk3
   1:                        EFI ⁨EFI⁩                     209.7 MB   disk3s1
   2:                  Apple_HFS ⁨Untitled                3.0 TB     disk3s2

/dev/disk4 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *3.0 TB     disk4
   1:                        EFI ⁨EFI⁩                     209.7 MB   disk4s1
   2:                  Apple_HFS ⁨Untitled⁩                3.0 TB     disk4s2

/dev/disk5 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *3.0 TB     disk5
   1:                        EFI ⁨EFI⁩                     209.7 MB   disk5s1
   2:                  Apple_HFS ⁨Untitled⁩                3.0 TB     disk5s2
```

## Encrypty Disks

**coreStorage convert is no longer available as of macOS Big Sur, use macOS Recovery**

## Verify Conversion

```
randvmb@RANDVMBs-Server ~ % diskutil coreStorage list | grep Conversion
|       Conversion Status:       Complete
|       Conversion Status:       Complete
|       Conversion Status:       Complete
|       Conversion Status:       Complete
        Conversion Status:       Complete
```

## Get Disk UUIDs

```
randvmb@RANDVMBs-Server ~ % diskutil list
...
/dev/disk7 (internal, virtual):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                  Apple_HFS ⁨Untitled                +3.0 TB     disk7
                                 Logical Volume on disk4s2
                                 CE9D8D81-8359-45A9-9BA9-F25E2A45FCB6
                                 Locked Encrypted

/dev/disk8 (internal, virtual):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                  Apple_HFS ⁨Untitled⁩                +3.0 TB     disk8
                                 Logical Volume on disk2s2
                                 E9A2DF94-334E-43E1-BD02-8320EEF3244E
                                 Locked Encrypted

/dev/disk9 (internal, virtual):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                  Apple_HFS ⁨Untitled                +3.0 TB     disk9
                                 Logical Volume on disk1s2
                                 E11AF673-B163-438E-A062-E35D1400D824
                                 Locked Encrypted

/dev/disk10 (internal, virtual):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                  Apple_HFS ⁨Untitled⁩                +3.0 TB     disk10
                                 Logical Volume on disk3s2
                                 05C2AC3E-14C1-4E4F-B13D-C286452B838B
                                 Locked Encrypted

/dev/disk11 (internal, virtual):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                  Apple_HFS ⁨Untitled                +3.0 TB     disk11
                                 Logical Volume on disk5s2
                                 1DA10BB0-4CF4-472C-B644-70AD9EB9D598
                                 Locked Encrypted

```

## Unlock Disks

**unmount disks before continuing**

```
diskutil coreStorage unlockVolume -nomount E11AF673-B163-438E-A062-E35D1400D824
diskutil coreStorage unlockVolume -nomount E9A2DF94-334E-43E1-BD02-8320EEF3244E
diskutil coreStorage unlockVolume -nomount CE9D8D81-8359-45A9-9BA9-F25E2A45FCB6
diskutil coreStorage unlockVolume -nomount 05C2AC3E-14C1-4E4F-B13D-C286452B838B
diskutil coreStorage unlockVolume -nomount 1DA10BB0-4CF4-472C-B644-70AD9EB9D598
```

## Create ZPOOL

```
randvmb@RANDVMBs-Server ~ % sudo zpool create -f -o ashift=12 \
-O compression=lz4 \
-O casesensitivity=insensitive \
-O atime=off \
-O normalization=formD \
Vault raidz2 /dev/disk7 /dev/disk8 /dev/disk9 /dev/disk10 /dev/disk11
```

## Verify ZPOOL

```
randvmb@RANDVMBs-Server ~ % zpool status
  pool: Vault
 state: ONLINE
config:

	NAME                                            STATE     READ WRITE CKSUM
	Vault                                           ONLINE       0     0     0
	  raidz2-0                                      ONLINE       0     0     0
	    media-E11AF673-B163-438E-A062-E35D1400D824  ONLINE       0     0     0
	    media-E9A2DF94-334E-43E1-BD02-8320EEF3244E  ONLINE       0     0     0
	    media-CE9D8D81-8359-45A9-9BA9-F25E2A45FCB6  ONLINE       0     0     0
	    media-05C2AC3E-14C1-4E4F-B13D-C286452B838B  ONLINE       0     0     0
	    media-1DA10BB0-4CF4-472C-B644-70AD9EB9D598  ONLINE       0     0     0

errors: No known data errors
```
