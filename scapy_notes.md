## Create a single egress TCP 443 socket:

```
>> p=IP(dst="google.com")/TCP(dport=443)
>>> sr1(p)
Begin emission:
Finished sending 1 packets.
.*
Received 2 packets, got 1 answers, remaining 0 packets
<IP  version=4 ihl=5 tos=0x0 len=44 id=0 flags=DF frag=0 ttl=123 proto=tcp chksum=0xccce src=142.250.217.78 dst=xxx.xxx.10.12 |<TCP  sport=https dport=ftp_data seq=1868052310 ack=1 dataofs=6 reserved=0 flags=SA window=65535 chksum=0xbcca urgptr=0 options=[('MSS', 1412)] |<Padding  load='\x00\x00' |>>>          

>>> p.show()
###[ IP ]### 
  version   = 4
  ihl       = None
  tos       = 0x0
  len       = None
  id        = 1
  flags     = 
  frag      = 0
  ttl       = 64
  proto     = tcp
  chksum    = None
  src       = xxx.xxx.10.12
  dst       = Net("google.com/32")
  \options   \
###[ TCP ]### 
     sport     = ftp_data
     dport     = https
     seq       = 0
     ack       = 0
     dataofs   = None
     reserved  = 0
     flags     = S
     window    = 8192
     chksum    = None
     urgptr    = 0
     options   = ''
```

- With the show() methods, we can see information of the detail of a certain package. The difference between show() and show2() is that the show2() function shows the package as it is sent by the network

```
>>> p.show2()
###[ IP ]### 
  version   = 4
  ihl       = 5
  tos       = 0x0
  len       = 40
  id        = 1
  flags     = 
  frag      = 0
  ttl       = 64
  proto     = tcp
  chksum    = 0x47d2
  src       = xxx.xxx.10.12
  dst       = 142.250.217.78
  \options   \
###[ TCP ]### 
     sport     = ftp_data
     dport     = https
     seq       = 0
     ack       = 0
     dataofs   = 5
     reserved  = 0
     flags     = S
     window    = 8192
     chksum    = 0x5b16
     urgptr    = 0
     options   = ''

>>> p[0].summary()
'IP / TCP xxx.xxx.10.12:ftp_data > Net("google.com/32"):https S'
```

## Create a TCP three-way handshake

```
>>> syn = IP(dst='www.google.com') / TCP(dport=80, flags='S')
>>> syn
<IP  frag=0 proto=tcp dst=Net("www.google.com/32") |<TCP  dport=http flags=S |>>
>>> syn_ack = sr1(syn)
Begin emission:
Finished sending 1 packets.
.*
Received 2 packets, got 1 answers, remaining 0 packets
>>> syn_ack
<IP  version=4 ihl=5 tos=0x0 len=44 id=0 flags=DF frag=0 ttl=123 proto=tcp chksum=0xccb8 src=142.250.217.100 dst=xxx.xxx.10.12 |<TCP  sport=http dport=ftp_data seq=1822455244 ack=1 dataofs=6 reserved=0 flags=SA window=65535 chksum=0x8261 urgptr=0 options=[('MSS', 1412)] |<Padding  load='\x00\x00' |>>>                                        
>>> getStr = 'GET / HTTP/1.1\r\nHost: www.google.com\r\n\r\n'
...: request = IP(dst='www.google.com') / TCP(dport=80, sport=syn_ack[TCP].dport,
...:              seq=syn_ack[TCP].ack, ack=syn_ack[TCP].seq + 1, flags='A') / getStr
...: reply = sr1(request)
Begin emission:
Finished sending 1 packets.
*
Received 1 packets, got 1 answers, remaining 0 packets
>>> getStr
'GET / HTTP/1.1\r\nHost: www.google.com\r\n\r\n'
>>> sr1(request)
Begin emission:
Finished sending 1 packets.
*
Received 1 packets, got 1 answers, remaining 0 packets
<IP  version=4 ihl=5 tos=0x0 len=40 id=0 flags=DF frag=0 ttl=123 proto=tcp chksum=0xccbc src=142.250.217.100 dst=xxx.xxx.10.12 |<TCP  sport=http dport=ftp_data seq=1822455245 ack=0 dataofs=5 reserved=0 flags=R window=0 chksum=0x99fb urgptr=0 |<Padding  load='\x00\x00\x00\x00\x00\x00' |>>>                                                     
>>> reply
<IP  version=4 ihl=5 tos=0x0 len=40 id=0 flags=DF frag=0 ttl=123 proto=tcp chksum=0xccbc src=142.250.217.100 dst=xxx.xxx.10.12 |<TCP  sport=http dport=ftp_data seq=1822455245 ack=0 dataofs=5 reserved=0 flags=R window=0 chksum=0x99fb urgptr=0 |<Padding  load='\x00\x00\x00\x00\x00\x00' |>>>                                                     
>>> 
>>> reply.show()
###[ IP ]### 
  version   = 4
  ihl       = 5
  tos       = 0x0
  len       = 40
  id        = 0
  flags     = DF
  frag      = 0
  ttl       = 123
  proto     = tcp
  chksum    = 0xccbc
  src       = 142.250.217.100
  dst       = xxx.xxx.10.12
  \options   \
###[ TCP ]### 
     sport     = http
     dport     = ftp_data
     seq       = 1822455245
     ack       = 0
     dataofs   = 5
     reserved  = 0
     flags     = R
     window    = 0
     chksum    = 0x99fb
     urgptr    = 0
     options   = ''
###[ Padding ]### 
        load      = '\x00\x00\x00\x00\x00\x00'
```
