# Configuring-Thonny-Remote-Interpreter
How to configure Thonny on one platform to program on a remote platform. For example, using Thonny on Windows to program Python on a Raspberry PI Zero.

## Background
There are Raspberry PI's not suitable to run a GUI that would support local Thonny. Examples are Raspberry PI PICO and Raspberry PI Zero W. Thonny can be configured as the program development IDE to these platforms from a separate host. This article describes how to set this up

> NOTE: At the present time Thonny does not support debugging of code over Remote SSH.

At the moment, this article does address how to do the following:

- Remote Access to PICO - This is a special circumstance since PICO runs firmware rather than OS. You can find out how to use a remote Thonny at this page [Getting started with Raspberry Pi Pico](https://projects.raspberrypi.org/en/projects/getting-started-with-the-pico/2)
- Remote Access to ZERO without Wi-Fi (No W) - Need to look at that. This document assumes the ZERO W is accessible from the remote host.

The following assumes the following:

- The ZERO W is running the Raspberry PI 32 bit 'Bulleye' OS; the Lite version. If you are installing the version with the full GUI, you don't need remote access
- Wi-Fi and SSH has been enabled and are working
- There is a username and password to use. This is normally the development account.
- We are not trying to install/run remote access to a Virtual Environment (VENV). This seems to be a requirement for Bullseye but as far as I know, Bullseye is not available for the ZERO yet. VENV is a good idea and I may come back to address it hear since Thonny supports it.

## Version of Thonny to Use
The P400 Raspberry PI Bullseye came with Thonny 4.0.2 installed. On the [Thonny](https://thonny.org/) home page, the latest release is 4.1.4. If you hover over the 'Download version 4.1.4 ..' panel, a popup window shows that for Debian and Raspberry PI you should use `sudo apt install thonny`. When I did this, Thonny 4.0.1, an older version was installed which I removed.

I found that the 4.0.2 version I was running was installed locally at ~/apps/thonny.

On the [Thonny install page](https://github.com/thonny/thonny/releases/tag/v4.1.4) it says to run `thonny-4.1.4.bash`. This script seems to install Thonny into ~/apps/thonny as well. I have not had the nerve to try installing 4.1.4 using the BASH script yet until I can confirm I can restore 4.0.2 if needed.

On the [Introduction to Raspberry Pi Pico guide](https://projects.raspberrypi.org/en/projects/introduction-to-the-pico/2) it says Thonny is already installed but to update it, run `sudo apt update && sudo apt upgrade -y` which does down update the version.

I mention all of this because in Thonny 4.1.0 they fixed "Fix remote Python 3 (SSH) connection error" which I have run into. So if you run into problems with an older version, you may wish to consider upgrading.

## Log into the ZERO W from the Remote Host
The ZERO has to be first configured to accept the Thonny connection from the remote host
```
$ ssh paul@pizero.local
```

## Verify Python

Confirm that a Python3 is installed.
```
$ python --version
Python 3.9.2
```
If you want a newer version of Python, now is the time to upgrade it.

Now, there can be a lot of confusion on which Python you are running especially if you upgraded and not removed older versions. In Thonny you will be asked to provide a pointer to the Python executable on the ZERO. Here is a 'clean' setup where 'Python' points to 'Python3' which points to the executable 'Python3.9'
```
$ whereis python
python: /usr/bin/python3.9-config /usr/bin/python /usr/bin/python3.9 /usr/lib/python2.7 /usr/lib/python3.9 /etc/python3.9 /usr/local/lib/python3.9 /usr/include/python3.9 /usr/share/man/man1/python.1.gz

$ ls -lh /usr/bin/python
lrwxrwxrwx 1 root root 7 Mar  2  2021 /usr/bin/python -> python3

$ ls -lh /usr/bin/python3
lrwxrwxrwx 1 root root 9 Apr  5  2021 /usr/bin/python3 -> python3.9

$ ls -lh /usr/bin/python3.9
-rwxr-xr-x 1 root root 4.5M Mar 11  2021 /usr/bin/python3.9
```
What all this means is that you can take the default of `/usr/bin/python` in Thonny. If you have a mix of different versions, you will likely have to select the real executable interpreter. In this case it would be `/usr/bin/python3.9`.

## Verify PIP
You will need PIP for the next step of installing JEDI. If you are using a fresh Bullseye OS install, PIP is not installed. I'll explain a bit based on my own experiences. 
One has to be carefull about both installing PIP and running PIP. For example, running `pip` versus `sudo pip` will install the Python package in different directories. The former is for the users own use; the latter system wide. You need to be consistant. The approach I'm taking here is to install PIP for the local users use.

First check to see if PIP is installed
```
$ python3 -m pip --version
/usr/bin/python3: No module named pip
```
If you get the above message, install PIP

### SUDO APT INSTALL PYTHON3-PIP (NOT RECOMMENDED)

There are many install guides for PIP. One I found was [How to install pip on the Raspberry Pi](https://pimylifeup.com/raspberry-pi-pip/). A site where I get a lot of good information from but they seem to take the approach of installing packages for the system as opposed to the user. This can create problems when different Python scripts require different versions of the same package. This problem is overcome with Python Virtual Environments and one of the reasons I chose to install PIP in the local user.
```
$ sudo apt install python3-pip
```
After this you can test the install
```
$ python -m pip --version
pip 20.3.4 from /usr/lib/python3/dist-packages/pip (python 3.9)
# and
$ pip --version
pip 24.0 from /home/paul/.local/lib/python3.9/site-packages/pip (python 3.9)
```
So, you can see the issue here. There is a differences in version and the directory structure they are working in.

### SUPPORTED METHODS TO INSTALL PIP
I'm using [Installing PIP](https://pip.pypa.io/en/stable/installation/) which lists the supported methods for installing PIP from the [PIP Documentation V24](https://pip.pypa.io/en/stable/). The method I chose to use was to download ```get-pip.py`` since I did not do a fresh Python install. I open the URL then saved the page to `get-pip.py`
```
$ ls -hl
total 2.6M
-rw-r--r-- 1 paul paul   11 Mar  2 10:26 date_new_install.txt
-rw-r--r-- 1 paul paul 2.6M Mar  2 16:46 get-pip.py
$
```
Running the install
```
$ python get-pip.py
Defaulting to user installation because normal site-packages is not writeable
Looking in indexes: https://pypi.org/simple, https://www.piwheels.org/simple
Collecting pip
  Downloading https://www.piwheels.org/simple/pip/pip-24.0-py3-none-any.whl (2.1 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 2.1/2.1 MB 553.2 kB/s eta 0:00:00
Installing collected packages: pip
Successfully installed pip-24.0
$
```
Verifying the install
```
$ python -m pip --version
pip 24.0 from /home/paul/.local/lib/python3.9/site-packages/pip (python 3.9)
paul@piZERO:~ $ pip --version
pip 24.0 from /home/paul/.local/lib/python3.9/site-packages/pip (python 3.9)
$
```
We now have a consistant PIP installation.

## Install the JEDI Python Package
Thonny remote requires the [JEDI package](https://jedi.readthedocs.io/en/latest/). To ensure you have it, run the following
```
$ pip install jedi
```
If JEDI is not installed on the ZERO then you will receive the following error from Thonny on the remote hose.
```
Python 3.9.2 (/usr/bin/python) (/usr/bin/python on piZERO.local)
>>> PROBLEM IN THONNY'S BACK-END: Exception while handling 'highlight_occurrences' (ModuleNotFoundError: No module named 'jedi').
See Thonny's backend.log for more info.
```
## Configure Thonny for Remote Interpreter 
On the remote host (not ZERO) start Thonny
In Python, select Tools-> Options-> Interpreters. Enter the information need to configure the connection:
- From the pull down, select 'Remote Python 3 (SSH)'
- Enter the host name either as a name or IP address
- Select user password (SSH) authentication. In this example 'password' is used. You will be asked for the password on the next screen
- Interpreter. Use the directory use discovered above in Verify Python.
- Leave the 'shebang scripts' default checked.

![Screenshot from 2024-03-01 16-56-37](https://github.com/DS256/Configuring-Thonny-Remote-Interpreter/assets/32932990/e1bd2de3-e64d-4f05-91be-906495d611b0)

You will next be prompted for the ZERO user password which can be saved for future accesses

![Screenshot from 2024-03-01 16-58-32](https://github.com/DS256/Configuring-Thonny-Remote-Interpreter/assets/32932990/60aabeb7-c0b9-4da1-8974-0ec9a49f92fd)

You should now be configured to a) Use Thonny on your remote host connecting to b) your ZERO W where the program is run. Your can perform a simple test to confirm everything is working. Note the desciprition of the Remote Python Interpreter at the bottom of Thonny window.

![Screenshot from 2024-03-01 17-02-09](https://github.com/DS256/Configuring-Thonny-Remote-Interpreter/assets/32932990/fb263d22-b1f5-40f3-8bdf-527ca1848555)

## Additional Notes
### Saving Files
If you open the files panel (View->Files) you will be presented with a subpanel for files on your local computer and one for the ZERO where you logged in. The directories presented can be changed by clicking on the directories. By clicking on the upper right 3 bars you can also upload and download files. However, there can be confusion if, as in the example, the directory names are the same on both computers.

When you need to save a file, you will be presented if you want to save the file locally or on the ZERO
![Screenshot from 2024-03-02 11-58-03](https://github.com/DS256/Configuring-Thonny-Remote-Interpreter/assets/32932990/f39e6525-47ee-48a5-aaaa-9eeb2cc43a28)

### Confirming Remote Configuration
You can always confirm Thonny is still accessing the remote ZERO W by pressing the STOP/RESET BACKEND red button in the toolbar. This will display the interpreter.

![Screenshot from 2024-03-02 12-07-35](https://github.com/DS256/Configuring-Thonny-Remote-Interpreter/assets/32932990/96d7c768-6d00-4535-a1c0-39b0cf9bc9c5)




