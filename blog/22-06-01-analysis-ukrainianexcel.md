[Back to Main Page](../index.html) 

# An analysis of the excel targeting various ukrainian organizations

<img src="../img/blog-22-malwarebanner.PNG" width="1000">

It is important to know that our approach can be generalized to any other malicious documents, and used in the future. These documents were used to Ukrainian organizations in the context of the military conflict between Russia and Ukraine. Make sure to use a Virtual Machine for all further steps..

## Analysis of the .xlsx

The first file we will be analyzing is an Excel document that has been spreaded via various ways, including basic e-mails. There are various ways to get access to it, one would be checking for the SHA and downloading the sample [here](https://tria.ge/220208-vpr4vsbegl).

After downloading it, we can use [oleid](https://github.com/decalage2/oletools) to check if there Microsoft OLE2 files (also called Structured Storage, Compound File Binary Format or Compound Document File Format), such as Microsoft Office documents or Outlook messages, mainly for malware analysis, forensics and debugging. 

```
oleid malware.xlsm
```



