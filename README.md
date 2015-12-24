# install-visualisations-from-scratch
The steps needed to install the UCC Organism visualisation on Odroids and MacMinis from scratch

[STILL UNDER DEVELOPMENT]

We will be running this on a completely virgin copy of MACOSX EL CAPITAN. We expect it will work also on future versions of OSX.

Open Terminal

[A note on Terminal: normally the default shell on MACOSX (bash) shows the current directory you are in as well as the name of the volume and user logged in. So for example on our test system the command prompt look like this:

![Normal Terminal look](/images/terminal.png)

Specifying we are in on the “CIIDResearch” volume in the “ucc-organism” folder with the user “cr”.
For the sake of clarity we will here type the full path from the user’s home folder, but leave out volume and user name, so the same prompt here will be:

```
~/ucc/ucc-organism$ [some command here]
```

We will keep this nomenclature throughout this document]

# Installing Node.js and dependencies
download and install Node.js from the www.nodejs.org website. Avoid using Homebrew for now. It will work, but there might be problems later on when trying to update Node.js.
 
check that Node actually installed. It should output the current version number like `v4.2.1`

```
~$ node --version
```

install some packages we need (we are installing them globally, hence sudo)
```
~$ sudo npm install -g npm
~$ sudo npm install -g coffee-script
~$ sudo npm install -g forever
~$ sudo npm install -g browserify
~$ sudo npm install -g brfs
```

Create the directory ucc

```
~$ mkdir ucc
```

go into the directory you just created

```
~$ cd ucc
```

This is the root folder for everything we will be doing after this.

# Installing the test server

Install the source for the backend server
```
~/ucc$ git clone https://github.com/ucc-organism/uccorg-backend
~/ucc$ cd uccorg-backend
~/ucc/uccorg-backend$ npm install
```

Run the server

```
~/ucc/uccorg-backend$ coffee uccorg-backend.coffee dev.json
```

It will start writing a lot of lines like this:
![Backend Running](/images/backendinterminal.png)

Open a browser and point it to `http://localhost:8080`. You will see something like this:
![Backend running in browser](/images/backendinbrowser.png)

Kill the backend in the terminal with CTRL-C

# Install the ucc-organism visualisation for local testing

open a second terminal window and make sure you’re in the root folder of the UCC Project (~/ucc)
Install and prepare the client build

```
~/ucc$ git clone https://github.com/UCC-Organism/ucc-organism
~/ucc$ cd ucc-organism
~/ucc/ucc-organism$ npm install
~/ucc/ucc-organism$ ./bundle.sh
```

Back in the first (backend) terminal window start the test server as described before.
In the second (frontend) terminal window change directory to the build directory and run a simple server to connect to the visualisation.

```
~/ucc/ucc-organism$ cd build
~/ucc/ucc-organism/build$ python -m SimpleHTTPServer
```

Open your browser and connect it to `http://localhost:8000`

You should see something like this:
![Visualisation running in browser](/images/visualisationinbrowser.png)

CTRL-C in each terminal window to close the backend server and the frontend http server.

# Making the visualisation run on Odroid
In the ucc root folder make a folder called “buildsystem“

```
~$ cd ucc
~/ucc$ mkdir buildsystem
```

Download the Android SDK Tools via browser:
* point your browser to https://developer.android.com/sdk/installing/index.html?pkg=tools
* click “download the SDK now”
* click the MacOSX package (in time of writing “android-sdk_r24.4.1-macosx.zip”) and accept the license for it to download
* unpack and move the unpacked folder to the ucc/buildsystem folder (you can place it somewhere else but this walkthrough is assuming it is in that location)
* Download and install the Java JDK. We got it from http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html. Otherwise google “Java JDK”. We selected the 8u66 version (note: at time of writing 8u66 was listed *after* 8u55).
* Download and unpack the Apache ANT zip from http://ant.apache.org/bindownload.cgi
*Also move this unpacked folder to the ucc/buildsystem folder

back in the terminal (ucc root directory) navigate to the android folder and then the tools folder and run the tool (remember the dot)

```
~/ucc$ cd buildsystem/android-sdk-macosx
~/ucc/buildsystem/android-sdk-macosx$ cd tools
~/ucc/buildsystem/android-sdk-macosx/tools$ . android
```

