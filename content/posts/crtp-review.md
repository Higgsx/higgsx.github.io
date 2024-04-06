+++
title = 'Crtp Review'
date = 2024-04-06T10:53:10-04:00
draft = false
+++

Introduction

Before starting HackTheBox Offshore I needed some background on attacking Active Directory environment. I had had some sysadmin experience in AD and it was time to going through penetration testing course focused on Active Directory. I had already heard about PentesterAcademy training course (Attacking and Defending Active Directory) and didn’t spend time on searching much back then and immediately purchased this course which is accompanying training course for CRTP examination.

activedirectorylab
TL;DR

Yes! it’s great training course, get it.
Course Content

When you purchase the course, you are given following:

    Presentation slides in a PDF format, about 350 slides
    37 Video recordings including lab walkthroughs. Each about 25-30 minutes
    Lab manual with detailed walkthrough in PDF format
    (Unofficial) Discord channel dedicated to students of CRTP
    Lab with multiple forests and multiple domains

Attacking and Defending Active Directory course is about as its name suggests attack vectors against AD and what mitigation steps exists to defend it. For instance, if course instructor (Nikhil) covers let’s say kerberoasting, he always includes prevention and mitigation steps with very detailed explanation.

It covers

    Local Privilege Enumeration (but in very small amount, which is understandable. LPE is whole another topic)
    Domain Privilege enumeration
    Abuse domain and forest trusts
    AD Domain enumeration with 2 tools: PowerView and MS AD Module
    Heavy mimikatz, kekeo and Rubeus usage
    DCSync, DCShadow, Forging Golden, Silver and Inter-Realm TGTs for abusing cross domain trusts
    Persistence approaches: Scheduled Tasks, AdminSDHolders, abusing ACL of domain objects and etc.
    Contrained and Unconstrained delegation abuse
    Ways to bypass Constrained Language Mode (CLM)
    etc

As I’ve already said when attack approach is introduced, there is always defense methodologies and Nikhil does explain that very well. So, it would be awesome course not only for red teamers but also for blue teams. Not only defense but also Nikhil discusses few detection ways such as: Honey(decoy) service accounts, Microsoft ATA (Advanced Threat Analytics) and etc.

This course is for beginners with very little knowledge in AD but I believe this would be good experience also for experienced red team operators.

If you ask me, attacks are about 70% and rest 30% is dedicated to defense.
LAB

When you purchase the course, lab is included and it contains several domains and several forests where you have to pivot through. Even though this is a shared lab you will not get distracted and you’ll feel like are’re alone and that’s awesome :)

You start in a child domain of a forest, then you have to move up to forest root domain via forging IR-TGTs or abusing krbtgt NTLM hash and forging TGT which will give us Enterprise Admin privilege escalation.

Also you have multiple MSSQL servers linked together and you have to abuse it. MSSQL servers are so common in todays enterprise environments and having background in attacking MSSQL servers is useful when you have actual pentest.

You have options of lab packages: 30/60/90 days. If you have some background and just want as a refresher, opt for 30 days. If you are completely new then 60 days will be best choice. I don’t think 90 day package will be beneficial to anyone. This course isn’t too complex and/or too long and is doable within 60 days.

You will not have CVEs at all. Everything you abuse will be feautures and misconfigurations of windows. Don’t download CVE exploits! OSCP have no power here.

oscp
Exam

Exam is not hard as long as you know everything taught in the course. It will not include anything that isn’t in the course. If you want success, make sure you master everything you already learned. Don’t just remember tool commands like arguments, make sure you have reasonable explanations as to why this tool worked and why not. Master more than one tool, e.g if you mastered PowerView, make sure you master ADModule as well. It is good practice not only for this exam but in general. Sometimes tools just don’t work and you have to find a way.

Note: Don’t think that you will be ok with just PowerView. That’s true that difference between ADModule and PowerView is small, but ADModule is genuine Microsoft module and you should master it. It will be especially useful in scenarios when your objective is living off the land and staying low on the radars of blue teams. Using legitimate module will give you more additional powers of invisibility.

Also make sure you understand principle of abusing domain trust. visualize paths that domain uses when you visit service located in different domain or forest. You have little complex structure when you are dealing with IR-TGTs or Inter-Realm TGTs. So make sure you understand this well.

In addition to that, I also built my own AD environment in the cloud via AWS. It helped me visualize where everything is. For example When you change Service Principal Name of an account via Set-DomainObject cmdlet, what is changed exactly on a GUI side? or when you change UserAccountControl(NOT UAC) flags what is changed exactly? that’s questions you will find answers when you have your own AD lab. You can choose AWS or whatever you like.

Don’t rush, and make sure you enumerate properly. Hacking is all about thinking and enumeration.

Exam support team will be ready even if you took exam on weekends on non-working hours
PentesterAcademy support

I would like to dedicate separate section on support of PentesterAcademy. They are very fast in replying and are good at resolving your issue. When you send email you will get answer within 1-2 hours.
Should I purchase this course?

Yes, definitely. I highly recommend this course to everyone working in Cyber Security. Doesn’t matter whether you are on red team or blue team.

If you have solid background in red teaming AD, then maybe opt for more advanced course: CRTE. If it isn’t enough for you then opt for GCB/PACES, which is the most advanced course and certification on pentester academy and is suitable for experienced red teamers. Don’t rush and don’t just dive into directly without solid background.
