---
title: "Automating File Organization with Python and Cron: A Step-by-Step Guide"
date: 2023-12-23 20:19:04
tags:
---

<img src="/home/starvader/ashutosh_workspace/git_ws/github/ashutoshgupta.dev/source/_posts/Automating-File-Organization-with-Python-and-Cron/Brown-Minimalist-Beauty-Blog-Banner.png" alt="Landing img">

In this blog post, I'm excited to share my experience of how I automated file organization using `Python` and `Cron`. Like many others, I encountered the daunting task of managing numerous files without a proper organizational structure.

Searching for files became time-consuming, and maintaining structure seemed like an insurmountable challenge. Additionally, when attempting to push my specific files to `GitHub`, the absence of a streamlined organization made the task even more difficult.

However, by creating a `Python script` to organize the file and scheduling a `cronjob`, I was able to automate the process. Throughout this post, I'll guide you through the steps I took, from understanding `Python script` to setting up the `Cron Job`.

---

## Overview

The project is divided into three main parts:

<details data-node-type="hn-details-summary"><summary>Part 1: Understanding Python Script</summary><div data-type="detailsContent">The Python script efficiently <code>sorts all my files into separate directories based on their names</code>. It groups the files with the same name into a common folder to streamline my file organization.</div></details><details data-node-type="hn-details-summary"><summary>Part 2: Scheduling Cronjob</summary><div data-type="detailsContent">The Cronjob has been scheduled to <code>execute my Python script every day at a scheduled time</code> and after execution, <code>save the console output into a log file</code>.</div></details>

---

## Prerequisite

* Linux-based operating system (this guide assumes a `Ubuntu 22.04` environment).
    
* Python (modules used OS, Subprocess)
    

---

## Creating Script File

* Open Terminal (in Linux using `Ctrl + Alt +T` )
    
* Change the directory to your preferred location using `cd /path/to/folder`
    
* Create a new file using `touch <file_name>.py`
    
* Give Executable permission to the file using `chmod 777 <file_name>.py`
    

---

## Part 1: Understanding Python Script

* **Import** `modules`
    
    ```python
    import os
    import subprocess
    ```
    
* **Setup** `target` and `current` directory
    
    ```python
    curr_dir = "/path/to/current/directory"
    target_dir = "/path/to/target/directory"
    ```
    
    **Note:** If your files are in the same directory as your `python script` you can just use `os.getcwd()` for the `current` directory.
    
* **Walkthrough** `current` directory and `list files`
    
    ```python
    for root, dirs, files in os.walk(curr_dir):
        for file_list in files:
    			temp_file = file_list.strip(".cpp")
    ```
    
* **Create separate** `directory` for every `uniquely named file`
    
    ```python
    command = f"mkdir {target_dir}/{temp_file}"
    result = subprocess.run(command, stdout=subprocess.PIPE, shell=True, universal_newlines=True)
    print(result.stdout) #for testing the output
    ```
    
* **Copy files from** `current` directory to their respective `directory`
    
    ```python
    command1 = f"cp {curr_dir}/{file_list} {target_dir}/{temp_file}/"
    result = subprocess.run(command1, stdout=subprocess.PIPE, shell=True, universal_newlines=True)
    print(result.stdout) #for testing the output
    ```
    
* **Setup** `permissions`
    
    ```python
    command3 = f"chmod a+rwx {target_dir}/{temp_file}"
    result = subprocess.run(command3, stdout=subprocess.PIPE,
    shell=True, universal_newlines=True)
    print(result.stdout) #for testing the output
    ```
    
* **Print current** `date` and `time`
    
* This will be used as our log file
    
    ```python
    command4 = "date"
    result = subprocess.run(command4, stdout=subprocess.PIPE, shell=True, universal_newlines=True)
    print(result.stdout)
    ```
    

---

## Script Content

```python
import os
import subprocess

curr_dir = (f"/home/starvader/ashutosh_workspace/cp_ws/codechef/codechef_code")
print(curr_dir)
target_dir = "/home/starvader/ashutosh_workspace/git_ws/github/CPP-Practice/Codechef_Solution"

for root,dirs,files in os.walk(curr_dir):
    for file_list in files:
        temp_file = file_list.strip(".cpp")
        command = (f"mkdir {target_dir}/{temp_file}")
        result = subprocess.run(command, stdout=subprocess.PIPE, shell=True, universal_newlines=True)
        command1 = (f"cp {curr_dir}/{file_list} {target_dir}/{temp_file}/")
        result = subprocess.run(command1, stdout=subprocess.PIPE, shell=True, universal_newlines=True)
        command3 = (f"chmod a+rwx {target_dir}/{temp_file}")
        result = subprocess.run(command3, stdout=subprocess.PIPE, shell=True, universal_newlines=True)     
        
command4 = (f"date")
result = subprocess.run(command4, stdout=subprocess.PIPE, shell=True, universal_newlines=True)     
print(result.stdout)
```

---

## Testing Script

To check whether your script is working correctly. Execute :

`python -m /path/to/folder/<file_name>.py`

---

## Part 2: Scheduling Cronjob

### Understanding `Cron` and `crontab`

* **Cron**: It is a service or utility used for scheduling and automating an execution of a script at a particular time.
    
* **Crontab:** It is a file that contains a set of commands which tells when to execute and how many times to execute.
    

### Accessing `crontab`

* In the terminal, write `crontab -e` and press `Enter`
    
* Select your preferred `editor` from the options (only for the first time)
    

### Adding `cronjob`

* Syntax: `* * * * * command_to_be_executed`
    
* Explanation: The `cron` syntax consists of five fields representing `minute`, `hour`, `day of month`, `month` and `day of week` respectively. Using appropriate values or wildcards (`*`), you can specify the desired schedule.
    

### Scheduling `Python Script` and `log file`

* Syntax: `* * * * * python -u "/path/to/your/script.py" >> /path/to/<file_name>.txt`
    
* Example: `20 17 * * * python -u "/home/starvader/ashutosh_workspace/Project/file_to_folder/script.py" >> /home/starvader/ashutosh_workspace/Project/file_tO_folder/codechef_log.txt`
    
* Exit the `editor.`
    

### Verifying `cronjob`

* In the terminal, write `crontab -l` and press `Enter` , your `cronjob` will be visible to you.
    

---

## Logfile Output

```

```

---

## Conclusion

By setting up and scheduling the `cronjob`, the `Python script` will run automatically every day at the specified time. It will `organize files`, `create folders` based on file names, `copy files` to their respective folders, `set appropriate permissions`and store the execution details in a `log file`. This daily routine simplifies and automates the file organization process, improving efficiency and maintaining an organized file structure.

---

Thank you for reading!