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
  * setup-x86_64.exe -q -P wget,nano,python2,python2-pip,python2-devel,automake,gcc,make,libpq-devel,libpq5,zlib,zlib-devel,libjpeg8,libjpeg-devel
  * Choose any Available Download Site and click Next
  * cygwin64 will automatically start installing dependencies of NetBox

(Dev note: If you don't install libpq5, importing the psycopg2 module WILL NOT WORK with a really vague error)
(Dev note: We may need gcc-g++ but so far not needed)
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
* Install the required python modules
  * pip2 install -r requirements.txt

## Configuration
* Open Cygwin64 Terminal
* Change to the "current" directory
  * cd /cygdrive/c/netbox/current
* Copy the example configuration file into place
  * cp netbox/netbox/configuration.example.py netbox/netbox/configuration.py
* Open configuration.py with your preferred editor and set the following variables: ALLOWED_HOSTS, DATABASE, SECRET_KEY - if you need help consult main NetBox docs

## Run Database Migrations
* Open Cygwin64 Terminal
* Change to the "current/netbox" directory
  * cd /cygdrive/c/netbox/current/netbox
* Run the migrations
  * ./manage.py migrate
## Create a Super User
Create a super user account.
```
# ./manage.py createsuperuser
Username: admin
Email address: zorlin@gmail.com
Password:
Password (again):
Superuser created successfully.`
```

## Collect Static Files
```
# ./manage.py collectstatic --no-input

You have requested to collect static files at the destination
location as specified in your settings:

    /opt/netbox/netbox/static

This will overwrite existing files!
Are you sure you want to do this?

Type 'yes' to continue, or 'no' to cancel: yes
```

## Load Initial Data (Optional)
Populate netbox with some initial example data. HIGHLY recommended.
```
# ./manage.py loaddata initial_data
Installed 43 object(s) from 4 fixture(s)
```

## Test the Application
* Start up NetBox in testing mode
```
 ./manage.py runserver 0.0.0.0:8000 --insecure
Performing system checks...

System check identified no issues (0 silenced).
June 09, 2017 - 08:54:45
Django version 1.11.2, using settings 'netbox.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
```
* Open your browser to http://localhost:8000/

## Congratulations!
If all has gone well so far, you should see NetBox! If not, make sure you have 'localhost' in ALLOWED_HOSTS.

Running NetBox in this setup is not really suitable for production use, however.

# Web Server
## Web Server Installation
We'll be using nginx, gunicorn and supervisord. You will want to replace "wings" with your Windows username throughout this section.

## nginx Installation
* Create the directory C:\nginx\
* Visit http://nginx.org/en/download.html and download the latest Mainline version of nginx for Windows.
* Extract it into C:\nginx (nginx.exe should be @ C:\nginx\nginx.exe)
* Open C:\nginx\conf\nginx.conf in your favourite editor.
* Replace the entire server {} block with the following:
```
server {
    listen 80;

    server_name localhost;

    client_max_body_size 25m;

    location /static/ {
        alias C:/netbox/current/netbox/static/;
    }

    location / {
        proxy_pass http://127.0.0.1:8001;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        add_header P3P 'CP="ALL DSP COR PSAa PSDa OUR NOR ONL UNI COM NAV"';
    }
}
```
* Save and exit the file
* We'll come back to nginx later.

## gunicorn Installation
* Install gunicorn
  * pip2 install gunicorn
* Open C:\netbox\current\gunicorn_config.py in your favourite editor (note, it doesn't exist yet)
```
command = '/usr/bin/gunicorn'
pythonpath = '/cygdrive/c/netbox/current/netbox'
bind = '127.0.0.1:8001'
workers = 4
user = 'wings'
```

Again, we'll come back to gunicorn later.

## supervisord Installation
* Install supervisord
  * pip2 install supervisor
* Create conf and log directories
```
mkdir -p /etc/supervisor/conf.d
mkdir -p /var/log/supervisor/
```
* Open /etc/supervisor/supervisord.conf with your favourite editor and fill it with the contents of this gist, then save...
  * https://gist.github.com/Zorlin/4471e6609326390fc4d35c0502be1929
  * TODO: clean this up a bit?
* Open /etc/supervisor/conf.d/netbox.conf with your favourite editor (also doesn't exist yet), fill it with this and save...
```
[program:netbox]
command = gunicorn -c /cygdrive/c/netbox/current/gunicorn_config.py netbox.wsgi
directory = /cygdrive/c/netbox/current/
user = wings
```

## Testing it all out
We're going to do a quick dry-run to make sure everything works.

* Boot up gunicorn via supervisord
  * /usr/bin/supervisord -n -c /etc/supervisor/supervisord.conf
* Visit localhost:8001 in your browser. If the page loads but has no styling, that's good, don't worry!
* Go to C:\nginx and open nginx.exe either in the command line or file explorer. It should pop up a black window that immediately closes.
* Visit localhost:80 in your browser. Everything should look perfect at this point. If so, congratulations!

If that worked, shut everything back off.
* Switch to the Cygwin64 Terminal and STOP supervisord by hitting CTRL-C.
* Kill any nginx processes that are running.

# Making it permanent - Services!
Time to make your installation a bit more permanent. This will allow you to manage Nginx and Supervisord as services, allowing them to start on boot and other nice stuff. We will install and run supervisord first, then add nginx.

## Install cygwin run service utility
Before we proceed, we'll need to add a utility to cygwin called cygrunsrv.
* In an administrator command prompt, go back to your Downloads folder
* Re-run the cygwin installer with this command
  * setup-x86_64.exe -q -P cygrunsrv

## supervisord
* In an administrator command prompt, go to C:\cygwin64\bin
* Run this command to install supervisord as a service
  * cygrunsrv --install supervisord --path /usr/bin/python --args "/usr/bin/supervisord -n -c /etc/supervisor/supervisord.conf"
* We now need to revisit our configuration as it changes at this point - services run under the SYSTEM user.
* Open /cygdrive/c/netbox/current/gunicorn_config.py and replace "wings" (or your user) with "SYSTEM".
* Open /etc/supervisor/conf.d/netbox.conf and replace "wings" (or your user) with SYSTEM."
* In an administrator command prompt, run NET START supervisord - hopefully the service will start successfully.
* Test the service-ified supervisord by visiting localhost:8001. You should see an unstyled version of NetBox, like you did earlier.

## nginx
(Adapted from https://stackoverflow.com/questions/10061191/add-nginx-exe-as-windows-system-service-like-apache)
* Grab the latest Windows Service Wrapper from GitHub - https://github.com/kohsuke/winsw/releases - you want one called WinSW.NET4.exe
* Drag that into your C:\nginx directory and then rename it to nginxservice.exe
* With your favourite editor, open C:\nginx\nginxservice.xml. Fill it with this and save...
```
<service>
  <id>nginx</id>
  <name>nginx</name>
  <description>nginx</description>
  <executable>c:\nginx\nginx.exe</executable>
  <logpath>c:\nginx\</logpath>
  <logmode>roll</logmode>
  <depend></depend>
  <startargument>-p</startargument>
  <startargument>c:\nginx</startargument>
  <stopexecutable>c:\nginx\nginx.exe</stopexecutable>
  <stopargument>-p</stopargument>
  <stopargument>c:\nginx</stopargument>
  <stopargument>-s</stopargument>
  <stopargument>stop</stopargument>
</service>
```
* Finally, as an administrator, CD to c:\nginx and run "nginxservice.exe install"
* Run "net start nginx"

Now visit localhost:80 and check that everything works. If it does, congratulations! You have a full NetBox install running on Windows.

(This should still be considered experimental. For example, nginx on Windows is only in beta! Still, enjoy.)

Hopefully your browser now looks like this :)
![A demon of NetBox on Windows in production mode](http://i.imgur.com/cGk0Xyb.png)