This opens a window in your desktop environment.
Set the following variables and only those (uncheck anything else already checked):

* SDK Platform Tools 23.0.1
* SDK Build-Tools 20
* Android 5.0.1 API 21

![View of checked items in Android tool](/images/android_sdk_manager.png)

once the tool has downloaded and installed the 16 packages (it takes a while) close it.

Now go back to the ucc/buildsystem folder and clone the crosswalk git repository

```
~/ucc/buildsystem/android-sdk-macosx/tools$ cd ucc/buildsystem
~/ucc/buildsystem$ git clone https://github.com/UCC-Organism/crosswalk-odroid-ucc.git
```

Edit the file setenv.sh to point to you system’s Android SDK, ANT and XWALK directories (you can edit in textedit if you have desktop access to the specific document or nano as below). Note: Check that the ANT path has the name of the version you actually downloaded.

```
~/ucc/buildsystem$ cd crosswalk-odroid-ucc
~/ucc/buildsystem/crosswalk-odroid-ucc$ nano setenv.h
```

![setenv.sh](/images/setenv.sh.png)

# Compiling the APK
Now that we have specified where the compiler can find its directories we can compile.

First we make a new build of the ucc-organism app.
We want to first give it the right version number. Go to the ucc/ucc-organism folder and open the manifest.json file (you can do this with textedit or nano as below)

```
~/ucc/buildsystem/crosswalk-odroid-ucc$ cd ~/ucc/ucc-organism
~/ucc/ucc-organism$ nano manifest.json
```

we change the name according to the following format: YYYYMMDDVV where YYYY is year (2015), MM is month (12), DD is day (06) and VV is version of that day (usually just 01).
So for the first version of this day of writing (December 6th, 2015) it is 2015120601.

[picture of nano with version number]

Then we hop back to the crosswalk compiler folder in the build system

```
~/ucc/ucc-organism$ cd ~/ucc/buildsystem/crosswalk-odroid-ucc
```

First you set the environment (you will have to do this every time you open a new terminal window to compile. It’s enough to do it once per terminal window session though)

```
~/ucc/buildsystem/crosswalk-odroid-ucc$ source sentenv.sh
```

Then you run the compiler with the following command: ```python make_apk.py --enable-auto-update --enable-remote-debugging --manifest=[path-to-your-apps-manifest]```
Notice that in the terminal command below the part [path-to-your-apps-manifest] is substituted with the actual manifest.json in the build folder of our ucc-organism folder:

```
~/ucc/buildsystem/crosswalk-odroid-ucc$ python make_apk.py --enable-auto-update --enable-remote-debugging --manifest=~/ucc/ucc-organism/build/manifest.json
```

This should successfully compile the apk.

You find the newly compiled apk file in the same crosswalk-odroid-ucc folder as you compiled from, in this case it is called ```ucc-organism-2015120601.apk```

Next we need to get it on the Odroid.

# Installing on SDCARD for ODROID
Download the current version ODROID-OS-IMAGE from https://drive.google.com/open?id=0BytYPd68_1xNfkNPWlU2NXh1enNaYmliQks5VVV4ZnZGZkUyaXM5VXNkZEZaamZNQlpZSmM - this will take a while (~900MB)

## STEP 1 : Creating SDCARD

```
~$ diskutil list
~$ sudo diskutil unmount /dev/diskXs1
~$ sudo dd if=ucc_odroid.img of=/dev/rdiskX bs=1m
```
Open a new terminal window and cd to the folder where the downloaded Odroid image is (where you can find image named - ucc_odroid.img). In the following we are assuming you downloaded to the Download folder and unzipped it there.

```
~$ cd Downloads
```

Make sure the sdcard is inserted onto your Mac and then in terminal issue the command:

```
~/Downloads$ diskutil list
```

You should see a list of mounted disk on your system - identify the one which represents the sdcard (example: /dev/disk2)

![output of diskutil list](/images/diskutil_list.png)

Let’s say /dev/diskX represent the sdcard on your system - in terminal issue command:

```
~/Downloads$ sudo diskutil unmount /dev/diskXs1
```

