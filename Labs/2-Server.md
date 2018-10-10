# Server

A web server is a program that receives and processes HTTP requests. HTTP (the "Hypertext Transfer Protocol") is the main protocol used to request and transmit web pages over the global Internet.

In this lab, you'll turn your Raspberry Pi into a web server, create a basic web page, and experiment with a few of Linux's security features. This will give you practice:
  
  - Installing and configuring the nginx web server
  - Connecting to the server using its IP address and HTTP
  - Creating and styling a basic HTML web page
  - Shadow password files
  - Password cracking with John the Ripper

## Initial Setup

Repeat the basic setup process from the last lab:

  1. Connect your Pi to your laptop using an ethernet cable.
  
  2. Open the terminal application (for Mac users) or PuTTY (for Windows users).
  
  3. Connect the power supply to the Pi and wait about a minute
  
  4. Log in to `pi@raspberrypi.local`. For Mac users, the terminal command is
  
  ```
  prompt$ ssh pi@raspberrypi.local
  ```
  
  The default password is `raspberry`.
   
Once you're logged in to the Pi, verify that your wi-if is working:

```
prompt$ ping 8.8.8.8
```

Remember that you can use `CTRL + c` to terminate the `ping` program.

## Install nginx

nginx is one the two main web server programs widely used in the modern Internet. The other is Apache.

Install nginx using `apt-get`.

```
prompt$ sudo apt-get install nginx
```

Press `y` and then `Enter` if you are prompted to confirm the installation.

**Question**: what does the `sudo` command do?  Re-read the previous lab if you need an explanation.

nginx is automatically configured to start as soon as its host boots. This makes sense for a production web server, which should start up as quickly as possible after any downtime, but it isn't necessary for our Pi. As a general rule, never keep a server open on your machine when you aren't actively using it for something. To disable autorun:

```
prompt$ sudo update-rc.d -f nginx disable
```

Start nginx:

```
prompt$ sudo nginx
```
If this command gives you a "could not bind" error, that's okay. It just means that nginx started running when you installed it.

Your server is now up and sending a default page in response to HTTP requests.

## Test

Get your Pi's IP address:

```
prompt$ ifconfig
```

Look to the portion of the output labeled `wlan0`. It will look similar to this:

```
wlan0     Link encap:Ethernet  HWaddr b8:27:eb:7d:98:e8  
          inet addr:172.23.17.102  Bcast:172.23.31.255  Mask:255.255.240.0
          inet6 addr: fe80::3a58:e6c:c5e6:140c/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:401 errors:0 dropped:354 overruns:0 frame:0
          TX packets:88 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:86595 (84.5 KiB)  TX bytes:16455 (16.0 KiB)
```

Look at the `inet addr` field. This is your Pi's IP address, a unique identifier that browsers and other systems can use to reach your machine over the Internet. The IP address is a 32-bit value, but is conventionally written in "dot notation," showing the integer values of each of its four bytes.

Open up a web browser (Chrome, Firefox, Safari, etc) on your computer and type `http://172.23.17.102` in the browser's address bar&mdash;replace my example IP address with your Pi's own IP address printed by `ifconfig`. The default nginx web page will load in your browser.

The default page is located at `/var/www/html/index.nginx-debian.html`. Edit it using `nano`:

```
prompt$ sudo nano /var/www/html/index.nginx-debian.html
```

The HTML ("Hypertext Markup Language") file shows the text content of the web page, together with **tags** that describe how it should be structured and formatted. For example, the `<h1>` indicates a top-level header that appears large and bold when the page is displayed. Tags are usually grouped in matched pairs, with closing tags denoted by a `/`, e.g. `</h1>`.
  
Make a change to the file by adding

```
<h1>OHAI DERE</h1>
```

to the existing `<body>` section. Save with CTRL + o (and Enter to confirm) and exit with CTRL + x. Reload the page in the web browser and you should see your changes in the page's text.

## Configure and Serve a New File

Now we'll create a new web page in a new location and configure nginx to serve that page instead of the default one.

Change back to your home directory and make a new directory called `web`.

```
prompt$ cd ~
prompt$ mkdir web
prompt$ cd web
```

The name `web` is not significant; it's just easy to remember.

Let's make a basic web page. It needs to be named `index.html`.

```
prompt$ nano index.html
```

