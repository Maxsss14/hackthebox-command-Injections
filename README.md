# Comprehensive Write-Up for the HTB Bug Bounty Hunter Certification Skill Assessments

## Introduction

This document provides a thorough analysis of the skill assessments linked to the HTB Bug Bounty Hunter Certification. Each assessment is approached as a distinct penetration test, focusing on a specific vulnerability and offering recommendations for remediation.

### Command Injection Skill Assessment

**Scenario Overview:**

You have been hired to conduct a penetration test for a company, and during your assessment, you encounter a file management web application. Given that file managers often execute system-level commands, it is essential to investigate potential command injection vulnerabilities.

Utilize the various techniques introduced in this module to identify a command injection vulnerability, exploit it, and bypass any security filters present.

**Target of Interest:**

"94.237.xx.xxx:xxxx"

### Reconnaissance Analysis

Initially, it's crucial to engage with the application in the same manner as a typical user to comprehend its functionalities.

The file manager web application appears to interface with the operating system, enabling us to view files hosted on the server.

![q4](https://github.com/user-attachments/assets/4630e27c-bddb-4e8b-a93b-47fd5b1ac5a8)

Upon clicking on the **tmp** folder, I observed that the URL parameter reflects as `to=tmp`.  
![1_sP63QruxRhfbTcO9YVNyXA](https://github.com/user-attachments/assets/59ef09be-f7b8-4d57-ae3a-62aecaf0b1a9)

This behavior is also noted when accessing a .txt file, where the parameter appears as `to=&view=the .txt file`.  
![1_J-RQt28WxAwk5Wa5-wT5CQ](https://github.com/user-attachments/assets/380f5e5b-af7d-4f72-b085-d875a704f593)

The application features several action buttons for files and folders, including a "copy to" option.  
![1_gjQ3JZfX-MGnxSe0qEu3pg](https://github.com/user-attachments/assets/7d2d989d-c1dc-40cd-80d5-59db260dfaf7)

This functionality enables users to copy files to other directories.  
![1_r5KkM70_NqJH2h7LhVrpQA](https://github.com/user-attachments/assets/daeecf1b-bf80-4078-a102-71ee641a2e89)

The HTTP request structure for moving a file to a directory is as follows:  
**Link:** "http://94.237.49.166:46423/index.php?to=tmp&from=51459716.txt&finish=1&move=1"  
![1_rD7m-OmRqTgTcoTCIt09pw](https://github.com/user-attachments/assets/23ec78ec-92da-4abb-9928-638f6f414487)

Note that once the file is moved from one directory, it can’t be re-performed in the same action as the file has been moved.  
![1_qVTBRRDDLA6BJDUSu86tSw](https://github.com/user-attachments/assets/46c09bbc-c89c-4a1c-b950-2270bbf6c6a8)

You can also see advanced search next to the normal search bar.  
![1_68X6VX6zfQBHoae9zy5QmA](https://github.com/user-attachments/assets/80acbefa-01e8-416c-9453-33fc2085bfd4)

![1_ZffheKduXtvwx1x9oT7tWQ](https://github.com/user-attachments/assets/0983529a-c562-45ef-bbb9-6791d4f1230d)

### Attack Vector

Now we have tested all functional areas, we want to target a few function areas and URL parameters to see if we can perform command injection like the moving file function area.

Below is the initial move of a file to a directory on the UI and the HTTP request.  
![1__9sjeuekvcmy2ap3PO5x6g](https://github.com/user-attachments/assets/94daae8f-ce9a-482b-9b92-6fbec09ea931)

![1_ncY80ky1RSacQ08pZvG0Ig](https://github.com/user-attachments/assets/2d8cddeb-4bb1-4720-a061-6bac9c63af17)

Below is the repeated action on the moving of the same file to the same directory on the UI and the HTTP request.  
![1_tLiphvDYbtwb8ctXiJ4mcg](https://github.com/user-attachments/assets/cb117f97-8f43-4e5c-9bf3-dc0b14d97c06)

![1_Do-CEGUOvPj5x7fATCfogw](https://github.com/user-attachments/assets/3c20054b-0170-4ff5-a154-528f5eb36ddf)

We want to detect the command injection in the parameter within the URL.

### Command Injection Methods

To inject an additional command into the intended one, we may use any of the following operators:  
![1_6RqtiecXLibfJG7bNnbggQ](https://github.com/user-attachments/assets/8e5a7ced-3d20-453b-9c21-9175d2e7fe26)

As I am trying to see if the “to” parameter is by injecting command injection and see whoami and ls. The error message updated to “malicious request denied”.  
![1_AugDlPIamC-dViU1mxSZog](https://github.com/user-attachments/assets/1e147d29-cfdb-400e-9811-e61a98f5b7f0)

![1_6lfoA-aykZLJy4O0FylvfA](https://github.com/user-attachments/assets/82451731-5b66-4185-b145-1d53d404ac7b)

This shows that basic commands are filtered. In this case, we will need to identify possible filters, which can include space, blacklisted characters, blacklisted commands, command obfuscation, and WAF filters.

I am going to utilize advanced command obfuscation with base64 encoding.

```bash
bash<<<$(base64 -d<<<"base64 encoded OS payload")