This unmounts the sd-card so that we can write to it. Then in terminal issue command:

```
~/Downloads$ sudo dd if=ucc_odroid.img of=/dev/rdiskX bs=1m
```

This will copy verbatim the bits in the image file to the sdcard. This can take awhile. Make a cup of coffee while you wait about 10 minutes. 
Once it is done (you see the terminal prompt again) manually eject the card (through Finder or Explorer).

## STEP 2: Edit system.conf file on the SDCARD

![system.conf on SDCARD](/images/odroid_system_conf.png)

A. Re-insert the sdcard on your machine (MAC or PC)
B. Using Finder or Explorer open the file <SDCARD>/Organism/system.conf using a basic text editor
C. Editable elements in /Organism/system.conf are seen below

```
"client_id": String
```

> string literal that represents the identifier of the node / odroid. the value is an integer (ex: “3” or “11”) and defaults to “0”. Remember parentheses. Example: `“client_id”:”0”`

```
"api_server_url": String
```

> IP address of the data server, providing all necessary data to the apk. Slashes “/” *must* be escaped with a backslash “\” and include the port number (Example: `“api_server_url”:”http:\/\/192.168.1.160:8080”`). The port number is always 8080 (unless the backend developer changes it).
> When the server url string is empty - ex: `“api_server_url”:””` - the application defaults to pre-scripted data scenarios without trying to contact the server.

```
"apk_update_on_boot": String { “true” or “false” }
```

> configuration that informs the system to look for apk update at boot time. Example: `“apk_update_on_boot”:”true”`

```
"apk_updater_server": String (IP address)
```

> apk server IP where the system looks for updates (NOTE: slashes “/” *must* be escaped with a backslash “\” - Example: `"apk_updater_server":"http:\/\/node.variable.io"`)

```
"apk_update_interval": String
```

> string literal informing the system on the rate at which it should look for apk updates (updates intervals). 
> the format of the string is `“X:Y”` where X is an integer and Y a character in the range {’M’, ‘H’, ‘D’} -  [‘M’ - Minutes, ‘H’ - Hours, ‘D’ - Days] (Example: `"apk_update_interval":"3:M"` => 3 Minutes update interval)

```
"apk_updater_server_port": String (server port)
```

> open port of the apk server. `"apk_updater_server_port":"8088"`


```
"apk_updater_path": String (path on the sever)
```

> path on the apk server where the repository of apks reside (NOTE: slashes “/” *must* be escaped with a backslash “\” - ex: `"apk_updater_path":"\/ucc_organism"`)

```
"sleep_rtc_time": String
```

> string literal representing the time when the Odroid needs to shut down its connected display and enter its sleep cycle.
> the format of the string is `“XX:YY”` where `XX` stands for Hours integer and `YY` for the Minutes integer (ex: `"sleep_rtc_time":"19:00"` => odroid sleeps at seven o’clock PM)

```
"wakeup_rtc_time": String
```

> string literal representing the time when the Odroid needs to wake up and reopen its connected display.
> the format of the string is `“XX:YY”` where `XX` stands for Hours integer and `YY` for the Minutes integer (ex: `"wakeup_rtc_time":"8:00"` => odroid wakes up at eight o’clock AM)

```
"ucc_organism_deamon_wdt_minutes": Integer
```

> integer informing the system on the interval (in minutes) to keep monitoring the state of the running apk on the odroid. Example: `"ucc_organism_deamon_wdt_minutes":1` Please note the value is without parentheses.

```
"ucc_organism_deamon_timeout_max": Integer
```

> integer informing the system on the maximum numbers of apk state failures before restarting the whole apk altogether.  Example: `"ucc_organism_deamon_timeout_max":2` Please note the value is without parentheses.


## STEP 3: Insert the SDCARD in the ODROID and turn on it's power

The system will boot the os image and when completed, the apk starts automatically after less than 1 minute.

Note 1: Boot time may vary depending on the quality of the sdcard (we observed that lesser quality sdcard take more time than higher quality ones)

Note 2: If an update is available on the apk server and the "apk_update_on_boot" is set to true, the application will update automatically on boot and restart after successful update completion.

# Installing the APK updater (TODO)
