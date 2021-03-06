OWASP-Juice-Shop
################
:Author: David Boyd
:Date: 2020-10-29

[Task 2] Let's Go on an Adventure!
**********************************

Questions
==========

1. What's the Administrator's email address?
--------------------------------------------
:Answer: admin@juice-sh.op

Walkthrough
^^^^^^^^^^^

**Walk through** the reviews

2. What paramater is used for searching?
----------------------------------------
:Answer: q

Knowledge: 'q' for 'query'!

3. What show does Jim reference in his review?
----------------------------------------------
:Answer: Star Trek

Walkthrough Method 1: *Walk Through*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

	1. **Walk through** the reviews searching for "jim".
	2. Google search "Fresh out of a replicator"

Walkthrough Method 2: Web Scrape Reviews
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

	1. Write a script to collect all the reviews.
	2. Search outfile for jim
	3. GOTO Walkthrough(1):Step 2

.. code-block:: Python

	# Web scrape Juice Shop's Reviews
	for i in range(36):
		url = "http://$TM/rest/products/" + str(i) + "/reviews"
		webpage = requests.get(url)
		print(webpage.json()) >> docs/Reviews-N.txt

.. code-block:: Bash

	# Terminal
	grep jim ./reviews/*

[Task 3] Inject the Juice
*************************
:OWASP: Injection

Introduction
============

SQLi
	malicious/malformed query input for the purpose to retriece/tamper data
	from DB --including account credentials.

command injection
	user input/user-controlled data into a web application that runs as system
	commands. email injection unauthorized & malicious users sending emails
	from the email server.  --occurs when extra data is added to the email
	fields, in which case, the server misinterprets.

SQL syntax
----------

single quote "'"
	closes the brackets in the SQL query.

"OR" keyword
	logical or.

double dashes "--"
	are comments.  In the case of "--" appearing at the end of an injection,
	it'll *comment out* the tailing web application's code (hopefully), and
	thus preventing any input validations.

Questions
=========
:Note: This SQLi method works for all emails found in web app (see: reviews!)

Question #1: Log into the administrator account!
------------------------------------------------
:Answer: 32a5e0f21372bcc1000a6088b93b458e41f0e02a

Walkthrough Method 1: Burp Suite
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Burp Suite's Intercept mode; then
2. replace "email":"<value>" with SQLi ' or 1=1--
3. Forward

Walkthrough Method 2: Web App
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Input *' or 1=1--* in the Email field
2. Password field required: use whatever.

Question #2: Log into the Bender account!
-----------------------------------------
:Answer: fb364762a3c102b2db932069c0e6b78e738d4066

Walkthrough
^^^^^^^^^^^

Email field: bender@juice-sh.op'--

[Task 4] Who Broke My Lock?!
****************************
:OWASP: Broken Authentication
:About: Weak passwords + high priv accts; forgotten passwords

Questions
=========

1. Bruteforce the Administrator account's password!
---------------------------------------------------
:ANSWER: c2110d06dc6f81c67cd8099ff0ba601241f1ac0e

Walkthrough
^^^^^^^^^^^

.. image:: ./docs/t4-q1-answer.png
	:alt: Burp Suite Sniper

Just make sure the login for admin is their email address, and not username.  I
wasted 3 days of re-running Burp Suite and hours of Hydra researching to
uncover this "small" overlook.  Although, I was never able to get Hydra to
properly run.

Hydra's Issues
^^^^^^^^^^^^^^
:Status: Unresolved

Hyrdra's issues with http-post-form (401 status code) and http-get (500 status
code):

.. code-block:: Bash

	# Hydra's code
	hydra -l admin -P ./best1050.txt $TM http-post-form \
	"/rest/user/login:email^CUSER^&password=^PASS^:F=Incorrect" -I

			# Hydra's Response --Reason: 2 types of OAuth
			[ERROR] the target is using HTTP auth, not a web form, received
			HTTP error code 401. Use module "http-get" instead.

	# Hydra module man page
	hydra -U http-get

	# Run http-get
	hydra -l 'admin@juice-sh.op' -P docs/best1050.txt $TM http-get
	"/rest/user/login"

			# Hydra's Response --Reason: HTTP Parameter Pollution attempts
			[WARNING] Unusual return code: 500 for admin@juice-sh.op:admin123

2. Reset Jim's password!
------------------------
:Answer: 094fbc9b48e525150ba97d05b942bbf114987257

Walkthrough
^^^^^^^^^^^
:URL: https://en.wikipedia.org/wiki/James_T._Kirk

1. Account > Forgot Password > jim@juice-sh.op
2. Research [Security Question]: eldest sibling + Star Trek
3. Reset Jim's password

[Task 5] Ah! Don't Look!
************************
:OWASP: Sensitive Data Exposure

Questions
=========

1. Access the Confidential Document!
------------------------------------
:Answer: edf9281222395a1c5fee9b89e32175f1ccf50c5b

Walkthrough
^^^^^^^^^^^

The legal document inadvertently gives us an **ftp** directory.

About > [<link>legal.md] > $TM/ftp > acquistions.md > $TM

2. Log into MC SafeSearch's Account!
------------------------------------
:Answer: 66bdcffad9e698fd534003fbb3cc7e2b7b55d7f0

Walkthrough
^^^^^^^^^^^
:Video: https://youtu.be/v59CX2DiX0Y

1. Watch the video --Its worth it ;)
2. Listen for the password

3. Download the backup file!
----------------------------
:Answer: bfc1e6b4a16579e85e06fee4c36ff8c02fb13795

Walkthrough
^^^^^^^^^^^
:Poison Null Byte: %00

1. $TM/ftp/package.json.bak%2500.md

Percent-encoding
""""""""""""""""
:RFC 3886: https://tools.ietf.org/html/rfc3986
:Wiki: https://en.wikipedia.org/wiki/Percent-encoding

RFC 3886 > 7.3 Back-End Transcoding
	"Percent-encoded octets must be decoded at some point during the
	dereference process.  Applications must split the URI into its component
	and subcomponents **prior** to decoding the octets, as **otherwise the
	decoded octets might be mistaken for delimeters.**

%00: NULL
	"...may require special handling and should be rejected if the application
	is not expecting to recieve raw data within a component."

%25: %
	To encode '%' in the URI, you'll need to encode the 'percent-sign' through
	percent-encoding.  Under percent-encoding reserved characters, you'll find
	%25 to decode to '%'.

[Task 6] Who's Flying This Thing?
*********************************

Questions
=========

1. Access the administration page!
----------------------------------
:Answer: 946a799363226a24822008503f5d1324536629a0

Walkthrough
^^^^^^^^^^^

1. Open Firefox > Menu > Web Developer > Debugger (C-S-z)
2. Open Main Thread > $TM > {} main-es2015.js > [click] {} (for readability)
3. <C-f> admin | Find the 'path' for administration
4. Deduce & GOTO URI: $TM/#/adminstration

2. View another user's shopping basket!
---------------------------------------
:Answer: 41b997a36cc33fbe4f0ba018474e19ae5ce52121

Walkthrough
^^^^^^^^^^^

1. As admin, GOTO *Your Basket*
2. Burp Suite intercept *GET /rest/basket/1*
3. Substitute User ID from *1->2*; THEN forward & turn off intercept
4. View User 2's basket!

3. Remove all 5-star reviews!
-----------------------------
:Answer: 50c97bcce0b895e446d61c83a21df371ac2266ef

Walkthrough
^^^^^^^^^^^

1. GOTO $TM/#/administration > Customer Feedback
2. [click] Trash next to the 5 star review(s)

[Task 7] Where Did That Come From?
**********************************

Intro
=====

There are three major types of XSS attacks:

+---------------+------------------------------------------------------------+
| XSS Attack    | Description                                                |
+===============+============================================================+
| DOM (Special) | DOM XSS (Document Object Model-based Cross-site Scripting) |
|               | uses the HTML environment to execute malicious javascript. |
|               | This type of attack commonly uses the <script></script>    |
|               | HTML tag.                                                  |
+---------------+------------------------------------------------------------+
| Persistent    | Persistent XSS is javascript that is run when the server   |
| (Server-side) | loads the page containing it. These can occur when the     |
|               | server does not sanitise the user data when it is uploaded |
|               | to a page. These are commonly found on blog posts.         |
+---------------+------------------------------------------------------------+
| Reflected     | Reflected XSS is javascript that is run on the client-side |
| (Client-side) | end of the web application. These are most commonly found  |
|               | when the server doesn't sanitise search data.              |
+---------------+------------------------------------------------------------+

Questions
=========

1. Perform a DOM XSS!
---------------------
:Answer: 9aaf4bbea5c30d00a1f5bbcfce4db6d4b0efe0bf

Walkthrough
^^^^^^^^^^^
:Note: iframe uses "`" grave/accent marks, not single quotes

iframe
	a common HTML element found in many web applications.

1. Input `<iframe src="javascript:alert(\`xss\`)">` into the search bar

Why does this work?
^^^^^^^^^^^^^^^^^^^

It is common practice that the search bar will send a request to the server in
which it will then send back the related information, but this is where the
flaw lies. Without correct input sanitation, we are able to perform an XSS
attack against the search bar.

2. Perform a persistent XSS!
----------------------------
:Answer: 149aa8ce13d7a4a8a931472308e269c94dc5f156

Walkthrough
^^^^^^^^^^^

1. As admin, GOTO Menu > Privacy & Security > Last Login IP
2. With Burp Suite's Interceptor, log out; intercept: *Get /rest/save/loginIP*
3. Under 'Headers' > Add

	- Name:True-Client-IP
	- Value:<iframe src="javascript=alert(`xss`)">

4. Forward; Turn Interceptor off
5. GOTO Step #1

Why do we have to send this Header?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The True-Client-IP  header is similar to the X-Forwarded-For header, both tell
the server or proxy what the IP of the client is. Due to there being no
sanitation in the header we are able to perform an XSS attack.

3. Perform a reflected XSS!
---------------------------
:Answer: 23cefee1527bde039295b2616eeb29e1edc660a0

Walkthrough
^^^^^^^^^^^
:Note: May have to terminate then reboot $TM

1. As admin, GOTO Menu > Orders & Payment > Order History > [CLICK] 'Truck'
2. Substitute: *5267-06c93c01f314c9ef -> <iframe src="javascript:alert(`xss`)>"*

Why does this work?
^^^^^^^^^^^^^^^^^^^

The server will have a lookup table or database (depending on the type of
server) for each tracking ID. As the 'id' parameter is not sanitised before it
is sent to the server, we are able to perform an XSS attack.

[Task 8] Exploration!
*********************

Questions
=========

1. Access the /#/score-board/ page
----------------------------------
:Answer: 7efd3174f9dd5baa03a7882027f2824d2f72d86e

Walkthrough
^^^^^^^^^^^

1. Don't go to */#/score-board/page*
2. GOTO */#/score-board*

