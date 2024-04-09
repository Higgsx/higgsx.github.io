+++
title = 'HTB Prolab: Offshore'
date = 2022-04-06T10:39:34-04:00
draft = false
+++
Introduction

![Untitled](/images/oldredsec/brucelee.png)

At the beginning, HackTheBox was platform known for just a single box exercises but it evolved a lot and become one of the best platform for honing your cyber security skills and tradecraft.

One of the best feature of this platform is its prolabs

- Dante
- Offshore
- RastaLabs
- Cybernetics
- APTLabs

Prolabs are huge labs composed of several domains and forests, spanned across different subnets and each one of them varies by difficulty and sizes.

What offshore offers

HackTheBox offshore is one of the prolab which is focused mainly on Active Directory exploitation and lateral movement and is rated as intermediate level difficulty and is good practice opportunity for those who has basic background in networking, active directory, coding or for those who simply completed OSCP certification exam and are eager to learn more.

This lab covers following areas:

- Lateral Movement between Domains/Forests
- Active Directory attacks
- Reverse Engineering
- Local Privilege Escalation
- A few CTF exercises
- Web Application Attacks

Prerequisite

This lab definitely requires some prerequisites such as:

- Basic knowledge and hands-on experience of Active Directory
- Basic working experience with Linux
- Basic reading/writing coding skills
- Web Application security
- Basics of Networking

Active Directory

This is maybe the most important thing you have to know to succeed in this lab, because it is about 70% Active Directory and without this prerequisite you will have utterly difficult times. Of course there are some people who accomplished without prior knowledge of AD, but it must be very hard and stressful and really doesn’t worth it if you devote 24/7 to only this lab :)

So what to do beforehand? there are several options

- purchase online course on AD administration from let’s say Udemy, Pluralsight, CBT nuggets etc.
- Build your own AD lab on your laptop, PC, on your dedicated servers or use cloud providers such as Azure and AWS. I personally recommend AWS.

Some training providers have capability to host labs on their platforms and you can practice there. For example CBT Nuggets have feature where when you are going through the course you can practice in lab provided. You are given windows machines and you can interact with them just through your browser, no additional hardware/software are necessary except simple web browser like Google Chrome, Edge etc.

If you ask me you should do both

- Build your own AD lab, create domains, forests and trusts between them, install MSSQL servers and etc…
- And go through training providers' own lab and training courses

You don’t need to be AD guru to do offshore, just basics will do fine and rest can be learned along the practice in offshore. Well, still it isn’t enough :)

knowledge of some AD is good but attacking is different story. AD has lots of attack vectors and some of them are still effective and you should know them.

what can you do to improve your skillset is attacking AD?

The best training course out there on attacking AD is this linked below

https://www.pentesteracademy.com/activedirectorylab

PentesterAcademy’s CRTP training course is BEST and very well organized. I will devote one review on CRTP course, but in short, it is worth to purchase and pass the exam before tackling offshore, you will thank me later :)

In short you will learn:

- DCSync attack and why it works the way it works
- DCShadow for persistence
- Abusing ACLs
- Abuse local service misconfigurations
- Neat persistent tricks such as: AdminSDHolder, SeEnableDelegationPrivilege and so on.

Coding skills

Well, this is not as required as AD but will be very beneficial to you, if you want to automate some tasks and read codes. There were several cases where programming experience helped me much. In short, you should have basic coding skills, especially in python and that’s it.
Web Application Attacks

Offshore has lots of web applications where you exploit and gain code execution. You should know basics of web app hacking such as: XSS, SQLi, XXE, SSRF and so on. Nothing fancy just basics.

Where can you learn basics? https://portswigger.net/web-security
Networking

During your lab time you definitely have to pivot through different domains. Even domains aren’t accessible directly and you have to chain multiple domains to get to the point you want. You will touch topics such as SSH port forwardings(Remote, Local, Dynamic), Chisel, socks proxies and etc…

You should also have basic knowledge of networking such as IP addresses, subnets, default gateways, MACs, TCP/UDP, ICMP and etc..

You don’t need to be CCNA certified but should have some basics. Even half of Network+ is enough
How to succeed?

    It is time consuming. No easy wins. Almost everything is patched. make sure you have time, 2-3 hours per day
    HackTheBox discord server. You should use this. You will get stucked very often and you will need nudges along the ways and also will learn something by teaching others(If you’re active on discord, people will contact you for nudges and that’s awesome). Everyone is different and everyone has their own unique way of solving puzzles.
    Patience, Patience and patience. You will get stucked many times. Be aware of that.

My background before offshore
1st Month prior to purchasing offshore

I didn’t know anything about AD. Bought some training courses on Active Directory and Windows server administrations. Also I built my own lab on AWS and practiced there.

If you are interested which training courses I took

    Windows Server 2019 AD and GPOs
    AD Group Policies
    Server Academy

2nd month prior to purchasing offshore

Bought CRTP and passed exam at the end of the month.
3rd and 4th month

Working on offshore. It took me 2 month. After 1 month I had compromised all domains, I mean, I had Domain Admins on all of the domains but I still had several flags left and 1-2 weeks took me to find those rest flags. On final weeks I was trying to use C2 framework: Covenant but I had severe issues with its stability and gave up.

Finally I collected all flags and obtained certification

![Untitled](/images/oldredsec/cert.png)