# Lab 0: Linux and git Basics

### Goals

This lab will let you working in Linux and using Command Line Interfaces (CLIs). 
Along the way, you'll get some practice with several important Linux sysadmin concepts, including

  - copying the operating system onto an SD card
  - logging in to a remote computer using SSH
  - editing files in the terminal
  - superuser acccounts and `sudo`
  - installing packages with `apt-get`
  - the `man` command
  - connecting programs with pipes
  - talking cows


### Pre-lab prep
Let's get the boring lecture out of the way before lab, so you can spend this entire lab period 
playing around with the basics of the Linux CLI and git.  To begin, watch these videos (in order or they won't make sense).  All of them are on YouTube and range from 7-10 minutes each and total about 1 hour.

1. [Command Line Interfaces](https://www.youtube.com/watch?v=3WddgzyhHk8)
2. [Directory Hierarchies](https://www.youtube.com/watch?v=MdMCWKpWbjc)
3. [Basic Linux Commands](https://www.youtube.com/watch?v=wNB72BaH6rA)
4. [Paths and Special Directories](https://www.youtube.com/watch?v=RPviGYRAdgU)
5. [Relative and Absolute Paths](https://www.youtube.com/watch?v=sqX6hu7oEew)
6. [Command Flags](https://www.youtube.com/watch?v=fU6BDYVE2og)
7. [Creating and Deleting Directories](https://www.youtube.com/watch?v=AVzqquRi_-g)
8. [Copying Files](https://www.youtube.com/watch?v=MYe58LbbfVU)
9. [Moving/renaming Files and Directories]

You may also want to refer to [Notes on Linux and the Terminal Environment](https://github.com/vsummet/IntroductionToComputerSystems/blob/master/Notes/01b-Linux_and_the_Shell.md) which you already read during the first week of class.

### Beginning
0. Note that all the following steps assume you've already created your accounts and the connection as specified in the software setup discussed on the first day of class.  If you haven't done this, you need to follow the instructions on the software setup page before beginning.  See the link on the class calendar.
1. Log into codeanywhere.  You should see the Fall2018 connection you created on the left hand sidebar.  Right click on the connection and choose "SSH Terminal".  You should now see a tab with "Fall 2018" and a prompt like: ```cabox@box-codeanywhere:~/workspace$```.  This terminal window will allow us to issue (text) commands to the Linux machine provided to us by codeanywhere.
2.  Now work through the following commands, one at a time, at the prompt. After each command (first column), the observations (second column) specify things to look for or notice. Observing these things will help you cement your understanding of the directory hierarchy and command line environment. 

Command | Observations
--------|--------------
``` LS ``` | case sensitivity is extremely important in CLIs.  If you get a "command not found" errors, check your capitalization and spacing
```pwd``` | print working (current) directory.  Notice the output: ```/home/cabox/workspace```.  This is your current location in the hierarchy.  codeanywhere has automatically created a directory called ```workspace```.  Sub-directories that you create will show in the graphical
hierarchy in the left hand sidebar.
```ls``` | list the contents of the directory.  Notice the lack of output.  There are no files/directories in this directory.
```ls /``` | list the contents of the root directory.  Notice the output.
-----------------|------------------------------------------------
```mkdir cms230``` | create a subdirectory for this class
```cd cms230``` | change into your cms230 directory (or, make cms230 the current (working) directory).  Notice how the prompt changed too!
```ls``` | directory should be empty so nothing is printed to the display
```pwd``` | notice the output: ```/home/cabox/workspace/cms230```
-----------------|------------------------------------------------
```touch one.txt``` | this command creates an empty data file named ```one.txt``` in the current directory
```ls``` | Can you see the file you just created?
```touch two.txt``` | same but a file named ```two.txt``` 
```ls -l``` | Notice the different format. The ```-l``` flag makes the ```ls``` command give us more information (ie, the "long format").  Can you figure out which field gives information about the file size?
-----------------|------------------------------------------------
```cd``` | Can you guess the current directory after this command executes?
```pwd``` |Check your guess. Did you guess correctly?
-----------------|------------------------------------------------
```cd /home``` | How did the prompt change? What is the prompt showing you at all times?
```ls /home``` | contains all the home directories for the users of the machine. So what you see depends on machine and authorized users.
```touch myfile.txt``` | This should fail for you. What error did you get? You will not have the proper permissions to write to this directory.
-----------------|------------------------------------------------
```cd``` | Where are you in the directory heirarchy? Check if you need to.
```cd ..``` | Now where are you? Check if you need to. What did the ```..``` do?
```cd ~/cms230``` | What happened?  Why?
```cd ~/workspace/cms230``` | Where are you? What directory does the ```~``` represent?
```cd ../..``` | Where are you? What happened when you used ```..``` twice?



3. Now let's kick it up a notch.  We're going to add in lots of small details that really make a difference.  Pay close attention to each command (left column) and the observations (right column).  If you get confused, ask me or someone sitting near you.  It's important to not gloss over things you don't understand now as we'll be using these commands all semester.

--------|--------------
``` cd ~/workspace/cms230 ``` | move into your ```cms230``` directory
``` mkdir temp ``` | make a temporary directory for this exercise
``` touch temp/file1.txt ``` | make an empty file in the temporary directory
``` ls ``` | list the contents of your ```cms230``` directory
``` ls -l ``` | list, long format.  Notice the output and how it differs from the previous command.  What do you think it means if a line of output starts with a 'd'?
``` ls -L ``` | did you get the same result?
------------------------------------------------------|------------------------------------------------
```mkdir exercise2``` | create another subdirectory for this exercise
```ls``` | you should now see the ```temp``` and ```exercise2``` directories and files you created earlier during exercise 1
```touch temp/file3.txt``` | make another file in the temporary directory
```touch exercise2/file1.txt``` |
```touch exercise2/file2.txt``` | make some files in the ```exercise2``` directory
```ls -l exercise2``` | what happens when you list a directory?
------------------------------------------------------|------------------------------------------------
```mv exercise2/file1.txt exercise2/file3.txt``` |
```ls exercise2``` | Where did ```file1.txt``` go in the ```exercise2``` directory?
```mv exercise2/file2.txt``` | What happened? What does this error mean?
```mv exercise2/file2.txt .``` | What is the meaning of the ```.``` at the end of the command?
```ls```|
```ls exercise2 ``` | What happened to ```file2.txt```? Why?
```ls . ``` | What is . a shortcut for?
------------------------------------------------------|------------------------------------------------
```touch temp/file2.txt``` | make another file in the temporary directory.
```ls -l temp``` | What order are the files in?
```ls -lt temp``` | What order are the files in now? What does the -t flag do?
```ls -lrt temp``` | What order are the files in now? What does the -r flag do?
------------------------------------------------------|------------------------------------------------
```cp temp/file1.txt boo.txt```|
```ls``` | Where did you copy the file to? Why?
```cp temp/file1.txt exercise2``` |
```ls temp``` |
```ls exercise2``` | What happened? Why?
```cp temp/file1.txt exercise2/file4.txt```|
```ls -l exercise2``` | What file(s) are in the directory ```exercise2```?
------------------------------------------------------|------------------------------------------------
```rm temp/File1.txt``` | What does the error tell you?
```rm temp/file1.txt``` |
```rm temp/file2.txt``` |
```rm temp/file3.txt``` |
```ls temp```| What is in the temp directory now?
```rmdir temp``` |
```ls temp``` | directory should be gone
```rm file2.txt``` | remove the file from this (current) directory
```rm boo.txt``` | remove file from this (current) directory
------------------------------------------------------|------------------------------------------------
```rmdir exercise2``` | What happened? What can you infer about the rmdir command?
```mv exercise2 exer2``` |
```ls``` | What is the effect of ```mv``` when invoked on a directory?
```ls exer2``` |  Note the files still in ```exer2```
```rm exer2/*.txt``` | cleanup ```exer2``` directory
```ls exer2``` | What happened in the previous step?  What was the effect of using ```*.txt```?
```rmdir exer2``` | finish cleanup

### Check Yourself
At this point, you're starting to build up some knowledge about the basic Linux commands and navigate (via text only!) around the directory hierarchy.  As a quick recap, you should be able give a 1 sentence explanation about what each of the following commands do.  These commands are the absolute essentials in Linux.

- ```cd```
- ```touch```
- ```ls```
- ```mkdir```
- ```pwd```
- ```rm```
- ```rmdir```
- ```mv```