```
<!DOCTYPE html>
<html>
    <!-- This is an HTML comment -->
    <!-- 
      --  The basic HTML file has two sections:
      --    <head> contains metainformation on the whole page
      --    <body> contains page content
      -->

    <head>
        <title>This appears at the top of the browser.</title>
    </head>
    <body>
        <h1>This is a Website.</h1>
        
        <p>This is a paragraph of text on a website.
        
        <p>Here's another paragraph.
        
        <p>That's all.
    </body>
</html>
```

Save the file using CTRL + o (Enter to confirm) and exit the editor using CTRL + x.

Next, edit the nginx configuration to serve your new page. Get the full path of the `web` directory.

```
prompt$ pwd
/home/pi/web
```

The default nginx config file is located at `/etc/nginx/sites-available/default`.

```
prompt$ sudo nano /etc/nginx/sites-available/default
```

Look for a block close to the top of the file that starts with `server`:

```
# Default server configuration
#
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;

        root /var/www/html;
```

Lines beginning with `#` are comments. The SSL block, used for secure authentication, is commented out.

The `root` line specifies the location of the directory holding the index page. Change it to

```
    root /home/pi/web;
```

Save the file, then reload nginx to make it use the new configuration.

```
prompt$ sudo nginx -s reload
```

Refresh the page in the web browser one more time to see your new page.

## Interior Decoration

cURL is a useful tool that let's you interact with a remote server from the command line. Among its MANY uses is grabbing files from a remote location. From your `/home/pi/web` directory, run

```
prompt$ curl -O https://upload.wikimedia.org/wikipedia/commons/0/0a/The_Great_Wave_off_Kanagawa.jpg
```

The `-O` flag saves the file locally.  Note that this is a capital o, not a zero.

Open your `index.html` file and add an image tag to its body.

```
<p>
<img src="The_Great_Wave_off_Kanagawa.jpg"/>
```

Reload the page and check out your picture.

Yikes. That's large. You can add style to the tag to scale the image to a percentage of the display width:

```
<p>
<img src="The_Great_Wave_off_Kanagawa.jpg" style="width:100%"/>
```

You can also set an absolute size in pixels, e.g. `"width:200px"`.

This method of styling elements was common in the old-school web, but modern practice favors separating page content from styling.

Move the `<style>` information up to the `<head>` block of your page:

```
<head>
    <title>This appears at the top of the browser.</title>

    <style>
        img {
            width: 100%;
        }
    </style>
</head>
```

The style rule specifies that the contents of all `<img>` tags should have their width set to 50% of the page size.

You can add elements to the style block to control the presentation of other parts of the page. For example, to style the contents of  the entire body:

```
<head>
    <title>This appears at the top of the browser.</title>

    <style>
        img {
            width: 200px;
        }

        body {
            font-family: "Helvetica", "Arial", sans-serif;
            font-size: 18pt;
            background-color: #FAFAFA;
        }
    </style>
</head>
```

The `font-family` parameter takes a list of fonts and uses the first one that's available on the system, with the last choice being the default system sans-serif font.

Colors are specified as three hex values, denoting the R, G, and B components of the color. Play around with some color parameters and see what you can generate.

You can also set `color` to control the color of the text.

Here's one last style block that makes things a little more readable by bringing the page contents to the center.

```
<style>
    img {
        width: 100%;
    }

    body {
        font-family: "Helvetica", "Arial", sans-serif;
        font-size: 18pt;
        color: #333333;
        background-color: #FCFCFC;
        margin: 40px auto;
        max-width: 640px;
    }
</style>
```

`max-width` controls the size of the display region in the browser.

`margin` sets a padding of 40px around all sides of the page content; `auto` centers the display region inside the browser frame, pulling everything to the middle. Note that this is centering the display region, not the content itself.

The background and text are softened a little away from strict white and black.

## Password Cracking

For the second part of the lab, we'll work on cracking some passwords using *John the Ripper*, an open source cracking program.

### How does a Linux system store your password?

Old systems actually stored users' passwords in a cleartext file called `/etc/passwd`. Unfortunately, this meant that anyone gaining access to that file would have every system users' real password.

Modern systems use a **shadow** password file, stored in `/etc/shadow`. The shadow file doesn't store user's real passwords. Instead, it stores **hashes** of each password.

Recall: **what's a hash function**?

