---
layout: post
title:  "Port Scanning with Bash"
date:   2016-02-29 00:00:01
categories: jekyll update
---

Imagine that you are performing an assessment and you find a system that you need to pivot to and would like to perform some light reconnaissance on your next destination. The system you are using does not have netcat or nmap installed. What do you do? One possibility is that you can dial up `/dev/tcp` with a for loop to iterate through a port range of your choosing.

`time for i in {0..1024}; do echo 'derp' > /dev/tcp/deadbadger.nullkey.net/$i && echo "$i is open"; done 2>/dev/null`

We start with the `time` command which will summarize system resource usage for the latter part of our one-liner. Next, we begin our `for` loop with the variable `i` and use brace expansion to define our range of ports that we would like the variable `i` to represent. Next, we need to tell Bash what to do for each iteration of the loop. In this example, we are telling it to `echo` the text “derp” to `/dev/tcp` for the host deadbadger.nullkey.net. Immediately following you will notice that we have placed the variable `$i` for the port numbers we told Bash we wanted it to iterate through earlier in the command. With chain re-direction of the text we are echoing to `/dev/tcp` to another `echo` string with the text “`$i` is open” with the AND Operator (`&&`) so that each time `/dev/tcp` receives a response from the host deadbadger.nullkey.net on current port iteration, it will print to STDOUT the text “22 is open”, for example. We tell the for loop to stop each iteration and begin the next one with `;` done and redirect STDERR to a file with `2>/dev/null` (in this case we are simply discarding it by sending it to the null device).

```
derp@zerocool:~# time for i in {0..1024}; do echo 'derp' > /dev/tcp/deadbadger.nullkey.net/$i && echo "$i is open"; done 2>/dev/null
22 is open
111 is open

real    0m30.084s
user    0m0.220s
sys    0m0.416s
```

Per RFC 793, when using TCP as your transport for this type of reconnaissance, the scanning host will know that the target host is listening on a particular port if it receives a SYN-ACK response to the initial SYN communication on whatever port is being examined. The host being scanned will send a RST-ACK if it receives a SYN on a non-listening port.

This kind of port mapping can also be accomplished with UDP traffic by dialing up `/dev/udp` where the scanning host will deduce that a port is listening if it does not receive an ICMP port unreachable message from the target host. Mapping open UDP ports relies on the assumption that there is nothing filtering ICMP traffic in between the scanning and the target host. If ICMP traffic is being filtered, it may appear to the scanner that all ports are open (remember UDP as a transport provides no confirmation of delivery so IP error detection is all we have unless something at the application layer responds as is the case with DNS).

You do not need to be root to perform this type of scanning since normal users can create sockets and establish full connections to any port.

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
