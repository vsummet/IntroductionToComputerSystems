# Setting up your Raspberry Pi to Complete your C Homework Assignments

Setup and log-in to your RPi as usual.  Make sure the network is active (use `ping 8.8.8.8` to make sure).

## Cleanup
The first thing we'll do is make sure our software is up to date.  This is the Linux equivalent of Windows updates except it is fast and doesn't require you to reboot multiple times.

First, check to make sure all the online sources we'll need are up-to-date:
```
prompt$ sudo apt-get update
```

Next, grab all the updated software for our version of Linux:
```
prompt$ sudo apt-get upgrade
```
If prompted, type `y' and then '[Enter]` to indicate you want to continue.  Upgrading the software will take awhile, but you should be able to see a text-based indicator of download and installation progress.

Finally, make sure all the old software has been removed:
```
prompt$ sudo apt-get autoremove
```
This shouldn't find anything to remove, but if it does, you can safely type `y` to confirm the removal.

## Setup

Now we'll move on to actually installing the software we need to do our work.  Fortunately, virtually all Linux systems come with `gcc` already installed and ready to go.  They also come with `nano`, a simple text-editor which you're already familiar with.

However, we need to install `git` and its supporting software.

```
prompt$ sudo apt-get install git
```
Type `y` and `[Enter]` to confirm if prompted.

Before we can push to github, we need to configure git so that it knows how to identify our code.  Substitute your name and email address in the following commands.  The email address does not need to match your GitHub credentials or your Rollins credentials.
```
prompt$ git config --global user.name "Your Name"
prompt$ git config --global user.email "Your Email"
```

# Developing
At this point, you should be ready to start developing your homework.  

Make a directory for your homework:
```
prompt$ mkdir hw
prompt$ cd hw
```
You can call this directory anything you want like `230` or `cms230`.  It's just somewhere to keep your work separate from all your other files from labs and other activities.

In your web browser, navigate to github.com and copy the URL of a repo you wish to clone (as you do at the start of every HW assignment).

On your Pi type:
```
prompt$ git clone URL-YOU-JUST-COPIED
```

This will create a directory containing all the code in your repo.  You can repeat this command with all your repos (including past HWs) as many times as you need to.

At this point, you can use the standard commands you are used to, including:
```
prompt$ gcc -Wall -Werror -o sorting sorting.c
prompt$ make
prompt$ python test.py
prompt$ git add .
prompt$ git commit -m "my final version"
prompt$ git push origin master
```

## Tips
We're doing a lot of work installing software which requires elevated privileges.  If you get an error about `permission denied`, check to make sure you included the necessary `sudo` command to elevate your user privileges.

I suggest opening and connecting two Putty/Terminal windows at the same time.  Yes, you can have multiple connections open at the same time.  Just remember that they're both controling the same piece of hardware.  In one window, open `nano` and edit your file and save it.  But don't press `CTRL x` to exit the text editor.  In the other window, keep your standard command line interface from which to compile, test, push, etc.  Develop in one window, run in the other.  You can actually get a pretty efficient workflow going by switching between windows in this manner.

When you're finished, exit `nano` in one window and then type `exit` to close that connection.  In the other window use
```
prompt$ sudo shutdown -h now
```
to shutdown your Pi as usual.

