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

image:: https://github.com/praneshsoundar/ANF_SNAPSHOT_OFFLOAD_BLOB/blob/master/images/image1.PNG

Content of the Container:

Now schedule the move of the snapshot to the secondary storage (Azure Blob)

azacsnap@azacsnap01:~> crontab -l
0,30 * * * * ( . ~/.profile ; cd /home/azacsnap/bin ; ./azacsnap -c
backup --volume=data --prefix=half_hourly --retention=5 --trim --
configfile=azacsnap.json)

image:: 

Syncing the hidden .snapshot directory (and all the underlying) to Blob

azacsnap@azacsnap01:~/bin> azcopy sync
'/hana/data/ANA/mnt00001/.snapshot'
'https://azacsnaptoblob01.blob.core.windows.net/ana?sv=2019-02-
02&ss=bfqt&srt=sco&sp=rwdlacup&se=2030-02-04T08:25:26Z&st=2020-02-
04T00:25:26Z&spr=https&sig=DUMMY' --recursive=true --deletedestination=
true

Or copy the snapshot folder to Blob. (no automated retention management)
azacsnap@azacsnap01:~/bin> azcopy copy
'/hana/data/ANA/mnt00001/.snapshot'
'https://azacsnaptoblob01.blob.core.windows.net/azacsnapblobcontainer0
1?sv=2019-02-02&ss=bfqt&srt=sco&sp=rwdlacup&se=2020-01-
31T23:13:23Z&st=2020-01-31T15:13:23Z&spr=https&sig= DUMMY' --
recursive=true

Even better using sync option to maintain the destination
azacsnap@azacsnap01:~/bin> azcopy sync
'/hana/data/ANA/mnt00001/.snapshot'
'https://azacsnaptoblob01.blob.core.windows.net/ana?sv=2019-02-
02&ss=bfqt&srt=sco&sp=rwdlacup&se=2020-01-31T23:13:23Z&st=2020-01-
31T15:13:23Z&spr=https&sig=DUMMY' --recursive=true --deletedestination=
true

Blob destination
SAS Access key

Restoring files in another directory
azcopy copy
'https://azacsnaptoblob01.blob.core.windows.net/ana/half_hourly.2020-
02-03_2302?sv=2019-02-02&ss=bfqt&srt=sco&sp=rwdlacup&se=2020-02-
04T06:51:35Z&st=2020-02-03T22:51:35Z&spr=https&sig=DUMMY'
'/hana/data/ANA/mnt00001/Test' --recursive=true

If there is a special procedure needed e.g. that only the last snapshot sync to blob or so you can create
a simple snap script to manage this.

azacsnap01:/hana/data/ANA/mnt00001/.snapshot # ls -ltr
total 20
drwxr-x--- 5 anaadm sapsys 4096 Jan 10 17:11 half_hourly.2020-01-31_1530
drwxr-x--- 5 anaadm sapsys 4096 Jan 10 17:11 half_hourly.2020-01-31_1500
drwxr-x--- 5 anaadm sapsys 4096 Jan 10 17:11 half_hourly.2020-01-31_1430
drwxr-x--- 5 anaadm sapsys 4096 Jan 10 17:11 half_hourly.2020-01-31_1400
drwxr-x--- 5 anaadm sapsys 4096 Jan 10 17:11 half_hourly.2020-01-31_1330

azacsnap01:/hana/data/ANA/mnt00001/.snapshot # ls -tr | sed '$!d'
half_hourly.2020-01-31_1330

# Conclusion:

Using the above procedure you would be replicate the ANF snapshot to the secondary region using a cost efficient method.

# Credits:

I would like to give credits to Ralf Khar who has provided guidance on how to achieve this functonality.
