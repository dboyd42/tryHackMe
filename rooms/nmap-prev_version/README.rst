Nmap
####
:Author: David Boyd
:Date: 2020-10-06

httponly flag

The Find-All Scan
=================

.. code-block:: Bash

	# -A(ll) -T(emplate)5[insane] --script <category> -o(utput)N(ormal) ip.addr
	nmap -A -T5 --script vuln -oN file.out $TM


Scripts
=======
:Summary: a light scripting engine
:Hardcore: Metasploit

.. code-block:: Bash

	# Run all scripts our of the vulnerability category		# takes a long time
	--script vuln
	# Run all scripts our of the http category
	--script http

	# Find information about script
	--script-help http-slowloris-check

