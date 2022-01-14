[Back to Main Page](../index.html) 

# Rubber Ducky

<img src="../img/banner_hackerblue.jpg" width="1000">

## Rubber Ducky Dropoff

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

