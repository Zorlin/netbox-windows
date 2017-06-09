# netbox-windows
This repository contains a guide for building and running NetBox on Windows. It is a work in progress; please submit a bug report for any issues you encounter.

# SUPERALPHA
This documentation needs a lot of improvement and validation. Please don't use this unless you know what you're doing for now! It's highly experimental in **multiple ways**. It is intended to follow roughly the flow of the original NetBox documentation, and the tooling and software chosen is supposed to match the Linux way as closely as possible.

Please also note that this is based on the latest version of python2 and has NOT been tested properly on python3. Use at your own risk!

# Why not Bash on Windows?
Various reasons. I would like to eventually explore Bash on Windows and maybe port this to it, but Bash/Ubuntu on Windows is not available for Windows 7 and is thus not as flexible.

# PostgreSQL
## Installation
* Visit the PostgreSQL website for Windows - https://www.postgresql.org/download/windows/
* Choose Interactive installer by EnterpriseDB - http://www.enterprisedb.com/products/pgdownload.do#windows
* At the download page, choose the latest version of PostgreSQL and Windows x86-64, then hit Download Now.
* Once the installer has finished downloading, run it. When prompted in the installer, set a nice superuser password you'll record or remember. Leave all other settings as defaults, but at the end do not choose to run Stack Builder at exit. Hit Finish.

## Database creation
* Open pgAdmin 4 (Start Menu -> Programs -> PostgreSQL 9.6 -> pgAdmin 4 or just find it by typing in the Start Menu).
* Expand Servers on the left, then double-click the single PostgreSQL 9.6 entry. Enter your superuser password from earlier.
* Right click Login/Group Roles, then Create -> Login/Group Role...
* Set Name to netbox, then click the Definition tab and set a password for it.
* Click the Privileges tab and set "Can login?" to Yes.
* Hit Save.
* Right click Databases, then Create -> Database...
* Set Name to netbox, then change Owner to the netbox user.
* Hit Save.

## Test your connection
* Open SQL Shell (psql)
* Leave server as localhost (just hit enter)
* Type "netbox" as the database
* Leave port as 5432 (just hit enter)
* Type "netbox" as the user
* Enter your password
* If successful, you should see a prompt like this:
netbox=>

# NetBox
## Installation
* Download cygwin64 from https://cygwin.com/install.html
* Open a command prompt as Administrator
* Change directories to the place you just downloaded
  * eg: cd C:\Users\Zorlin\Downloads
* Run the cygwin64 installer, asking it to install all required packages.
  * setup-x86_64.exe -q -P python2,python2-pip,wget
  * Choose any Available Download Site and click Next
  * cygwin64 will automatically start installing dependencies of NetBox

## Option A: Download a release
* Open Cygwin64 Terminal
* Create and then enter a directory
  * mkdir /cygdrive/c/netbox
  * cd /cygdrive/c/netbox
* Download the latest stable netbox release from GitHub and place it in the C:\netbox directory
* Extract it to its own directory (eg: C:\netbox\netbox-2.0.5)
* Open a command prompt as Administrator
* Change directories to the netbox root
  * eg: cd C:\netbox
* Make a symbolic link from netbox-2.0.5 to current
  * mklink /D current netbox-2.0.5

## Install Python packages
* Open Cygwin64 Terminal
* Change to the "current" directory
  * cd /cygdrive/c/netbox/current
