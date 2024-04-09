+++
title = 'DPAPI Decryption'
date = 2022-02-01T11:15:32-04:00
draft = false
+++

## Introduction

DPAPI stands for Data Protection API and was introducted by microsoft in Windows 2000 and since then it is used heavily by windows even if you aren’t aware of it. Simply put, DPAPI lets you encrypt/decrypt data. You don’t need to worry about encryption keys at all.

DPAPI has 2 simple API:
```
CryptProtectData()
CryptUnprotectData()
```

![Untitled](/images/oldredsec/dpapi1.png)

As you might already guessed it:

For encryption you use `CryptProtectData()`

For decryption you use `CryptUnprotectData()`

Windows operating system takes care of everything. for encryption it uses your password hash when you are in local user context. Different story is when you are part of a Active Directory domain. This post will be about local user context.
Credential Manager

Credential Manager is windows feature which contains all your passwords safe. You can keep RDP, web, Chrome, github passwords and etc. It is like password manager.

![Untitled](/images/oldredsec/credmanager.png)

You can see on screenshot that I have 2 RDP password saved (TERMSRV service means I have RDP credentials saved). Also I have github password. It is useful when you push new commits and don’t have to type credentials everytime. git commandline utility decrypts this password behind the scenes and uses it to authenticate remotely.

Web Credentials is for passwords related to web. You can also manually add new creds via such feature.

All of this is done via DPAPI. It uses something like Master Key which encrypts and decrypts your data. Master Keys are stored in user’s AppData directory. For testing purposes create test credentials:

![Untitled](/images/oldredsec/testcred.png)

My master key which was used to encrypt my data. Example image

b625df90-d6ad-4528-a3c1-0c3beb58df34 is our key file and we will see proof of that when we open it via mimikatz

interested what happens when we saved test credentials via Credential Manager? We can get that via procmon from sysinternals tools:

![Untitled](/images/oldredsec/procmon.png)

process in charge here was lsass.exe which took care of this. It opened Prefered file which points to recent masterkey in use. Takes that and uses it to encrypt our data and encrypted blob is put into A5FEB27BE0210EF5E455C689AAC6802B data blob.

Example image
Time to mimikatz

Let’s open this in mimikatz and see what is in it:

![Untitled](/images/oldredsec/mimikatz.png)

Mimikatz module for this is dpapi::cred which takes /in:<path> argument as a encrypted data blob. We can see from above picture that guidMasterKey or simply Master key used is b625df90-d6ad-4528-a3c1-0c3beb58df34.

![Untitled](/images/oldredsec/pbdata.png)

pbData is our actual encrypted credential

Now, use our master key and decrypt it. Because we are in a user context there is no need to decrypt master key and then decrypt encrypted data blob

Command is following:

mimikatz # dpapi::cred /in:"C:\Users\Higgsx\AppData\Roaming\Microsoft\Credentials\A5FEB27BE0210EF5E455C689AAC6802B" /unprotect

![Untitled](/images/oldredsec/unprotect.png)

Password is visible below: Example image

So as I said it was current user context. What happens when we log in via other account?

transfer master key and encrypted data blob to C:\Users\Public so that I can access them from another account:

![Untitled](/images/oldredsec/other.png)

we see that this data blob is expecting master key b625df90-d6ad-4528-a3c1-0c3beb58df34 as we expected.

Try to decrypt:

![Untitled](/images/oldredsec/other2.png)

but we failed:

![Untitled](/images/oldredsec/fail.png)

we failed because we need SID and password of the user who encrypted that.

SID: S-1-5-21-1160870239-168060136-1582979013-1000

Password: qweqwe

First decrypt master key:

![Untitled](/images/oldredsec/command.png)

below we see key value and sha1 value. any of them is applicable But in this case lets use key value (long one)

![Untitled](/images/oldredsec/mk.png)

Use that decrypted masterkey to unprotect data blog

![Untitled](/images/oldredsec/blah.png)

Finally:

![Untitled](/images/oldredsec/blah2.png)
Conclusion

This was part 1. In the next post I will try to explain the way Chrome web browser uses DPAPI to store its Cookies and ways to decrypt them. Sorry for small images here.

Stay tuned.
