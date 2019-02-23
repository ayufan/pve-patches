# Proxmox Patches

Proxmox VE is a complete virtualization management solution for servers. You can virtualize even the most demanding application workloads running on Linux and Windows Servers. Checkout their page: [proxmox.com](http://proxmox.com/products/proxmox-ve).

Due to lack of better backup functionality I prepared patches to support differential backups in Proxmox VE. These patches are in use for over a year.

<!--more-->

## What are differential backups?

According to Wikipedia:

> A differential backup is a type of data backup that preserves data saving only the difference in the data since the last full backup. (…) Another advantage, at least as compared to the incremental backup method of data backup, is that at data restoration time, at most two backup media are ever needed to restore all the data. This simplifies data restores as well as increases the likelihood of shortening data restoration time.

## What my patches do?

My patches extends vzdump, xdelta3 and Web-GUI support. And yes, these patches fully support OpenVZ and KVM.

System administrator can use one additional parameter in Backup Jobs (Datacenter -> Backup -> Add/Edit) the **Full Backup Every**:

* By default this value is set to 0. Which simply means: **use old behavior** (always create full backups).
* But if you specify value larger than 0, for example 7. It will instruct the vzdump to create full backup once a week and use differentials for the rest.

Please consider the following example:

        vzdump 101 --remove 0 --mode snapshot --compress lzo --storage local --node hvm --fullbackup 4


It will create full backup of VM101 every 4 days, compressed using lzo, stored on local storage.

## How to install?

The installation procedure is fairly simple:

    ```bash
    git clone https://github.com/ayufan/pve-patches
    cd pve-patches
    bash pve-5.1-diff-backup-addon apply
    ```

When everything went right, you’ll see:

    ```text
    Proxmox VE 5.1 - differential backup support
    Kamil Trzcinski, http://ayufan.eu/, ayufan@ayufan.eu

    PATCHED: /usr/share/pve-manager/
    PATCHED: /usr/share/perl5/PVE/

    Restarting PVE API Proxy Server: pveproxy.
    Restarting PVE Daemon: pvedaemon.
    ```

Then install latest version of `pve-xdelta3`, which enables the support for **LZOP** compressor. You can find the sources [here](https://github.com/ayufan/pve-xdelta3).

    ```bash
    dpkg -i pve-xdelta3_3.0.6-1_amd64.deb
    ```

Feel free to compile the sources yourself by downloading the repo and executing `dpkg-buildpackage -b -us -uc`.

## And what about uninstall?

The procedure is simpler than installation. Type in the bash:

    ```bash
    bash pve-5.1-diff-backup-addon revert
    ```

After a while, you’ll see:

    ```text
    Proxmox VE 5.1 - differential backup support
    Kamil Trzcinski, http://ayufan.eu/, ayufan@ayufan.eu

    RESTORED: /usr/share/pve-manager/
    RESTORED: /usr/share/perl5/PVE/

    Restarting PVE API Proxy Server: pveproxy.
    Restarting PVE Daemon: pvedaemon.
    ```

## What about UPGRADE? (READ THIS)

**This is important part.** If you will ever want to upgrade your Proxmox installation (by *apt-get dist-upgrade* or *apt-get upgrade*) *ALWAYS* revert/uninstall patches. You will still be able to apply them afterwards.

## How to apply new patch version?

* Use previous patch to revert changes.
* Download new patch version and apply as described before.

## The results

The results are really astonishing! These are real word values:

    ```text
    VM   full      diff 1day   diff 2days   diff 3days    diff 4days
    1.   39.10GB   41MB        47MB         51MB          55MB
    2.   96.84GB   1.07GB      1.38GB       1.43GB        1.68GB
    3.   83.95GB   1.68GB      2.66GB       3.69GB        4.25GB
    4.   9.19GB    76KB        76KB         166MB         198MB
    ```

You see the differences. The diff sizes strictly depends on the use of the VMs. Using differential backups I have backups from last month (full backup once a week, differential daily)

## Is it stable?

Yes, it is. This extensions uses **xdelta3** as differential backup tool, which proven to be well tested and stable. I use it for about 9 months on 4 different Proxmox based servers. No problems so far.

However, if you happen to be paranoidal about backups… You should consider running following script. The script simply tries to verify all differential backups. I recently updated the script to support new VMA archive. So now you can verify backups all supported backups.

    ```bash
    ./pve-verify-backups <backup-dir>
    ```

## FAQ

In case of any problems applying or reverting patches you can always simple revert back to stock. Simply reinstall modified packages:

        apt-get --reinstall install pve-manager pve-container qemu-server libpve-storage-perl 

Then you can try to reapply patches once again.

In order to remove all leftovers you have to edit */etc/pve/vzdump.cron* and remove *fullbackup* switch from *vzdump* command line.

## Changelog

* v1: initial public release with support for PVE2.2 and PVE2.3 (2013-03-05)
* v2: improved kvm backup size and speed for PVE2.3 (2013-03-08)
* v3: added support for PVE3.0 (2013-06-02)
* v3': updated pve-verify-backups to support VMA archives (2013-06-06)
* v3'': updated patches to support PVE3.1 (2013-08-24)
* v3'': updated xdelta3 to 3.0.6. More info about changes: http://xdelta.org/ (2013-08-24)
* v3'': updated patches to support PVE3.2 (2014-03-15)
* v3'': added FAQ (2014-04-30)
* v3'': updated patches to support PVE3.3 (2014-09-23)
* v3'': updated patches to support PVE3.4 (2015-02-26)
* v3'': updated patches to support PVE4.0 (2015-11-07)
* v3'': updated patches to support PVE4.1 (2016-01-22)
* v3'': updated patches to support PVE4.1-22 (2016-04-13)
* v3'': updated patches to support PVE4.2 (2016-05-02)
* v3'': updated patches to support PVE3.4-14 (2016-08-02)
* v3'': updated patches to support PVE4.2-17 (2016-08-02)
* v3'': updated patches to support PVE4.3 (2016-10-24)
* v3'': updated patches to support PVE4.4 (2017-01-23)
* v3'': updated patches to support PVE4.4-13 (2017-04-10)
* v3'': updated patches to support PVE5.0 (2017-07-20)
* v3''': updated patches to support PVE5.1 (2018-03-13)
* Please take a look at commit stream :)

## Detailed list of changes

1. pve-xdelta3:
    * repackaged from http://packages.debian.org/wheezy/xdelta3
    * added support for lzop
    * removed all python references
2. vzdump:
    * added “fullbackup” option
3. qmrestore and vzrestore:
    * added support for differential backups
4. PVE.dc.BackupEdit:
    * added controls for maxfiles and fullbackup

## Author

Kamil Trzciński, 2013-2018

## You find it valuable?

[Buy me a Beer](https://www.paypal.me/ayufanpl)

## License

MIT
