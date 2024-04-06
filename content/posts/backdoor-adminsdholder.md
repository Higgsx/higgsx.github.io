+++
title = 'Backdoor Adminsdholder'
date = 2024-04-06T10:44:04-04:00
draft = false
+++


In a red team operation when we gain Domain Admin privileges, we want to make sure we will have access to the environment even though system administrators or blue teamers notice it. In Active Directory environment amount of TTPs (Techniques, Tactics, Procedures) for persistence is huge. One common way to persist is to use AdminSDHolder container, which is in System container

![Untitled](/images/oldredsec/Untitled.png)

Every 60 minutes SDPropagator in AD, will overwrite Protected groups ACL with AdminSDHolder ACL. In other words, if you change DACL of Domain Admins Group, it will be overwritten by SDProp every 1 hour.

Many folks will change DACL of Domain Admins but there is high chance that someone will notice it. In case we change DACL of AdminSDHolder, even though administrator notices and changes DACL of DA, in 60 minutes Domain Admins DACL will be changed and will give you more time to move laterally.

Note: For the following demonstrations I am using ZeroPointSecurity CRTO Lab.

First, Let’s see what we can do if we have FullControl (same as GenericAll) over DA group.

Create new user: panda

![Untitled](/images/oldredsec/Untitled%201.png)

Test if we really have that user via ActiveDirectory module, installed default on every Domain Controller

> Get-ADUser Panda

![Untitled](/images/oldredsec/Untitled%202.png)

and we want to give panda something powerful. Open Domain Admins group DACL or as everyone is familiar with: Permissions. We have DACL and SACL. DACL stands for: Discretionary Access Control List and SACL stands for: System Access Control List. SACL is for audit purposes but the blog post isn’t about that.

![Untitled](/images/oldredsec/Untitled%203.png)

ACL is a list or chain of ACEs. ACEs at the upper level takes precedence over ACEs below.

Click add to add our user panda to ACL

![Untitled](/images/oldredsec/Untitled%204.png)

And give full permission (Full Control) also known as GenericAll

![Untitled](/images/oldredsec/Untitled%205.png)

and we see listed here

![Untitled](/images/oldredsec/Untitled%206.png)

Now that panda has Full Access to this group we can add users to this group or change properties of DA group.

![Untitled](/images/oldredsec/Untitled%207.png)

first, in order to make sure panda has full access to the group we can use powershell.

> Get-ACL -Path "AD:CN=Domain Admins,CN=Users,DC=dev,DC=cyberbotic,DC=io" | select -ExpandProperty Access

![Untitled](/images/oldredsec/Untitled%208.png)

![Untitled](/images/oldredsec/Untitled%209.png)

now, execute powershell in a context of domain user: panda and try to add user to DA group

![Untitled](/images/oldredsec/Untitled%2010.png)

![Untitled](/images/oldredsec/Untitled%2011.png)

![Untitled](/images/oldredsec/Untitled%2012.png)

![Untitled](/images/oldredsec/Untitled%2013.png)

After Deletion

![Untitled](/images/oldredsec/Untitled%2014.png)

Important Note: just because you have GenericAll on Domain Admins group, doesn’t mean you can manipulate users enrolled in the group. So you can’t reset passwords of users within this group and etc.

You can add your user temporarily and grant yourself DA privileges. But issue is that it isn’t permanent because as we said: AdminSDHolder. Every 60 minutes, SDPropagator propagates AdminSDHolder’s ACL into Protected Groups: Domain Admin Print Operators etc.

![Untitled](/images/oldredsec/Untitled.png)

if you compare lets say DA group to adminsdholder group acls. they will be equal

![Untitled](/images/oldredsec/Untitled%2015.png)

So in order to make smart move, you can add your new user to AdminSDHolder DACL and in 60 minutes you will be added to every protected groups such as Domain Admins, Print Operators, Backup Operators and etc.

Many sysadmin aren’t aware of that feature, but are aware of membership of Domain Admins group. There is less chance of detection if the above mentioned container will be used to persistence.
