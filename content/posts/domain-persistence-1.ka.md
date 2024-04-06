+++
title = 'Domain Persistence 1'
date = 2024-04-06T09:34:15-04:00
draft = false
+++
![Untitled](/images/persistence-one/ad-logo.png)
მოგესალმებით

```
krbtgt იუზერის ჰეში არის ძალიან,ძალიან მნიშვნელოვანი. თვალის ჩინივით გაუფრთხილდით :))
```
Active Directory გარემოში არსებობს უამრავი გზები თუ როგორ შეინარჩუნოს შემტევმა პირმა წვდომა მას შემდეგ რაც მოიპოვებს წვდომას DA (Domain Admin) ან EA (Enterprise Admin) - ზე. თუკი მოხდება შემტევი პირის მიერ დომეინის “გაწევა” და თუკი კარგად გათვიცნობიერებულია, ძალიან რთულია ე.წ backdoor - ების ამოღება/განეიტრალება მათი სავარაუდო რაოდენობიდან გამომდინარე.

Domain Persistence იქნება პოსტების ციკლი, რომელიც მიეძღვნება ამ თემას. ეს პირველი ნაწილია და ამ ნაწილში შემოგთავაზებთ შემდეგ Persistence მეთოდებს:

- Forging Golden Tickets
    - PAC ვალიდაცია და Microsoft - ის პაჩი CVE-2021-42287
- Skeleton Keys
- Abuse AdminSDHolder
- Abuse Domain Object DACL

TGT და TGS

სანამ დავიწყებდეთ, პირველ რიგში გავიხსენოთ საფუძვლები რაც ვახსენეთ წინა პოსტებში. ძალიან რომ არ გაგვიგრძელდეს პოსტი, უბრალოდ დავლინკავ წინა პოსტს სადაც ახსნილია საფუძვლები როგორ ხდება TGT და TGS ების მოთხოვნა AD დომეინში.

https://redefense.xyz/post/attacking-forest/

ნებისმიერი TGT თიქეთი არის დაშიფრული krbtgt დომეინ იუზერის პაროლის ჰეშით. ამ NTLM ჰეშის დაკრეკვაზე დროს ნუ დაკარგავთ. ძალიან გრძელი random ად დაგენერირებული პაროლი აქვს ყოველთვის krbtgt - ს. უფრო მეტიც, თქვენს გემოზე პაროლი რომ შეუცვალოთ, სისტემა მას მიანიჭებს მაინც სხვა პაროლს.

TGT ში ჩაწერილია თუ ვინ ვართ ჩვენ, იუზერის სახელი, ჯგუფის სახელი, დომეინის SID, იუზერის SID და ა.შ. დომეინ კონტროლერი (კონკრეტულად KDC სერვისი) “ენდობა” TGT ში ჩაწერილ ინფორმაციას თუკი მის დეშიფრაციას შეძლებს krbtgt იუზერის ჰეშით.

შესაბამისად თუკი გვაქვს krbtgt იუზერის ჰეში, მაშინ ჩვენ შეგვიძლია არ შევაწუხოთ KDC და ჩვენით შევქმნათ TGT თიქეთები.

Golden Ticket

Golden Ticket არის იგივე რაც TGT თიქეთი, რომელიც იქნება ჩვენი შექმნილი. როგორც ვახსენეთ ჩვენ ეს შეგვიძლია, რადგანაც გვაქვს krbtgt იუზერის ჰეში. როგორც წესი მასში წერენ დომეინ ადმინის იუზერს, რომელიც იძლევა საშუალებას თავი “მეფედ” ვიგრძნოთ.

პირველ რიგში, გვჭირდება krbtgt იუზერის ჰეში. როგორ შეგვიძლია მისი შოვნა? თუკი დომეინზე გვაქვს დომეინ ადმინის უფლებები. ჩვენ ვხედავთ, რომ იუზერ lasha - ს, აქვს დომეინ ადმინის უფლებები.

![Untitled](/images/persistence-one/Untitled.png)

შესაბამისად შეგვიძლია DCSync ოპერაციის განხორციელება mimikatz ის დახმარებით. ვინაიდან და რადგანაც ამ იუზერს აქვს dcsync უფლებები, როგორც ეს ჩანს Domain Object - ის DACL - ში:

![Untitled](/images/persistence-one/Untitled%201.png)

ეს 3 პერმიშენი განსაზღვრავს DCSync ის უფლებებს და როგორც სურათზე ჩანს Domain Admin - ს აქვს ამის უფლებები


- Replication Directory Changes
- Replication Directory Changes All
- Replication Directory Changes in Filtered Set

![Untitled](/images/persistence-one/Untitled%202.png)

როგორც სურათზე ჩანს გვაქვს krbtgt იუზერის ჰეში: 8fc9122863f27ba3cc445583435a8e04

