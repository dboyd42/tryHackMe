INRO_NETW
#########
:Author: David Boyd
:Date: 2020-10-22

[Task 1] Introduction
*********************

The aim of this room is to provide a beginner's introduction to the basic
principles of networking.

+----------------------------------------+
| Topics                                 |
+========================================+
| OSI Model                              |
+----------------------------------------+
| TCP/IP Model                           |
+----------------------------------------+
| How these models look in practice      |
+----------------------------------------+
| Introduction to basic networking tools |
+----------------------------------------+

[Task 2] The OSI Model: An Overview
***********************************

#. Which layer would choose to send data over TCP or UDP?

	- 4 - Transport Layer

#. Which layer checks received packets to make sure that they haven't been corrupted?

	- 2 - Data Link Layer

#. In which layer would data be formatted in preparation for transmission?

	- 2 - Data Link Layer

#. Which layer transmits and receives data?

	- 1 - Physical Layer

#. Which layer encrypts, compresses, or otherwise transforms the initial data to give it a standardised format?

	- 6 - Presentation Layer

#. Which layer tracks communications between the host and receiving computers?

	- 5 - Session Layer

#. Which layer accepts communication requests from applications?

	- 7 - Application Layer

#. Which layer handles logical addressing?

	- 3 - Network Layer

#. When sending data over TCP, what would you call the "bite-sized" pieces of data?

	- segments

#. [Research] Which layer would the FTP protocol communicate with?

	- 7 - Application Layer

#. Which transport layer protocol would be best suited to transmit a live video?

	- UDP

[Task 4] Wireshark
******************

Packet Layout
=============
:Prerequisites: Select packet in 1st row
:Location: Middle row

There are 5 pieces of information here.

Frame
-----
:Layer: Physical | Network Interface
:Descr: the size of the packet received in terms of bytes
:Note: Frame # comes from the ordering position in which Wireshark captured it.

Ethernet II
------------
:Layer: Data Link | Network Interface
:Descr: the transmission medium (in this case an Ethernet cable)
:Descr2: source and destination MAC addresses of the request.

Internet Protocol Version 4
----------------------------
:Layer: Network | Internet
:Desc: the source and destination IP addresses of the request.

Transmission Control Protocol
------------------------------
:Layer: Transport | Transport
:Descr: it's telling us that the protocol was TCP (and more!)

Hypertext Transfer Protocol
----------------------------
:Layer: Application | Application
:Descr: This is the HTTP GET request, which is requesting a web page from a remote server.

[Task 9] [Networking Tools] Dig
*******************************
:SYNTAX: dig <domain> @<dns-server-ip>

Dig
	allows us to manually query recursive DNS servers of our choice for
	information about domains.

It is a very useful tool for network troubleshooting.

[dig:section] ANSWER
====================

This is a lot of information. We're currently most interested in the ANSWER
section for this room; however, taking the time to learn what the rest of this
means is a very good idea. In summary, that information is telling us that we
sent it one query and successfully (i.e. No Errors) received one full answer --
which, as expected, contains the IP address for the domain name that we
queried.

Another interesting piece of information that dig gives us is the TTL (Time To
Live) of the queried DNS record. As mentioned previously, when your computer
queries a domain name, it stores the results in its local cache. The TTL of the
record tells your computer when to stop considering the record as being valid
-- i.e. when it should request the data again, rather than relying on the
cached copy.

TTL
===

The TTL can be found in the second column of the answer section:

It's important to remember that TTL (in the context of DNS caching) is measured
in seconds, so the record in the example will expire in two minutes and
thirty-seven seconds.

#. What is DNS short for?

	- Domain Name *System*

#. What is the first type of DNS server your computer would query when you search for a domain?

	- recursive DNS server			// note: computer 1st checks its local cache
		- [+] automatically known to your routers on your network.
		- [+] many ISPs maintain their own recursive servers, but
			companies such as Google and OpenDNS also control recursive servers.
		- [+] This is how your comptuer automatically knows where to send the request
			for information: details for a recursive DNS server are stored in your
			router.
		- [+] This server will also maintain a cache of results for popular domain;
			however, if the website you've requested isn't stored in the cache, the
			recrusive server will the request on to a root name server.

#. [Research] Google runs two public DNS servers. One of them can be queried with the IP 8.8.8.8, what is the IP address of the other one?

	- 8.8.4.4

