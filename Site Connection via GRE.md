
<!-- #$34T# = Your Monitor Number -->

### REMOVE L3 OPERATIONS
~~~
!@EDGE
conf t
 no ip routing
 end
~~~

## Site to Site connection via VTI (GRE)

### SET GATEWAY
~~~
!@EDGE
conf t
 int g0/0/1
  ip add 200.0.0.#$34T# 255.255.255.0
  no shut
  exit
 ip route 0.0.0.0 0.0.0.0 200.0.0.1
 end
~~~

&nbsp;
---
&nbsp;

### CONFIGURE NAT
1. Define INSIDE and OUTSIDE
~~~
!@EDGE
conf t
 int g0/0/1
  ip nat outside
 int g0/0/0
  ip nat inside
~~~

<br>

2. Create an ACL to match traffic to be translated.
~~~
!@EDGE-~~       MARK AN EXCLAMATION !
conf t
 int g0/0/0
  ip nat inside
 int g0/0/1
  ip nat outside
 ip access-list NAT-TRAFFIC
  deny ip 10.~~.0.0 0.0.255.255 10.11.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.12.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.21.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.22.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.31.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.32.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.41.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.42.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.51.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.52.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.61.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.62.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.71.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.72.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.81.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.82.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.91.0.0 0.0.255.255
  deny ip 10.~~.0.0 0.0.255.255 10.92.0.0 0.0.255.255
  permit ip any any
  end
~~~

<br>

3. Define NAT
~~~
!@EDGE
conf t
 ip nat inside source list NAT-TRAFFIC int g0/0/1 overload
 end
~~~

<br>
<br>

Verify: 
~~~
!@TAAS, BABA, CUCM, EDGE
ping 8.8.8.8
ping 8.8.4.4
~~~

~~~
!@EDGE
show ip nat translations
~~~

&nbsp;
---
&nbsp;

### MODIFY ROUTING FOR INSIDE
~~~
!@EDGE
conf t
 no router ospf 1
router ospf 1
 router-id #$3AT#.0.0.1
 network 10.#$3AT#.#$3AT#.0 0.0.0.255 area 0
 default-information originate always
 end
~~~

&nbsp;
---
&nbsp;

### CONFIGURE GRE TUNNELS

&nbsp;
---
&nbsp;


### SET STATIC ROUTING THROUGH GRE TUNNELS

<br>
<br>

Verify:

~~~
!@TAAS, BABA, CUCM, EDGE
traceroute 10.11.1.10
traceroute 10.12.1.10
~~~

&nbsp;
---
&nbsp;

### SET NAMESERVER 
> [!IMPORTANT] 
> MUST be WINSERVER.  
> NOT a public DNS

<br>

~~~
!@TAAS, BABA, CUCM, EDGE
conf t
 ip domain lookup
 ip name-server 10.#$34T.1.10
 end
~~~

<br>

Configure WINSERVER with a DNS Forwarder for 8.8.8.8 and 1.1.1.1
