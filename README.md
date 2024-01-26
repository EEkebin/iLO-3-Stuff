# iLO Debugging and Others

This is a simple repository that is meant for helping get the Integrated Lights-Out 3 running and accessible.

## Contents

1. [Introduction](#introduction)

2. [HPE Integrated Lights-Out Standalone Remote Console](#hpe-integrated-lights-out-standalone-remote-console-for-windows)

3. [Connect to iLO via SSH](#connect-to-ilo-via-ssh)

4. [Updating iLO through SSH](#updating-ilo-through-ssh)

5. [Credits](#credits)

6. [Disclaimers](#disclaimers)

---

## Introduction

Sometimes, iLO 3 with Default Settings will not allow you to connect via the Standalone Remote Console. It will also disallow the connection through the web portal. Tools and steps in this guide will _hopefully_ get them operational and accessible.

**These steps were produced using my knowledge of HPE Proliant DL380 G7. Although, I imagine that they wont be too different, if at all, for different versions of servers as well.**

---

## HPE Integrated Lights-Out Standalone Remote Console for Windows

### **Download links**

> Direct Links maybe broken or outdated. If so, please use the pages to download.

1. Hewlett-Packard Enterprise

   - [Official Page](https://support.hpe.com/connect/s/softwaredetails?language=en_US&softwareId=MTX_bc8e3ffa59904ec3b505d9964d)
   - [Direct Link](https://downloads.hpe.com/pub/softlib2/software1/pubsw-windows/p390407056/v138774/Setup.exe)

2. Archive.org
   - [Official Page](https://archive.org/details/hpe-lights-out-standalone-remote-console-for-windows)
   - [Direct Link](https://archive.org/download/hpe-lights-out-standalone-remote-console-for-windows/Setup.exe)

## Connect to iLO via SSH

Connecting to iLO via SSH requires two extra parameters.

    1. -oHostKeyAlgorithms
    2. -oKexAlgorithms

> Any steps proceeding have variables, such as **<EXAMPLE_VARIABLE>**.  
> Please replace these, including the arrows, with their respective, correct value.

1. Connect iLO port to a network that you can access.

2. Obtain the IP Address of iLO by either setting it as static through Configure iLO inside of the server.

3. Make a user of iLO through Configure iLO, or through Configure iLO.

4. Attempt a connection via SSH, with its given IP, through a different device that has network access to iLO.

   ```sh
   ssh <USERNAME>@<IP_ADDRESS>
   ```

   > **For example:**
   >
   > ```sh
   > ssh Administrator@192.168.1.2
   > ```

   You will get a response back that looks something like the following:

   ```
   Unable to negotiate with <IP_ADDRESS> port 22: no matching key exchange method found. Their offer: <list-of-words-or-names>
   ```

   > \*In my case, the list-of-words-or-names was **diffie-hellman-group14-sha1,diffie-hellman-group1-sha1\***

   Given this, you have your _-oKexAlgorithms_ parameter.

5. Attempt a second connection via SSH with the _-oKexAlgorithms_ parameter included.

   ```sh
   ssh -oKexAlgorithms=+"<list-of-words-or-names>" <USERNAME>@<IP_ADDRESS>
   ```

   > **For example:**
   >
   > ```sh
   > ssh -oKexAlgorithms=+"diffie-hellman-group14-sha1,diffie-hellman-group1-sha1" Administrator@192.168.1.2
   > ```

   You will get a similar response to the previous. Something like the following:

   ```
   Unable to negotiate with 192.168.1.25 port 22: no matching host key type found. Their offer: <list-2>
   ```

   > \*In my case, the list-2 was **ssh-dss\***

   Given this, you have your _-oHostKeyAlgorithms_ parameter.

6. Finally, make a successful connection with iLO using these two parameters via SSH.

   ```sh
   ssh -oKexAlgorithms=+"diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha256" -oHostKeyAlgorithms=+"ssh-dss" root@192.168.1.25
   ```

   > **For example:**
   >
   > ```sh
   > ssh -oKexAlgorithms=+"diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha256" -oHostKeyAlgorithms=+"ssh-dss" Administrator@192.168.1.2
   > ```

   After specifying the password, you will be greeted to some information about the status of the session, the server, and iLO.

## Updating iLO through SSH

1. Retrieve the firmware file from the following two sources:

   - [[Official] HPE](https://support.hpe.com/connect/s/search?language=en_US#q=HPE%20Integrated%20Lights-Out%20Online%20ROM%20Flash%20Component&t=DriversandSoftware&sort=relevancy&numberOfResults=25&f:@kmswsoftwaretypekey=[swt8000029]&f:@kmswsoftwaresubtypekey=[swst9000213])
     - Download the correct file for your iLO version, architecture and operating system of your PC.
   - [[Unofficial] list of all iLO firmware packages by pingtool.](https://pingtool.org/latest-hp-ilo-firmwares/)

   Retrieve the **_.bin_** file. This is the firmware of iLO.

2. [Connect to iLO SSH.](#connect-to-ilo-via-ssh)

3. Navigate to the `/map1/firmware1` directory.

   ```sh
   cd /map1/firmware1
   ```

4. Push the update via a file from the internet.

   ```sh
   load -source <url>
   ```

   > **For example:**
   >
   > ```sh
   > load -source https://example.com/ilo3-1.94/ilo3_194.bin
   > ```

5. Wait for the iLO to update. This could disconnect your SSH connection to iLO.

## Fix AES Encryption to allow access to the web portal of iLO.

1. [Connect to iLO via SSH.](#connect-to-ilo-via-ssh)

2. Navigate to the `/map1/config1` directory.

   ```sh
   cd /map1/config1
   ```

3. Set the Enforce AES Encryption property to yes.

   ```sh
   set /map1/config1 oemhp_enforce_aes=yes
   ```

4. Wait for the iLO to reboot. This could disconnect your SSH connection to iLO.

## Credits

> [EEkebin](https://github.com/EEkebin)

## Disclaimers

> I am not sponsored by or affiliated with Hewlett-Packard Company (HP), Hewlett-Packard Enterprise (HPE), and/or any of its affiliates.

> I am not sponsored by or affiliated with any of the tools linked, used, or referred to and/or any of their affiliates.

> All of the tools and information here are publicly available through either HPE's Official Website, and/or the public web.
