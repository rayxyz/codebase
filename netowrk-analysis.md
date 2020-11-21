# nmap
```
ray@ray-pc:~$ nmap -v -sn 192.168.0.0/24

Starting Nmap 7.01 ( https://nmap.org ) at 2020-11-21 16:33 CST
Initiating Ping Scan at 16:33
Scanning 256 hosts [2 ports/host]
Completed Ping Scan at 16:33, 2.52s elapsed (256 total hosts)
Initiating Parallel DNS resolution of 256 hosts. at 16:33
Completed Parallel DNS resolution of 256 hosts. at 16:33, 0.06s elapsed
Nmap scan report for 192.168.0.0 [host down]
Nmap scan report for 192.168.0.1
Host is up (0.014s latency).
Nmap scan report for 192.168.0.2 [host down]
Nmap scan report for 192.168.0.3 [host down]
Nmap scan report for 192.168.0.4 [host down]
Nmap scan report for 192.168.0.5 [host down]
Nmap scan report for 192.168.0.6 [host down]
Nmap scan report for 192.168.0.7 [host down]
Nmap scan report for 192.168.0.8 [host down]
Nmap scan report for 192.168.0.9 [host down]
......
```

# mtr
```
ray@ray-pc:~$ mtr --report -i 5 -c 10 12.34.56.78
Start: Sat Nov 21 17:46:21 2020
HOST: ray-pc                      Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 192.168.0.1                0.0%    10    2.8   3.7   2.7   6.2   1.2
  2.|-- 1.96.35.121.broad.sz.gd.d  0.0%    10   11.0  12.0   7.2  27.7   5.9
  3.|-- 61.146.242.61             40.0%    10    6.2   8.3   5.3  13.0   3.4
  4.|-- 183.56.65.66               0.0%    10    9.5  12.3   9.5  20.5   3.0
  5.|-- 202.97.94.138             60.0%    10   16.4  12.7  10.8  16.4   2.4
  6.|-- 202.97.94.98              50.0%    10   21.9  23.5  19.8  26.3   2.7
  7.|-- 202.97.60.214             30.0%    10  241.3 232.4 194.5 241.3  16.7
  8.|-- ae-2.r31.tokyjp05.jp.bb.g 40.0%    10  184.7 187.0 184.6 190.3   2.4
  9.|-- ae-3.r02.tokyjp05.jp.bb.g 30.0%    10  244.3 236.8 207.1 255.2  15.3
 10.|-- ce-0-14-0-1.r02.tokyjp05. 10.0%    10  3156. 530.3 195.5 3156. 984.7
 11.|-- ???                       100.0    10    0.0   0.0   0.0   0.0   0.0
 12.|-- ???                       100.0    10    0.0   0.0   0.0   0.0   0.0
 13.|-- ???                       100.0    10    0.0   0.0   0.0   0.0   0.0
 14.|-- ???                       100.0    10    0.0   0.0   0.0   0.0   0.0
 15.|-- ???                       100.0     9    0.0   0.0   0.0   0.0   0.0
 16.|-- 12.34.56.78.vultr.com 11.1%     9  1316. 375.9 238.5 1316. 380.0
```

# iperf
```
ray@ray-pc:~$ iperf -i 2 -u -c abcxyz.com
------------------------------------------------------------
Client connecting to abcxyz.com, UDP port 5001
Sending 1470 byte datagrams
UDP buffer size:  208 KByte (default)
------------------------------------------------------------
[  3] local 192.168.0.101 port 36005 connected with 12.34.56.78 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0- 2.0 sec   257 KBytes  1.05 Mbits/sec
[  3]  2.0- 4.0 sec   256 KBytes  1.05 Mbits/sec
[  3]  4.0- 6.0 sec   256 KBytes  1.05 Mbits/sec
[  3]  6.0- 8.0 sec   257 KBytes  1.05 Mbits/sec
[  3]  8.0-10.0 sec   256 KBytes  1.05 Mbits/sec
[  3]  0.0-10.0 sec  1.25 MBytes  1.05 Mbits/sec
[  3] Sent 893 datagrams
[  3] WARNING: did not receive ack of last datagram after 10 tries.
```

# 
