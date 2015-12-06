# install-visualisations-from-scratch
The steps needed to install the UCC Organism visualisation on Odroids and MacMinis from scratch

[STILL UNDER DEVELOPMENT]

We will be running this on a completely virgin copy of MACOSX EL CAPITAN. We expect it will work also on future versions of OSX.

Open Terminal

[A note on Terminal: normally the default shell on MACOSX (bash) shows the current directory your in as well as the name of the volume and user logged in. So for example on our test system the command prompt look like this:

CIIDResearch:ucc-organism cr$ [some command here]

Specifying we are in on the “CIIDResearch” volume in the “ucc-organism” folder with the user “cr”.
For the sake of clarity we will here type the full path from the user’s home folder, but leave out volume and user name, so the same prompt here will be:

~/ucc/ucc-organism$ [some command here]

We will keep this nomenclature throughout this document]

download and install nodejs from the www.nodejs.org website. Avoid using Homebrew for now. It will work, but there might be problems later on when trying to update Node.js.
 
check that Node actually installed
~$ node —version
install some packages we need (we are installing them globally, hence sudo)
~$ sudo npm install -g npm
~$ sudo npm install -g coffee-script
~$ sudo npm install -g forever
~$ sudo npm install -g browserify
~$ sudo npm install -g brfs

Create the directory ucc

~$ mkdir ucc

go into the directory you just created

~$ cd ucc

This is the root folder for everything we will be doing after this.

———
Installing the test server

Install the source
~/ucc$ git clone https://github.com/ucc-organism/uccorg-backend
~/ucc$ cd uccorg-backend
~/ucc/uccorg-backend$ npm install

Run the server
~/ucc/uccorg-backend$ coffee uccorg-backend.coffee dev.json

It will start writing a lot of lines like this:
[picture from terminal]

Open browser and check look at http://localhost:8080, you will look like something like this:
[picture from safari]

———
Install the ucc-organism visualisation for local testing

open a second terminal window and make sure you’re in the root folder of the UCC Project (~/ucc)
Install and prepare the client build
~/ucc$ git clone https://github.com/UCC-Organism/ucc-organism
~/ucc$ cd ucc-organism
~/ucc/ucc-organism$ npm install
~/ucc/ucc-organism$ ./bundle.sh

Back in the first (backend)terminal window start the test server as described before.
In the second (frontend) window change directory to the build directory and run a simple server to connect to the visualisation.
~/ucc/ucc-organism$ cd build
~/ucc/ucc-organism/build$ python -m SimpleHTTPServer
Open your browser and connect it to http://localhost:8000

You should see this:
[picture of browser]

CTRL-C to close what’s running in both terminal windows.




MAKING THE VISUALISATION RUN ON ODROID
in the ucc folder make a folder called “buildsystem“
~$ cd ucc
~/ucc$ mkdir buildsystem

Download the Android SDK Tools via browser:
point your browser to https://developer.android.com/sdk/installing/index.html?pkg=tools
click “download the SDK now”
click the MacOSX package (in time of writing “android-sdk_r24.4.1-macosx.zip”) accept the license for it to download
move it to the ucc/buildsystem folder (you can place it somewhere else but our walkthrough is assuming it is there)
Download and install the Java JDK. We got it from http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html. Otherwise google “Java JDK”. We selected the 8u66 version. Not sure what is best, guess the most up to date will do.

back in the terminal (ucc root directory) navigate to the android folder and then the tools folder and run the tool
~$ cd ucc/buildsystem/android-sdk-macosx
~/ucc/buildsystem/android-sdk-macosx$ cd tools
~/ucc/buildsystem/android-sdk-macosx/tools$ . android (remember the dot)
This opens a window in your desktop environment.
Set the following variables and only those:
SDK Platform Tools 23.0.1
SDK Build-Tools 20
Android 5.0.1 API 21
[screen for android tool]
once downloaded and installed (it takes a while) close the tool

Download and install the Apache ANT zip from http://ant.apache.org/bindownload.cgi
Also move this to the ucc/buildsystem folder

clone the crosswalk git repository
~/ucc/buildsystem$ git clone https://github.com/UCC-Organism/crosswalk-odroid-ucc.git

Edit the file setenv.sh to point to you system’s Android SDK, ANT and XWALK directories (you can edit in textedit if you have desktop access to the specific document or nano as below). Note: Check that the ANT path has the name of the version you actually downloaded.
~/ucc/buildsystem$ cd crosswalk-odroid-ucc
~/ucc/buildsystem/crosswalk-odroid-ucc$ nano setenv.h
[picture of setenv.sh]



COMPILING APK::::
——————————
Now that we have specified where the compiler can find its directories we can compile.

First we make a new build of the ucc-organism app.
We want to first give it the right version number. Go to the ucc/ucc-organism folder and open the manifest.json file (you can do this with textedit or nano as below)

~/ucc/buildsystem/crosswalk-odroid-ucc$ cd ~/ucc/ucc-organism
~/ucc/ucc-organism$ nano manifest.json

we change the name according to the following format: YYYYMMDDVV where YYYY is year (2015), MM is month (12), DD is day (06) and VV is version of that day (usually just 01).
So for the first version of this day of writing (December 6th, 2015) it is 2015120601.

[picture of nano with version number]

Then we hop back to the crosswalk compiler folder in the build system

~/ucc/ucc-organism$ cd ~/ucc/buildsystem/crosswalk-odroid-ucc

First you set the environment (you will have to do this every time you open a new terminal window to compile. It’s enough to do it once per terminal window session though)

~/ucc/buildsystem/crosswalk-odroid-ucc$ source sentenv.sh

Then you run the compiler with the following command: python make_apk.py --enable-auto-update --enable-remote-debugging --manifest=[path-to-your-apps-manifest]
Notice that in the terminal command below the part [path-to-your-apps-manifest] is substituted with the actual manifest.json in the build folder of our ucc-organism folder:

~/ucc/buildsystem/crosswalk-odroid-ucc$ python make_apk.py --enable-auto-update --enable-remote-debugging --manifest=~/ucc/ucc-organism/build/manifest.json

This should successfully compile the apk.

You find the newly compiled apk file in the same crosswalk-odroid-ucc folder as you compiled from, in this case it is called ucc-organism-2015120601.apk

Next we need to get it on the Odroid.


Installing on SDCARD for ODROID
Download the current ODROID-OS-IMAGE [https://drive.google.com/open?id=0BytYPd68_1xNfkNPWlU2NXh1enNaYmliQks5VVV4ZnZGZkUyaXM5VXNkZEZaamZNQlpZSmM] - this will take a while (~900MB)

To install it on an SDCARD follow the description here [link]


once it is installed and you have booted the Odroid (with a screen attached) to check that  it is working, we need to update the apk via the apk updater.

Installing the APK updater
