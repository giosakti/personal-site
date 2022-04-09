---
title: "Networking"
authors: ["gio"]
---

#### TCP Dump

- Capture pcap file using `tcpdump -i <interface> -s 65535 -w <file>`
- Then open it using wireshark
- Other options is to use [tcpick(8)](https://linux.die.net/man/8/tcpick)

References:

- [A tcpdump Tutorial with Examples — 50 Ways to Isolate Traffic](https://danielmiessler.com/study/tcpdump/)

#### Linux ftrace TCP Retransmit Tracing

- We can use [tcpretrans](https://github.com/brendangregg/perf-tools/blob/master/net/tcpretrans) part of [perf-tools](https://github.com/brendangregg/perf-tools) that is maintained by Brendan Gregg

References:

- [Brendan Gregg's Blog - Linux ftrace TCP Retransmit Tracing](https://www.brendangregg.com/blog/2014-09-06/linux-ftrace-tcp-retransmit-tracing.html)

#### Monitor TCP Accept Queue Length and Overflows

Check for overflows

```
[centos ~]$ nstat -az | grep -i listen
TcpExtListenOverflows           3518352            0.0
TcpExtListenDrops               3518388            0.0
TcpExtTCPFastOpenListenOverflow 0  0.0

[centos ~]$ netstat -s | grep -i LISTEN
    3518352 times the listen queue of a socket overflowed
    3518388 SYNs to LISTEN sockets dropped
```

Monitor queue sizes:

```
$ ss -n state syn-recv sport = :80 | wc -l
119
```

References:

- [How can I monitor the length of the accept queue?](https://unix.stackexchange.com/questions/328746/how-can-i-monitor-the-length-of-the-accept-queue)
- [Investigating Linux Network Issues with netstat and nstat](https://perfchron.com/2015/12/26/investigating-linux-network-issues-with-netstat-and-nstat/)
- [How TCP backlog works in Linux](http://veithen.io/2014/01/01/how-tcp-backlog-works-in-linux.html)
- [SYN packet handling in the wild](https://blog.cloudflare.com/syn-packet-handling-in-the-wild/)

#### On HTTP

- [HTTP Keep-Alive, Pipelining, Multiplexing and Connection Pooling](https://www.haproxy.com/blog/http-keep-alive-pipelining-multiplexing-and-connection-pooling/)

#### References

- [How to Get to the Bottom of Network Timeout Issues](https://www.alibabacloud.com/blog/595248)
