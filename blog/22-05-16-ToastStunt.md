[Back to Main Page](../index.html) 

# Setting up a MOO

<img src="../img/banner_hackerblue.jpg" width="1000">

ToastStunt is a fork of the LambdaMOO / Stunt server.

## Basic Installation of ToastStunt

First, let's get the machine up to date.

```
sudo apt upgrade 
sudo apt update
```

<img src="../img/blog-22-moo-apdupdate.png" width="750">

Then install all the libs needed to run ToastStunt

```
sudo apt install build-essential bison gperf cmake libsqlite3-dev libaspell-dev libpcre3-dev nettle-dev g++ libcurl4-openssl-dev libargon2-dev libssl-dev
```

Then we clone the repos and copy the toastcore.db into the toaststunt directory.

```
git clone https://github.com/lisdude/toaststunt
git clone https://github.com/lisdude/toastcore
cp toastcore/toastcore.db toaststunt/toastcore.db
```

<img src="../img/blog-22-moo-gitclone.png" width="750">

Making a build directory inside /toaststunt to make the game.

```
cd toaststunt
sudo mkdir build && cd build
sudo cmake ../
sudo make -j2
```

<img src="../img/blog-22-moo-makej2.png" width="750">

Starting the basic moo and run it as a background job.

```
./moo ../toastcore.db <database-name>.db
< Control Z >
jobs
bg 1
```

## Run dome-client.js

dome-client.js is a webclient to use instead of a basic telnet connection. First, we're cloning the repo as well.

```
cd ..
cd ..
git clone https://github.com/javachilly/dome-client.js/
```

Install NPM, which will be needed for the webinterface.

```
sudo apt install NPM
sudo apt update
```
Next let's fetch the required update modules by using npm.

```
cd dome-client.js
npm install
```

<img src="../img/blog-22-moo-npm.png" width="750">

```
sudo npm install -g forever
```

Next we will edit the config/default.js to make sure the basic information is correct.

```
nano config/default.js
< CTRL X >
```

Now we're starting the server on port 5555 in debugging mode.

```
debug.sh
```

<img src="../img/blog-22-moo-debugsh.png" width="750">

## Connect to your Moo

Let's up telnet connection to our fresh moo.

```
telnet localhost 7777
```

Due proxy settings on local machine we will not see your logon page, connect to the wizard.

```
connect wizard
```

<img src="../img/blog-22-moo-moorunning.png" width="750">


## MOO Commands

I will add some commands I have struggled with as I find them.

### Basic Commands

Set a password for yourself.
```
@password <new-password>
```

Creating a new account for the MOO.
```
@make-player <name> <e-mail>
```

Use kids, parents and @show to find the correct objects.
```
@parents #3
@kids #2
@kids #3
@show #62
```

Rename and describe yourself or any other object.
```
@rename #2 to <your-name>
@describe #2 as <your-description>
```

### Building Commands

Create a new room and set up the correct messages.
```
@dig north,n to "Roomname"
@leave north is "You make your way to the north."
@oleave north is "goes north."
@oarrive north is "walks in from the south."
```

Move to the new room and configure it as well.
```
@dig south,s to #<previous-room>
@rename #<new-room> to [red]Red Colored Roomname[normal]
@describe #<new-room> as [grey]Grey Description.[normal]
@leave south is "You make your way to the south."
@oleave south is "goes south."
@oarrive south is "walks in from the north."
```

Things may vary with front or back, such as:
```
(@oleave front is "goes to the front.")
```

### Programmer Commands

Emable programmer window on webclient.
```
@edit-options local+
```

Editing a Verb, like showing exit names
```
@edit $room:tell_contents

Then replace
if ((!this.dark) && (contents != {}))
with
if ((!this.dark) && ((contents != {}) || (this.exits != {})))

And add the following as an additional if in ctype=3, after endif that is linked to players on the very bottom.
if (this.exits)
      player:tell($string_utils:english_list($list_utils:map_prop(this:obvious_exits(), "name")));
endif
```

Slowing down movement speed.

```
@edit $room:e east w west s south n north ne northeast nw northwest se southeast sw southwest u up d down
@edit $exit:move

Add interrupt wherever you feel it is right.
```

### Configuration Commands

Enable basic ANSI options.
```
@ansi-option +colors
@ansi-option +blinking
@ansi-option +bold
@ansi-option +truecolor
@ansi-option +misc
```
