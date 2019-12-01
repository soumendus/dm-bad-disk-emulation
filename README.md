Author

## **Soumendu Sekhar Satapathy**

Email

**satapathy.soumendu@gmail.com**

# BAD DISK EMULATION
Modified ~/drivers/md/dm-dust.c file present in linux kernel version 5.4.1 to add write disk error emulation

I have modified  the  ~/drivers/md/dm-dust.c  file present in the  linux kernel stable ver 5.4.1 (version as of writing this comment) and have added functionality for emulating disk with "write errors" also.  I needed "write error"  functionality  for my  test  suite, so I have added  this  functionality to  the existing code. However the code that I have written needs some re factoring and cleanups but it works for me.

STEPS TO TEST THE ADDED FUNCTIONALITY
-------------------------------------


**How to Fetch the code and build. Please use latest linux kernel 5.4.1** 

$ git clone https://github.com/soumendus/modified-kernel-code.git

$ cd modified-kernel-code/

$ make

This will build the dm-dust.ko driver. 








**Create, lets say a 128MB file for your storage**

dd if=/dev/zero of=/tmp/myfile bs=1M count=128 # 128MB file






**Attach the file with a block device. loop16 here, behaves as a block device.**

losetup /dev/loop16 /tmp/myfile






**Load the dm-dust.ko module**

insmod ~/dm-dust.ko






**Get the size of your block storage**

blockdev --blksz /dev/loop16






**Create the device dust1: 512 here is the block size. You can check dust1 device created in /dev/mapper**

dmsetup create dust1 --table '0 2621440 dust /dev/loop16 0 512'






**Check the status of the device dust1 that you have created. Initially, defaults will be set.**

dmsetup status dust1

0 2621440 dust 7:16 bypass verbose

7:16 bypass verbose






**Add bad blocks to fail write. Add blocks 61,65,67,72,87**

dmsetup message dust1 0 addbadblock write 61

dmsetup message dust1 0 addbadblock write 65

dmsetup message dust1 0 addbadblock write 67

dmsetup message dust1 0 addbadblock write 72

dmsetup message dust1 0 addbadblock write 87






**Check the blocks that you have added to fail write**

dmsetup message dust1 0 countbadblocks write






**Enable to fail write on the added blocks**

dmsetup message dust1 0 enable write






**Check the status now after enabling write to fail on the added blocks.**

dmsetup status dust1

0 2621440 dust 7:16 bypass verbose

7:16 fail_write_on_bad_block verbose






**Now try to write to your disk,  You should get write error if the write goes to the added blocks.**

dd if=/dev/zero of=/dev/mapper/dust1 bs=512 count=128 oflag=direct










