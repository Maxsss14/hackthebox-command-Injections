# Hack The Box: Command Injections

**Detailed guide for the Hack The Box Bug Bounty Command Injections skill assessments. This write-up focuses on identifying and exploiting command injection vulnerabilities.**



## Command Injection Skill Assessment

### Scenario Overview

In this skill assessment for the HTB Bug Bounty Hunter Certification, you are tasked with conducting a penetration test for a client. During your assessment, you discover a file manager web application. Given that file managers often execute system commands, this presents an opportunity to investigate potential command injection vulnerabilities.

Your objective is to utilize the techniques covered in this module to identify and exploit a command injection vulnerability while bypassing any existing filters.
![q4](https://github.com/user-attachments/assets/4630e27c-bddb-4e8b-a93b-47fd5b1ac5a8)

### Target Information

- **Scoped Target**: `94.237.xx.xxx:xxxx`

### Reconnaissance

To begin, we will interact with the application as a regular user to understand its functionalities. The file manager allows us to see files stored on the server at the operating system level.

While examining the application, I noticed the following:

- When I clicked on the "tmp" folder, the URL updated to reflect the parameter `to=tmp`.
- Accessing a `.txt` file similarly updated the URL to `to=&view=the.txt`.

The application features several action buttons for each file and folder, including a "copy to" button, which permits users to copy files to different directories.

For example, the following request moves a file:

```
http://94.237.49.166:46423/index.php?to=tmp&from=51459716.txt&finish=1&move=1
```

It’s important to note that once a file is moved, you cannot perform the action again on the same file.

Additionally, there is an advanced search option next to the standard search bar, which might be of interest.

### Attack Vector

Having tested the various functionalities, I will focus on a few specific areas of the application and the corresponding URL parameters to see if command injection can be performed in the file-moving functionality.

We need to investigate the "to" parameter in the URL for potential command injection.

### Command Injection Techniques

To inject additional commands into the intended operation, various operators can be utilized. 

When I attempted to inject commands using the `to` parameter, such as:

```bash
; whoami; ls
```

I received an error stating, “malicious request denied.” This indicates that certain basic commands are filtered.

Next, I identified the filtering mechanisms that may include restrictions on spaces, blacklisted characters, blocked commands, command obfuscation techniques, and Web Application Firewall (WAF) filters.

To circumvent these filters, I will apply advanced command obfuscation techniques, including Base64 encoding. For example, I can use:

```bash
bash<<<$(base64 -d<<<"base64_encoded_payload")
```

After executing `whoami`, the output reveals that the current user is `www-data`. I will further test with `ls -la` to list the files in the directory.

### Exploitation

Now that I have identified the attack vector and recognized the filtering mechanisms, it’s time to proceed with the exploit. My goal is to check the contents of the root directory.

I discover files such as `config.php`, `files`, and `index.php`, but I cannot find the target file named `flag.txt`. Therefore, I will attempt to move the `/flag.txt` file to the `tmp` directory to inspect its contents.

To do this, I will use the following payload:

```bash
mv ${PATH:0:1}flag.txt ${PATH:0:1}var${PATH:0:1}www${PATH:0:1}html${PATH:0:1}files${PATH:0:1}tmp
```

Encoded, this becomes:

```bash
bXYgJHtQQVRIOjA6MX1mbGFnLnR4dCAke1BBVEg6MDoxfXZhciR7UEFUSDowOjF9d3d3JHtQQVRIOjA6MX1odG1sJHtQQVRIOjA6MX1maWxlcyR7UEFUSDowOjF9dG1w
```

Executing the command to move the file reveals that while the `flag.txt` can indeed be relocated to the `tmp` folder, I lack the necessary permissions to complete the action. However, this indicates that we have discovered the location of the `flag.txt` file, even if we cannot see it directly.

Now, to retrieve the contents of the `flag.txt`, I will construct a payload that incorporates Base64 encoding, bypassing any blacklisted characters, and using backslashes to overcome single-character filters:

```bash
%26c\a\t%09${PATH:0:1}flag.txt
```

Here, the `&cat` command is combined with a horizontal tab to access the flag file.

### Consequences of Command Injection

Command injection vulnerabilities are among the most severe types of security flaws. They enable an attacker to execute operating system-level commands directly on the back-end server, potentially leading to a complete network compromise. If a web application processes user-controlled input to execute system commands, it may be possible to inject malicious payloads that disrupt the intended command execution.

### Remediation Strategies

With a comprehensive understanding of how command injection vulnerabilities manifest and how certain mitigation techniques, such as character and command filtering, can be bypassed, we can outline several preventative measures:

- **Avoid Executing System Commands**: Wherever possible, refrain from executing OS-level commands based on user input.
- **Input Validation**: Ensure thorough validation of all user inputs to prevent unexpected command execution.
- **Input Sanitization**: Implement sanitization measures to clean user inputs before processing.
- **Secure Server Configuration**: Properly configure your web server to limit the execution of potentially harmful commands.

By following these guidelines, developers and security professionals can significantly reduce the risk of command injection vulnerabilities in their web applications.

---
```

### Key Formatting Elements:
- **Headings**: Clear headings for each section to improve navigation.
- **Code Blocks**: Utilized triple backticks to format commands and URLs for clarity.
- **Bullet Points**: Used lists to organize information in a digestible format.
- **Bold Text**: Highlighted important points for emphasis.

This structure will enhance the readability of your GitHub write-up while presenting the technical details clearly and professionally.