შემდგომ, გადავიდეთ იუზერზე panda, რომელიც არის ჩვეულებრივი Domain User

![Untitled](/images/persistence-one/Untitled%203.png)

![Untitled](/images/persistence-one/Untitled%204.png)

დავრწმუნდეთ, რომ არ გვაქვს წვდომა დომეინ კონტროლერზე



![Untitled](/images/persistence-one/Untitled%205.png)

ვინაიდან და რადგანაც გვაქვს krbtgt იუზერის ჰეში შეგვილია შევქმნათ TGT თიქეთები ჩვენი სურვილისამებრ.

ეს შეგვიძლია 2 ხელსაწყოს დახმარებით:

    mimikatz
    Rubeus

ავირჩიოთ ამ შემთხვევაში mimikatz ხელსაწყო:


kerberos::golden /user:Administrator /domain:test.local /sid:S-1-5-21-3928239400-634352526-1646374684 /krbtgt:8fc9122863f27ba3cc445583435a8e04 /ptt

![Untitled](/images/persistence-one/Untitled%206.png)

აქ ვხედავთ, რომ შევქმნენით ახალი თიქეთი და გავაკეთეთ ინექცია ჩვენს სესიაში როგორც ეს ჩანს ქვემოთ:

![Untitled](/images/persistence-one/Untitled%207.png)

წესით ჩვენ უნდა გვქონდეს უკვე უფლება მივწვდეთ დომეინ კონტროლერის C შეარს თუმცა სანამ ამას მიზავთ პირველ რიგში ჩავრთოთ wireshark.

![Untitled](/images/persistence-one/Untitled%208.png)

როგორც ვხედავთ აქ მივიღეთ Access Denied რაც უცნაურია რადგანაც თითქოს არაფერი შეცდომა არ დაგვიშვია. ასეთ შემთხვევაში დაგვჭირდება პრობლემის მოძიება. ერთ-ერთი ხელსაწყო ამისათვის არის wireshark და მოდი ჩავხედოთ რა ხდება:

![Untitled](/images/persistence-one/Untitled%209.png)

Microsoft - ის პაჩი: CVE-2021-42287

როგორც სურათიდან მიხვდით მაშინ როცა მოვითხოვეთ CIFS სერვისზე წვდომა(იგივე შეარზე წვდომა) და თან გავაყოლეთ ჩვენი TGT, დომეინ კონტროლერმა დაგვიბრუნა ერორი: KRB5KDC_ERR_TGT_REVOKED

და ასევე ჩვენს სესიაში წაიშალა(და-revoked-და) თიქეთი

![Untitled](/images/persistence-one/Untitled%2010.png)

დომეინ კონტროლერზე თუ შევიჭყიტებით ვნახავთ, რომ დაგენერირდა Event ლოგი ID ით: 37

![Untitled](/images/persistence-one/Untitled%2011.png)

მთელი “პრობლემა” იმაშია, რომ Microsoft - მა გამოუშვა პაჩი(CVE-2021-42287), რომელიც ნაწილობრივ აგვარებს Golden Ticket - ების პრობლემას. ამ პაჩის წყალობით PAC სტრუქტურაში(სადაც ინახება იუზერის ინფო) დაემატა დამატებითი სტრუქტურა: PAC_REQUESTOR

სქრინშოტის ორიგინალი ბმული: https://www.varonis.com/blog/pac_requestor-and-golden-ticket-attacks

![Untitled](/images/persistence-one/Untitled%2012.png)

PAC_REQUESTOR სტრუქტურაში ჩაიწერება მომთხოვნი იუზერის ID (4191 სქრინშოტის მიხედვით) და როდესაც მოვითხოვთ TGT - ს მაგალითად იუზერ: “Administrator” - ზე. დომეინ კონტროლერი შეადარებს Administrator - ის აიდის და 4191 - ს და თუ არ დაემთხვა დააერორებს.

მარტივად რომ ავხსნათ: აქამდე შესაძლებელი იყო golden ticket - ის დაგენერირება ნებისმიერი იუზერის მითითებით(მათ შორის არარსებული დომეინ იუზერის). ახლა პრაქტიკულად ის შეიცვალა, რომ ამ პაჩის წყალობით აღარ შეგვეძლება ნებისმიერი იუზერის მითითება და აწი მხოლოდ დომეინ იუზერებით უნდა შემოვიფარგლოთ.

P.S security - ის მხრივ დიდად არაფერი შეცვლილა მე თუ მკითხავთ :)

მაშ ალბათ გაგიჩნდება კითხვა ეს თუ ასეა მაშინ ჩვენ დაგენერირებულ TGT თიქეთზე Administrator გვაქვს იუზერი მითითებული და User ID: 500. როგორც წესი Administrator - ის აიდი ზუსტად რომ 500 - ია.