When a user logs in, the system can calculate the hash of the user password and compares it to the stored hash in the shadow file. If the hashes match, then the system assumes that the submitted password is the real password and accept the user's credentials.

Note that for this to be secure, it must be **impossible** to recover the real password given only the hash in the shadow file. Strong cryptographic hash functions have this property. In practice, however, weak passwords may still be easy to crack.

### Who Knows What Secrets Lurk in the Hearts of Men? The Shadow Knows

Try to dump the contents of `/etc/shadow/` to the screen using the `cat` command.

```
prompt$ cat /etc/shadow
```

You'll get `Permission denied` as a result.

As you might expect, the `shadow` file contains sensitive information, so it's not accessible to regular users. You have to use root-level priviliges to interact with it.

```
prompt$ sudo cat /etc/shadow
```

Look for a line that begins with `pi:$6$X8NL...`. This is shadow password file entry for user `pi`, the default user that we've used to log in to the Raspberry Pi.

The huge string specifies the hash of Pi's password. In this case, we know the password is `raspberry`, but it would be hard to determine that if all we had was the `shadow` file.

The leading `$6$` at the beginning of the hash string is called the **salt**. The number in the salt identifies the function used to generate the hashed password entry. The value 6 indicates the SHA-512 hash function.


### Get Ripped

A password cracker takes a shadow password file as input and reverse-engineers the real passwords that correspond to the hashes it contains.

Install John the Ripper:

```
prompt$ sudo apt-get install john
```

Now make an example shadow password file:

```
prompt$ cd ~
prompt$ openssl passwd -1 "password" > shadow_test
```

`openssl` is a program for performing, among other things, crypto-related operations. The `-1` (one, not L) flag indicates the MD5 hash function. `password` is the real password that is being hashed.

The operation `> shadow_test` redirects the output of `openssl`, which would normally be printed to the terminal, to a file named `shadow_test`. You can check the contents of the file:

```
prompt$ cat shadow_test
```

Now get cracking:

```
prompt$ john shadow_test
```

John quickly tries a few basic cracking rules, one of which is blasting through a list of common passwords. Since `password` is on that list, it quickly identifies that the string `password` matches up with the data in the `shadow_test` file.

Let's try something a littler harder:

```
prompt$ openssl passwd -1 "raspberry" > shadow_test
```

You can run `john shadow_test` again and let it run for a minute or so, but it won't crack the password. `raspberry` is too unusual for the default cracking approach. 

Try pressing the spacebar as `john` runs to get a status update on what it's currently trying.

Can we do better? Yes, we can, with a **dictionary** attack. Press CTRL-C to end the currently running program.  Then download a large list of words:

```
prompt$ sudo apt-get install wamerican-large
```

Now run the program again, using the large wordlist as a list of candidate passwords.

```
prompt$ john --wordlist=/usr/share/dict/american-english-large shadow_test
```

It will take a minute, but `john` will eventually work its way through the list to find `raspberry` and crack the password. Press the spacebar for updates as it cracks; you will see it move alphabetically through the dictionary.

How about one more?

```
prompt$ openssl passwd -1 "raspberry1" > shadow_test
```

John is still one step ahead of you, because it can apply **mangling rules** to the candidate wordlist to generate new passwords that match common patterns.  Mangling rules cover common human tricks such as adding a number to the end of a common word or replacing o's with the number 0 in a common word.

```
prompt$ john --wordlist=/usr/share/dict/american-english-large --rules shadow_test
```

This approach is extremely effective. People are not very creative, on average, so a good list of words and a set of common mangling patterns is enough to crack a large fraction of the hashes in a typical password database.

How to create good passwords? Basically, a combination of length and unusualness.

- Long, truly random passwords are always strong, because they can only be cracked by brute force. On the other hand, they're almost impossible to remember without a password manager
    
- Failing that, a combination of length, a large character set with capitals and special characters, and a generation process that avoids common mangling rules.

**Think about:** What rules different companies (or your school) have about password requirements?

One popular approach is to randomly string together words from a list.

![xkcd 936](https://imgs.xkcd.com/comics/password_strength.png)

## Last Step

Shutdown your Pi by typing
```
prompt$ sudo shutdown -h now
```

This causes your Pi to immediately begin shutting down (closing your Putty or SSH connection).  Once the green light stops blinking, unplug the power cord, and put your Pi away.


