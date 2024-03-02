# Configuring-Thonny-Remote-Interpreter
How to configure Thonny on one platform to program on a remote platform. For example, using Thonny on Windows to program Python on a Raspberry PI Zero.

There are Raspberry PI's not suitable to run a GUI that would support local Thonny. Examples are Raspberry PI PICO and Raspberry PI Zero W. Thonny can be configured program development to these platforms from a separate host. This article describes how to set this up.

At the moment, this article does address how to do the following:

- Remote Access to PICO - This is a special circumstance since PICO runs firmware rather than OS. You can find out how to use a remote Thonny at this page [Getting started with Raspberry Pi Pico](https://projects.raspberrypi.org/en/projects/getting-started-with-the-pico/2)
- Remote Access to ZERO without Wi-Fi (No W) - Need to look at that. This document assumes the ZERO W is accessible from the remote host.

The following assumes the following:

- The ZERO W us running the Raspberry PI 32 bit 'Bulleye' OS, the Lite version. If you are installing the version with the full GUI, you don't need remote access
- Wi-Fi and SSH has been enabled and is working
- There is a username and password to use
- We are not trying to install/run remote access to a Virtual Environment (VENV). This seems to be a requirement for Bullseye but as far as I know, Bullseye is not available for the ZERO yet.

## Log into the ZERO W from the Remote Host
The ZERO has to be first configured to accept the Thonny connection from the remote host
```
$ ssh paul@pizero.local
```

## Verify Python

Confirm that a Python3 is installed
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
Next check to make sure PIP is installed
```
$ python3 -m pip --version
/usr/bin/python3: No module named pip
```
If you get the abnove message, install PIP
```
$ sudo apt install python3-pip
```
After this you should get a message like this
```
$ python -m pip --version
pip 20.3.4 from /usr/lib/python3/dist-packages/pip (python 3.9)
```
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
In Python, select Tools-> Options-> Interpreters. Enter the information for the ZERO W.

![Screenshot from 2024-03-01 16-56-37](https://github.com/DS256/Configuring-Thonny-Remote-Interpreter/assets/32932990/e1bd2de3-e64d-4f05-91be-906495d611b0)

You will be prompted for the password which can be saved for future accesses

![Screenshot from 2024-03-01 16-58-32](https://github.com/DS256/Configuring-Thonny-Remote-Interpreter/assets/32932990/60aabeb7-c0b9-4da1-8974-0ec9a49f92fd)

You should now be configured to a) Use Thonny on your remote host connecting to b) your ZERO W where the program is run. Your can perform a simple test to confirm everything is working. Note the desciprition of the Python Interpreter at the bottom of Thonny window.

![Screenshot from 2024-03-01 17-02-09](https://github.com/DS256/Configuring-Thonny-Remote-Interpreter/assets/32932990/fb263d22-b1f5-40f3-8bdf-527ca1848555)

## Additional Notes

If you open the files panel (View->Files) you will be presented with a subpanel for files on your local computer and one for the ZERO where you logged in. The directories presented can be changed by clicking on the directories. By clicking on the upper right 3 bars you can also upload and download files. However, there can be confusion if, as in the example, the directory names are the same on both computers.

When you need to save a file, you will be presented if you want to save the file locally or on the ZERO
![Screenshot from 2024-03-02 11-58-03](https://github.com/DS256/Configuring-Thonny-Remote-Interpreter/assets/32932990/f39e6525-47ee-48a5-aaaa-9eeb2cc43a28)



