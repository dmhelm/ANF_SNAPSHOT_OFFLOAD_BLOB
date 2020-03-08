# ANF_SNAPSHOT_OFFLOAD_BLOB

Introduction
This demo provides step by step instructions on how to offload the ANF snapshot to BLOB storage. Currently, most of the customers i work with has a requirement to offload the snapshots to the secondary region. This can be achieved using different methods mentioned below.

  1. Snap Mirror
  2. Using Cloud Sync Manager
  3. Using AzCopy

The intention of this blog is to focus on using AzCopy tool to offload the snapshot to the BLOB storage. This method is one of the cost effective and fast method of backing up ANF snapshots to Azure BLOB storage.

Without further due, let's start with the instructions.

# Step1: Download the latest AzCopy tool:

azacsnap@azacsnap01:~/bin> wget -O azcopy_v10.tar.gz https://aka.ms/downloadazcopy-v10-linux && tar -xf azcopy_v10.tar.gz --strip-components=1
Saving to: ‘azcopy_v10.tar.gz’
100%[===========================================================================================================================================>] 8,754,718 --.-K/s in 0.02s
2020-02-04 00:33:39 (373 MB/s) - ‘azcopy_v10.tar.gz’ saved

Couple of important features that helps to keep the cost effective. 

The most advanced feature is the SYNC option where azcopy will keep the source and the destination directory in SYNC. The Parameter --delete-destination is important otherwise azcopy is not deleting files at the destination site and the space will grow.

# Step 2: Create a BLOB container:

Before copying or syncing files it is required to create a Blob Container and create a SAS token for this Container.

![](master/images/image1.PNG)
