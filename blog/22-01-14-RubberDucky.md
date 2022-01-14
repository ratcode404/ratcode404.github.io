[Back to Main Page](../index.html) 

# Rubber Ducky

<img src="../img/banner_hackerblue.jpg" width="1000">

## Rubber Ducky Setup

Recently we decided to test our employees by dropping off a modified Rubber Ducky outside of the building and creating awareness for such social attacks.


The DuckyCode opens up a webpage.php, which tracks the device and logs it on logfile which has been placed on the webserver.

This is the uncompiled payload.

```
DELAY 1000
GUI r
DELAY 100
STRING https://company.tld/rubduck/link.php // insert link here
ENTER
```
  
After creating the payload we inject it on the USB, by placing the inject.bin in the root directory.

## Rubber Ducky Dropoff

## Rubber Ducky Result

```
2021-11-18 20:04:58: New RubberDucky initialisation detected from 192.168.1.105, 192.168.1.105
2021-11-18 20:05:45: New RubberDucky initialisation detected from 192.168.1.105, 192.168.1.105
2021-11-18 20:06:12: New RubberDucky initialisation detected from 192.168.1.105, 192.168.1.105
2021-11-18 20:06:31: New RubberDucky initialisation detected from 192.168.1.105, 192.168.1.105
2021-11-18 20:08:38: New RubberDucky initialisation detected from 192.168.1.105, 192.168.1.105
```

