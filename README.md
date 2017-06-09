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

## Database creation
## Test your connection

# NetBox
## Installation
* Download cygwin64 from https://cygwin.com/install.html
* Run the cygwin64 installer, and keep it around for later usage
** Install from Internet.
** You can install it anywhere you want but for this guide we'll be using C:\cygwin64 (the default) and installing for All Users.
** Leave Local Package Directory alone or change it. Up to you.
** Click next a couple of times till the main installer window opens.
** Choose Direct Connection unless you have a reason not to.
** Pick whatever mirror you want and hit next.

## Option A: Download a release

## Install Python packages
