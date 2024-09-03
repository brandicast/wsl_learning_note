# wsl_learning_note

Had been using Linux Mint as Desktop OS for years without much problem.  However, as kids are getting bigger and with more opinions, they start complianing about my Desktop are different from what they'd learned in school.  I tried to tell them though it looks a bit different and they almost have the same features.  But just like many of us at work, when it comes to document compliance, we have to somehow compromise with Microsoft Office.  

For such reason, I decided to give Windows 11 another try out.  But I am reluctant to give up my linux easily, especially I am using linux software raid (mdadm) for years.  So it seems wsl would be an option.   (I'd tried VirtualBox on Linux, but didn't feel gaining any benefit from it)


## WSL How to

There are plenty of resources about wsl.  Here is just my personal quick memo.

https://learn.microsoft.com/zh-tw/windows/wsl/

1. First, decide go for wsl 1 or wsl 2. 
2. wsl commands
3. mounting Linux HDD 
4. how to auto mount on login
5. how to auto un-mount on shutdown and why?


### Mounting Linux Raid on wsl

Refer to 

https://medium.com/nerd-for-tech/windows-11-shenanigans-how-to-mount-any-linux-filesystem-in-windows-e63a60aebb05

Brief  summarize:

1. Find out physical drive

    [PowerShell]

        wmic diskdrive list brief
2. Mount physical dirve from wsl

    [PowerShell]

        wsl --mount \\.\PHYSICALDRIVE0 --bare
        wsl --mount \\.\PHYSICALDRIVE1 --bare
    Certainly need to make sure and identify the desired drive first.
3. Mount HDD

    If it's regular linux drive, should be able to see the drive from /dev/sdx already.   

    As mine is Linux Software Raid, 

    [wsl terminal]

        sudo blkid
    make sure it's the raid disk. Then

    [wsl terminal]

        sudo mdadm --assemble /dev/sdx /dev/sdy ...

    Then should be able to find out /dev/mdX

    last, mount disk array

    [wsl terminal]

        sudo mount -t ext4 /dev/mdX /mnt/your_raid_name

### Auto mount

1. Create a script as following example:

        wsl --mount \\.\PHYSICALDRIVE0 --bare
        wsl --mount \\.\PHYSICALDRIVE1 --bare
        wsl -u root -e bash -c "mount /dev/md5 /mnt/raid_strip"

2. [Win+R] with to open task scheduling tool

3. Create a basic job 

4. Select the script and choose to execute during login

* remember to choose "execute with highest priority"

### Auto unmount

1. Create a script as following example:

        @echo off
        wsl --unmount <DiskPath>
2. [Win+R] with gpedit.msc to open policy editor

3. In [Computer settings] >> [Windows settings] >> [Scripts] 
    Select [shutdown] and add above script

* For Windows 11 Home Edition, by default, there's nogpedit.msc.  Use the following script with administrator to install/activate (according to copilot)

        @echo off
        pushd "%~dp0"
        dir /b %SystemRoot%\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientExtensions-Package~3*.mum >List.txt
        dir /b %SystemRoot%\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientTools-Package~3*.mum >>List.txt
        for /f %%i in ('findstr /i . List.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
        pause


### misc

- on explorer  \\wsl$  to access the home