![Untitled](/images/persistence-one/Untitled%2013.png)

მაშ რაშია საქმეა?

საქმე იმაშია, რომ mimikatz როცა ვიყენებთ გენერირდება თიქეთი, რომელსაც აქვს ძველი(პაჩამდე) PAC სტრუქტურა. ამ ხელსაწყოს განახლება არ მომხდარა და იგი ჯერ ასე აგენერირებს თიქეთებს. იდეაში ამაზე უკვე იზრუნეს:

![Untitled](/images/persistence-one/Untitled%2014.png)

ამ ხელსაწყოს დაემატა პარამეტრი: /newPAC

რომელიც დააგენერირებს თიქეთებს განახლებული სტრუქტურით. თუმცა ჩემი ტესტირებიდან გამომდინარე რატომღაც არ არის სტაბილური. შესაბამისად შევარჩიე მეორე ხელსაწყო ამისათვის: Rubeus.exe

საბედნიეროდ ამ ხელსაწყოზე უკეთესად იზრუნეს 🙂

![Untitled](/images/persistence-one/Untitled%2015.png)

როგორც უკვე ხედავთ ჩვენ უკვე გვაქვს წვდომა შეარზე:

![Untitled](/images/persistence-one/Untitled%2016.png)

და თუ მოგვინდება DCSync საც გავაკეთებთ :)))

![Untitled](/images/persistence-one/Untitled%2017.png)

Skeleton Keys

Skeleton Keys არის ერთ-ერთი მეთოდი persistence - სთვის, რომელიც შემუშავებულია ერთ-ერთი APT - ის მიერ. მთელი არსი მდგომარეობს იმაში, რომ დომეინ კონტროლერის lsass.exe პროცესი “იპაჩება” ანუ იცვლება ისეთნაირად, რომ ნებისმიერ იუზერზე შეგვეძლება შესვლა იმ პაროლით რითიც ჩვენ გვინდა.

mimikatz - ის დახმარებით შეგვიძლია ეს გავაკეთოთ თუმცა პაროლი მას დაჰარდკოდებული აქვს და ჩვენით ვერ მივუთითებთ.

![Untitled](/images/persistence-one/Untitled%2018.png)

პირველ რიგში დავრწმუნდეთ რომ წვდომა არ გვაქვს

![Untitled](/images/persistence-one/Untitled%2018.png)

ვასრულებთ შემდეგ ბრძანებას და ვუთითებთ პაროლს: “mimikatz”

runas /netonly /user:test\Administrator cmd.exe

![Untitled](/images/persistence-one/Untitled%2019.png)

ქვემოთ ვხედავთ, რომ პაროლი: “mimikatz” ის მეშვეობით შევედით შეარზე

![Untitled](/images/persistence-one/Untitled%2020.png)

ეს პრობლემა თუკი რეალურ გარემოში მოხდება, ამის განეიტრალება მხოლოდ დომეინ კონტროლერის რესტარტით შეიძლება, რადგანაც ჯერჯერობით არ არსებობს ისეთი skeleton key, რომ რესტარტი გადალახოს.
Abuse AdminSDHolder

დომეინ ადმინის ჯგუფში ახალი იუზერის დამატება აუცილებლად გამოიწვევს ყურადღებას. მეორე გზა არის Domain Admins ობიექტის DACL - ში გავაკეთოთ ახალი ჩანაწერი, რომელიც მისცემს ჩვენს ახალ იუზერს იმის უნარს, რომ აკონტროლოს დომეინ ადმინის ჯგუფის წევრობა.

მაგალითად, ავიღოთ იუზერი panda და მას მივანიჭოთ GenericALL ფერმიშენი Domain Admins ჯგუფზე. პირველ რიგში ვნახოთ, რომ ესეთი ფერმიშენი (ტექნიკურად უფრო სწორად იქნება ACE ჩანაწერი) არ გვაქვს:

![Untitled](/images/persistence-one/Untitled%2021.png)

ვამატებთ ახალ ACE ჩანაწერს PowerView - ს დახმარებით

Add-DomainObjectAcl -TargetIdentity 'CN=Domain Admins,CN=Users,DC=test,DC=local' -PrincipalIdentity panda -Rights All -PrincipalDomain test.local -TargetDomain test.local -Verbose

![Untitled](/images/persistence-one/Untitled%2022.png)

როგორც ვხედავთ დაემატა ახალი ჩანაწერი:

![Untitled](/images/persistence-one/Untitled%2023.png)

ესეთი ჩანაწერი შეგვიძლია PowerView დახმარებითაც ვნახოთ:

get-domainobjectacl "Domain Admins" | ?{$_.SecurityIdentifier -eq 'S-1-5-21-3928239400-634352526-1646374684-1116'}

