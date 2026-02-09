# Bumblebee Sherlock | HackTheBox Walkthrough
> ## Tracing Credential Theft From Server Logs and Database Artifacts with DB Browser

### [>>GOOGLE DOC VERSION <<](https://docs.google.com/document/d/1I6RoKLT22Mtl4I8rHDRUh5LMxEhWrVaj-6SbZ6HRagM/edit?usp=sharing) (Originally posted on Medium.com)

*Completed 11/30/2025* -- *Jack Dignam*

- - - 
<p align="center"> <img width="300" height="300" alt="1_BCXpUHxSwXZLgMAzVhqixg" src="https://github.com/user-attachments/assets/0dd7f82a-67aa-4d4c-9166-8f9758178c9b" />
<p align="center"> https://app.hackthebox.com/sherlocks/Bumblebee

# Introduction
Welcome back to another weekly [Hack The Box](https://www.hackthebox.com/) walkthrough! Todays challenge is [Bumblebee](https://app.hackthebox.com/sherlocks/Bumblebee), created by a user named **blitztide**. It is a beginner-friendly forensic/DFIR-style challenge that provides you with a SQLite database dump plus a web server access log for investigating.

By the end of this walkthrough, you will have learned how to navigate and extract useful information from a **phpBB SQLite database**, **trace malicious behavior**, **convert timestamps**, and **interpret raw log entries** to answer forensic questions.

If you find this writeup helpful, please feel free to **drop a follow**. Thank you for your consideration, now let's do this investigation!

- - - 

# Challenge Scenario
> An external contractor has accessed the internal forum here at Forela via the Guest Wi-Fi, and they appear to have stolen credentials for the administrative user! We have attached some logs from the forum and a full database dump in sqlite3 format to help > you in your investigation.

---

# Setup the Lab Environment:
As a good rule of thumb before any simulated investigation, it is best to use a **virtual machine (VM)**. 
This ensures the environment is completely isolated and safe. This lab requires a Windows-based virtual machine which can be installed by following this tutorial (Windows 10):

[![](https://github.com/user-attachments/assets/e9091b5f-0e05-4b4c-9272-0e1e7e0ab851)](https://youtu.be/CMGa6DsGIpc?si=Dif9kTTge-xOandS)

https://youtu.be/CMGa6DsGIpc?si=Dif9kTTge-xOandS

From your Windows virtual machine, download the Hack The Box file and unzip it using the password `HackTheBlue`. In this challenge, we are provided with two files, an access log and SQLite database file. For the access log, you can utilize any text file reader such as **notepad**.

To view the SQLite database file, I will use a freely available tool called **DB Browser for SQLite**. To install this tool for your investigation, download from https://sqlitebrowser.org/.

Click the **download tab** on the top of the screen and navigate to the *Current Status* section. 
Then select the version that corresponds to your virtual machine operating system. In my case, it is 64-bit Windows 10.

From there, simply follow the installation step and import the HTB SQLite Database file by clicking `open database` from the top of the screen and importing the provided HTB file.

<img width="1000" height="613" alt="1_NxoK0lGvVovjyaEWoggCoA" src="https://github.com/user-attachments/assets/dc24b0e5-3240-4c1f-adb5-9fa8305f783f" />

With everything setup and ready to go, we can begin!

---

# Walkthrough
## Task 1: What was the username of the external contractor?
Database `.sqlite3` files are structured tables with columns containing records with indexes and metadata. All of this is stored within a binary B-tree format inside the file. **DB Browser** interprets the internal structure of SQLite files and displays them in a human-readable, column-based table view.

Once the `phpbb.sqlite3` file is imported in DB Browser, we can see a ***tables*** option under the "Database Structure" tab. Within it, contains every single table defined in the file.

Task 1 asks us to discover the username of the external contractor. This can be located under the `phpbb_users` table:

<img width="481" height="772" alt="1_6qzRFg4XQqfziebjnLTE5A" src="https://github.com/user-attachments/assets/4c9321bc-c2bf-46db-84b5-a24b4e542d93" />

If we click *Browse Table* and view its contents, we can discover the user **apoole1** has a user_email field of `apoole1@contractor.net`.

<img width="1000" height="578" alt="1_H1HPxiT5GUD17QIj7hf-0A" src="https://github.com/user-attachments/assets/1f2cdfea-ea69-4386-8891-976598358537" />

In this case, the email is very obvious that it is an external contractor email. But if it weren't, we would have to utilize OSINT or similar methods to confirm which user is a contractor.

<img width="1000" height="146" alt="1_ZbBeRARb2tmy1bNZIM_MbA" src="https://github.com/user-attachments/assets/85abcb52-53c0-4f3c-ba34-145d0d049ab9" />

--- 

## Task 2: What IP address did the contractor use to create their account?
On the same table as before, there is a `user_ip` column. If we look under the contractor use we just discovered, we can see their associated IPv4 address.

<img width="1000" height="106" alt="1_QiIcCCHF68NZvtWcqNlsCA" src="https://github.com/user-attachments/assets/0f4291ef-7c41-49a6-8f09-136754406da3" />

The contractor's IP address is discovered to be **10.10.0.78**. Note this address for the future as we will need it to identify them in the access logs.

<img width="1000" height="145" alt="1_g3fK6w5ZIrIVH9oQaCoCXw" src="https://github.com/user-attachments/assets/3eee92a8-1a3a-41b5-a59a-a08986339c0e" />

--- 

## Task 3: What is the post_id of the malicious post that the contractor made?
The `Post_id` field can be located similarly to how we found the user_ip frield. But in this case, post IDs are located under a different table.

If we return back to the **Database Structure** containing every table, we can find malicious posts in the `phpbb_posts` table.

<img width="556" height="511" alt="1_iB9SBpDvCzw8Qh2Crqf8BQ" src="https://github.com/user-attachments/assets/89c3f00f-d374-4ec8-9807-45166daee7b2" />

Within it, we can use the previously identified IP address of the external contractor (10.10.0.78) to discover their suspicious posts.

<img width="1000" height="116" alt="1_j-C2nm1jw_Znp9bx9cZh2w" src="https://github.com/user-attachments/assets/98cc65ee-d4c9-478d-9b5e-f0162183035b" />

Here, we can see that the suspicious IP address's post_id field is **9**.

<img width="1000" height="142" alt="1_lFbSRLQEGlZnKPowoLDfkQ" src="https://github.com/user-attachments/assets/60e7baf0-2be3-4e35-a112-dd03a817d97f" />

--- 

## Task 4: What is the full URI that the credential stealer sends its data to?
On the same table, we can view the contents of the post the user created under the `post_text` column.

<img width="307" height="122" alt="1_ScVKcUsNcKdjGHSf1vs5Fw" src="https://github.com/user-attachments/assets/2057d08f-0d4d-48a2-b8cd-fc17a59f14b9" />

Within this column contains the actual body of each forum post, but not in clean text. Instead, it includes the *raw* message exactly as phpBB stores it internally.

In this challenge, the contractor created a malicious post with an **embedded phishing/tracking link**. This is the attacker's **payload**. 
It contains text stating the user needs to re-login, intending to trick admins into inputting their credentials.

<img width="1000" height="433" alt="1_lcHqvBaECsq_tNYI3wQNxg" src="https://github.com/user-attachments/assets/4486042a-9fe2-4b25-ab4f-f1864b7967e6" />

The formatting is raw HTML, not BBCode. 
BBCode is used by phpBB instead of raw HTML to prevent XSS attacks, HTML and CSS injections, and script executions. 
This is because it formats options without giving them dangerous controls. 
This type of data formatting is **sanitized** because it is clean and filtered code.

<img width="1000" height="145" alt="1_L7EO7vRaGkF8USsyPlRkgA" src="https://github.com/user-attachments/assets/9c6fce44-479f-45e3-9d28-7ce6f6fe3c55" />

--- 

## Task 5: When did the contractor log into the forum as the administrator? (UTC)
Log operations such as when a user logs in can be found in the `phpbb_log` table in the **Database Structure** tab. 
Once there, you can view what logs were created by particular users.

<img width="230" height="233" alt="1_x7XLsUl506bvG16VUIh6AA" src="https://github.com/user-attachments/assets/43bbb677-07a0-49d1-b265-2797f9fb1d18" />

If we scroll down, we can see the suspicious IP address from earlier is **adding users** (`LOG_USERS_ADDED`), **creating database backups** (`LOG_DB_BACKUP`), and **logging into admin accounts** (`LOG_ADMIN_AUTH_SUCCESS`).

<img width="1000" height="452" alt="1_eQ5jOVqwhb-vf4_p61lX_w" src="https://github.com/user-attachments/assets/61057fa8-2641-44b6-bdde-129470f175b5" />

The log operation that interests us most is `LOG_ADMIN_AUTH_SUCCESS`. It lists the `log_time` as **1682506392**. If we tried inputting this as our answer for this question, it would not work. This is because it is in **epoch time**.

**Epoch time** (UNIX or POSIX time) is a method computers use to represent dates and times as a single integer. Epoch time is the number of seconds that have passed since **January 1, 1970** at **00:00:00 UTC**. This time was selected as it was an agreed-upon starting point (Unix epoch) used by Linux, macOS, Android, and more!

To get our answer to this problem, we must translate Epoch time to UTC time. Let's use [EpochConverter.io](https://www.epochconverter.io/) to convert these two time formats.

<img width="813" height="762" alt="1_iF6srHe5bGeY4rabngPezA" src="https://github.com/user-attachments/assets/fb693b40-1a2c-4f0f-9d9e-4edcd6141bf4" />

<img width="1000" height="149" alt="1_nLdE25au4DrcjYTdwW5Y6w" src="https://github.com/user-attachments/assets/76aff990-70bd-4cf1-931d-b65429fdf7a6" />

--- 

## Task 6: In the forum there are plaintext credentials for the LDAP connection, what is the password?
This question refers to a LDAP connection, which is referring to the way clients connect to **Lightweight Directory Access Protocol** (LDAP) servers. LDAP is a protocol used to access and manage directory services over a network. phpBB supports LDAP authentication, and the credentials are stored in its configuration database.

If we return back to the "Database Structure" tab and open the `phpbb_config` tab, we can discover the plaintext credentials.

<img width="220" height="250" alt="1_Mlgu3xhcjE4SiWc55Aiv7g" src="https://github.com/user-attachments/assets/1808e298-5eba-4ae2-9351-1b160c0a2755" />

<img width="672" height="117" alt="1_xetMjMkDlGP_BH2Omk_maQ" src="https://github.com/user-attachments/assets/e4679541-3507-499a-afc6-63acddeec103" />

The `ldap_password` field is **Passw0rd1**, which is a very unsafe password as it uses a common password with little to no alterations. 
Using "**leet speak**", where you replace letters with numbers or symbols, is only effective with really long passwords without dictionary words. 
The attacker likely gained access to the database and achieved administrator access by **brute forcing** this LDAP password.

<img width="1000" height="147" alt="1_jzsV0_Xvy_OTFUZcGwdYrw" src="https://github.com/user-attachments/assets/af642acb-7be1-41e8-8c3b-efb17ee09ae9" />

--- 

## Task 7: What is the user agent of the Administrator user?
**User agents** are strings containing identifying elements such as web browser information or apps that identify itself to a server. They are generally formatted as such:

```
ProductName/Version (SystemInformation) RenderingEngine/Version BrowserName/Version [OptionalDetails]
```

So an example would be:

```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36
```
Understanding what user agents are will help us find the one the administrator used.

Earlier, when we were viewing the `phpbb_users` table, we saw not only apoole1's IP address (**10.10.0.78**) but also the administrator's IP address (**10.255.254.2**).

<img width="1000" height="92" alt="1_GYwJE0JH8YufHs8aMRsiJA" src="https://github.com/user-attachments/assets/2872c5ce-61e2-4c71-9162-d1728b3a5db8" />

If we filter through the access log text file we were provided with in this challenge, we can discover its user agent that it used to access the server.

<img width="508" height="899" alt="1_gkIn0jlR7oN3yJm_MxzmmA" src="https://github.com/user-attachments/assets/063b03f2-fb34-4645-b7b9-e717db0476d9" />

<img width="1000" height="103" alt="1_1bSivXUsjwX1UC1evX5Okg" src="https://github.com/user-attachments/assets/ae3bf7f6-9b36-4fbd-a0e5-c1775deec145" />

The user agent of the Administrator user was:

```
**Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36**
```

<img width="1000" height="143" alt="1_p9AXb9K4wlS0SatiS2zK0A" src="https://github.com/user-attachments/assets/ed2d669a-302f-4b2c-9b80-3e9d2f55cf8b" />

--- 

## Task 8: What time did the contractor add themselves to the Administrator group? (UTC)
If we refer back to the `phpbb_log` table from before, we can see that the contractor added themselves to the administrator group.

<img width="1000" height="459" alt="1_823oqwuHuaGtSlhBHEIFNw" src="https://github.com/user-attachments/assets/7ef00e38-e70d-4601-bbb1-94deb5be110d" />

We can use the same process as task 5 to translate the Epoch time of `LOG_USERS_ADDED` to UTC time to see when the attacker gained access. Let's use [EpochConverter.io](https://www.epochconverter.io/) to convert these two time formats.

<img width="818" height="772" alt="1_HC7VB-VD3h7fgs0MG1fHKg" src="https://github.com/user-attachments/assets/1bbb8188-925e-4ea2-b2ba-ff411d60d9e0" />

It is revealed that the attacker gained access to the Administrator's group at **26/04/2023 10:53:51**.

<img width="1000" height="143" alt="1_XJjps4EXRlo61Bc_so4u-w" src="https://github.com/user-attachments/assets/709c36a7-409a-4d6a-aad7-9e9e7c814a67" />

--- 

## Task 9: What time did the contractor download the database backup? (UTC)
Since we know that the database was contacted using a web browser because of their user agent, we can assume that HTTP requests were made when downloading from it by the attacker.

The access log file contains a history of all network activity to the SQL database. If we search for the keyword **SQL** with any HTTP GET requests, we can discover when the attacker downloaded a database backup.

<img width="1000" height="455" alt="1_HSTpdF_EwzgT4FlLNVZlGw" src="https://github.com/user-attachments/assets/79746f85-1deb-4f51-9317-8275cf9a7202" />

<img width="1000" height="93" alt="1_FyDQakaRGgTq6Og_dckkuw" src="https://github.com/user-attachments/assets/2002a4c1-a137-483f-91aa-898dff989647" />

Time stamps are listed as **+0100** in the access log. This means it is one hour ahead of UTC time, aka **Central European Time (CET)**.

<img width="393" height="75" alt="1_D7TLrHLZevKwbdU9XR6Zhw" src="https://github.com/user-attachments/assets/79a90e4b-a1b4-4ff8-bd9f-915c67f2b0b2" />

Subtract this one hour difference to discover the UTC time of **26/04/2023 11:01:38**.

<img width="1000" height="145" alt="1_x2TcrByWd3fbcwujj-WUuA" src="https://github.com/user-attachments/assets/fc101006-95dc-4e09-a26a-535d76fa05fc" />

--- 

## Task 10: What was the size in bytes of the database backup as stated by access.log?
On the same screen from the previous question, we can see the size in bytes of the backup that was downloaded from the database server.

<img width="750" height="131" alt="1_A95sNRqH-4xg72flsZMa2Q" src="https://github.com/user-attachments/assets/04b5a4f6-22bd-4b7b-b6c7-7a22ca77b0dd" />

The size of the downloaded backup is located to the right of the HTTP OK response message (200). It is revealed to be **34707 bytes**.

<img width="1000" height="146" alt="1_ySFFojtohpc1aBE_vjPu3w" src="https://github.com/user-attachments/assets/cbb674db-cdce-4c37-bcd0-519786de7339" />

--- 

# Conclusion
<img width="864" height="794" alt="1_iWlxCaVYP12SUQpx2H8hMA" src="https://github.com/user-attachments/assets/9a1d3c78-eac9-4d2a-a44d-7da7d1dd5455" />

The [Bumblebee](https://app.hackthebox.com/sherlocks/Bumblebee) challenge on [Hack The Box](https://www.hackthebox.com/) provides a realistic forensic and DFIR-style scenario. You need to sift through **raw SQLite dumps** from a phpBB forum to track the attacker's actions. The attacker is a **rogue contractor** who illegally accessed a database, stole **administrative credentials** through a **phishing post**, and used them to gain higher access.

We used the SQLite file to examine its **users**, **posts**, **logs**, and **configuration tables** to discover how the attack was conducted. The challenge also provided us with raw web server access logs, which was used to **extract IP addresses**, **user-agents**, **timestamps**, and **download activity**. These skills sharpened our practical skills in forensic database analysis, log-file interpretation, basic CLI and scripting habits, and investigative reasoning.

This challenge is great for foundational real-world incident response experience. If you found this walkthrough helpful, please feel free to **drop a follow**. Thank you for reading!

## References:

**Hack The Box Challenge:** https://app.hackthebox.com/sherlocks/Bumblebee

**DB Browser for SQLite:** https://sqlitebrowser.org/

**Epoch Time Converter:** https://www.epochconverter.io/

---
