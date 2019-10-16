# Parental Control and SafeBrowsing

## Initialization

Input data is a file with the list of host names that must be blocked (both PC & SB services have their own filter file):

	badsite1
	badsite2
	...

When PC/SB services are initializing they:

* get the total number of lines in file and create a hash map
* read the file line by line
* get SHA256 hash sum of the host name
* add the sum value into the hash map as shown below

Suppose that there are 2 host names with similar hash sums:

	01abcdef1234...
	01abcdef0987...

Add these hashes to the hash map like so that:

* the key equals to bytes [0..3] of each hash sum
* the value equals to an array of bytes [4..31] of each hash sum

e.g.:

	"01abcdef" -> []{"1234...", "0987..."}

## DNS messages

To check if the host is blocked, a client sends a TXT record with the Name field equal to the hash value of the host name.

	DNS Question:
	NAME=[0x08 "01abcdef" 0x00]
	TYPE=TXT
	CLASS=IN

The response to this request is the list of SHA256 hash values that start with "01abcdef".

	DNS Answers:
	[0]:
	NAME=[0x08 "01abcdef" 0x00]
	TYPE=TXT
	CLASS=IN
	TTL=1
	LENGTH=64
	DATA=["01abcdef1234...", "01abcdef0987..."]

Length is 64 because we are returning 2 SHA256 hash sums.