![Untitled](/images/persistence-one/Untitled%2024.png)

რადგანაც გვაქვს სრული უფლებები ამ ჯგუფზე ვცადოთ იუზერის დამატება. პირველ რიგში ვნახოთ ლისტი დომეინ ადმინების

![Untitled](/images/persistence-one/Untitled%2025.png)

ვამატებთ იუზერს batman

![Untitled](/images/persistence-one/Untitled%2026.png)

თუმცა ესეთი persistence 1 საათზე მეტხანს არასდროს გაგრძელდება ვინაიდან და რადგანაც Active Directory - ს აქვს ფუნქცია, რომელიც იცავს პრივილეგირებულ ჯგუფებს როგორიცაა:

Domain Admins
Enterprise Admins
Domain Controllers
და ა.შ

![Untitled](/images/persistence-one/Untitled%2027.png)

შესაბამისად როგორც სურათზე ხედავთ ჩვენი დამატებული ACE ჩანაწერი წაიშალა.

ეს პროცესი ხდება AdminSDHolder კონტეინერის პერმიშენების გადაწერა პრივილეგირებულ ჯგუფებზე. ამ შემთხვევაში ჩვენ გვაქვს დომეინ ადმინის ჯგუფი.

![Untitled](/images/persistence-one/Untitled%2028.png)

თუკი ერთმანეთს გადაადარებთ მიხვდებით, რომ იდენტური ფერმიშენები აქვთ.

შესაბამისად, პრივილეგირებულ ჯგუფებზე (ასევე ეძახიან protected groups) შეცვლილი DACL დიდად შედეგს არ მოგვიტანს. საჭიროა AdminSDHolder ზე შეცვლა:

![Untitled](/images/persistence-one/Untitled%2029.png)

![Untitled](/images/persistence-one/Untitled%2030.png)

როგორც ნახეთ შევცვალეთ AdminSDHolder კონტეინერის DACL. ახლა გვიწევს დალოდება თუ როდის გადაეწერება ეს ფერმიშენი სხვა პრივილეგირებულ ჯგუფებს. თუმცა ასე რომ არ ველოდოთ შეგვილია გამოვიყენოთ ხელსაწყო:

. .\Invoke-SDPropagator.ps1
Invoke-SDPropagator -timeoutMinutes 1 -showProgress -Verbose

![Untitled](/images/persistence-one/Untitled%2031.png)

![Untitled](/images/persistence-one/Untitled%2032.png)

![Untitled](/images/persistence-one/Untitled%2033.png)

თუმცა გარდა დომეინ ადმინისა, გადაეწერა Enterprise Admin - ების ჯგუფსაც

![Untitled](/images/persistence-one/Untitled%2034.png)

და ბოლოს, ესეთი შემთხვევები აგენერირებენ event ლოგს აიდით: 4737

![Untitled](/images/persistence-one/Untitled%2035.png)

Abuse Domain Object DACL

და ბოლოს, დომეინ ადმინის უფლებებს როდესაც მოიპოვებს შემტევი პირი, მას შეუძლია მთავარ დომეინ ობიექტზე:

![Untitled](/images/persistence-one/Untitled%2036.png)

შეცვალოს ფერმიშენები ისე, რომ ნებისმიერ იუზერს მისცეს DCSync უფლებები:

![Untitled](/images/persistence-one/Untitled%2037.png)

ვნახოთ PowerView - ს დახმარებით

Get-DomainObjectAcl  'DC=test,DC=local' -resolveguids | ?{$_.SecurityIdentifier -eq 'S-1-5-21-3928239400-634352526-1646374684-1118'}

![Untitled](/images/persistence-one/Untitled%2038.png)

![Untitled](/images/persistence-one/Untitled%2039.png)

![Untitled](/images/persistence-one/Untitled%2040.png)

როგორც სურათიდან ხედავთ მიუხედავად იმისა, რომ აბსოლიტურად ჩვეულებრივი იუზერი ვართ, შეგვიძლია DCSync ოპერაციის განხორციელება

![Untitled](/images/persistence-one/Untitled%2041.png)

შესაბამისად, რეკომენდაციები:

    ნუ ჩათვლით, რომ რადგან დომეინ ადმინის ჯგუფში არავინ ზის უსაფრთხოთ ხართ
    ACL - ებს თითქმის არავინ ამოწმებს. შეამოწმეთ.
    აკონტროლეთ AdminSDHolder ის კონტეინერი
    განაახლეთ სისტემები.
    პერიუდულად ჩაატარეთ შეღწევადობის ტესტირება ორგანიზაციაში
    AD - ს პენტესტის შემდგომ, სასურველია თუკი krbtgt იუზერის პაროლს შეცვლით ზედიზედ ორჯერ ( ერთხელ არ ცვლის ).
