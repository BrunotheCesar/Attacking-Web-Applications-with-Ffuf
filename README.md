# Attacking Web Applications with Ffuf
In this walktrough, we will use a list to send requests to a webserver, in hopes to find unknown and/or hidden web pages. Then, we will interact with it and observe how the webserver reacts to our requests. 

The HTTP response status codes will indicate when an attempt is successful. Status codes to keep an eye on are:

**200 OK - Successful request**

**404 Not found - Server did not find the requested resource**

**403 - Access denied**




## The website we are fuzzing today, initially do not gives us information of existing pages, and when viewing the page source, we confirm that the landing page is not linked to any other page.

![1](https://github.com/LaraBruno/Attacking-Web-Applications-with-Ffuf/assets/37584600/7c044473-44b9-4ca0-9524-383bd1686e06)


Lets use Ffuf and investigate it further.

The main two variables required from ffuf are:

**Wordlists (-w)**

**URL (-u)**

Using a Kali pre loaded wordlist, we will proceed using the following syntax:

> $ ffuf -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt:FUZZ -u http://159.65.20.216:30024/FUZZ

We find two pages, forum and blog, with response code 301, indicating that it has been moved permanently.
We can access these pages, although they are empty and do not return interesting information.

![2](https://github.com/LaraBruno/Attacking-Web-Applications-with-Ffuf/assets/37584600/2866582f-c8f4-48d5-bcdf-9c099d889201)


We are now going to fuzz on the pages we found, but first, we need to find what type of page the website uses (.html, .php, etc.).

Let's run the ffuf tool with a web extension wordlist and check if it returns the page extension for their index page.

> $ ffuf -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://159.65.20.216:30024/blog/indexFUZZ

![3](https://github.com/LaraBruno/Attacking-Web-Applications-with-Ffuf/assets/37584600/4ccfd2f7-66e3-4c4a-9a5f-d7096ea43c82)


Success! We received a 200 HTTP response status code, indicating this page runs on PHP

We will now fuzz from within the blog page we initially found, looking for .php pages.

> $ ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://159.65.20.216:30024/blog/FUZZ.php

Success! We found home.php. Let's check manually what's in that webpage:

![4](https://github.com/LaraBruno/Attacking-Web-Applications-with-Ffuf/assets/37584600/7d81c675-c281-4e51-af5d-b324e2f50463)


# Recursive Fuzzing

Considering real world scenarios where websites have several directories, sub-directories and files, it would be time consuming to manually fuzz each page individually. To automate this process, we will perform recursive fuzzing to scan directories and their sub-directories to the depth of our choice.

> $ ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://159.65.20.216:30024/FUZZ -recursion -recursion-depth 1 -e .php -v
