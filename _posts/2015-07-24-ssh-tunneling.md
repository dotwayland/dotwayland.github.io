---
layout: post
title:  "Setting up an SSH Tunnel for Safer Browsing"
date:   2015-07-24 00:00:01
categories: jekyll update
---

There may be times where you need to browse the web from an untrusted network. Examples include anything from a public library, coffee shop, a hotel, or even your place of employment. In any of these scenarios your communications could be monitored by a malicious user or curious network administrator.

This guide will show you how to establish a secure connection for browsing the web through a tunnel between your computer and your an SSH server that you control. After reading this, you will be able to set up a tunnel between your computer and your SSH server. All of your web traffic will be encrypted and forwarded from your SSH server on to its final destination.

This works by creating a SOCKS proxy listener on your computer using SSH and configuring your browser to send all web traffic through the SOCKS connection so that it can be sent by your SSH server to a given web destination on your behalf. This guide will also demonstrate how to prevent your browser from exposing information about your web browsing behavior via DNS leaks.

This guide assumes that your threat model includes the local network that your computer is currently connected, and not your trusted SSH server, the network that it is connecting from or the website(s) that you intend to visit.

What you will need

 * An SSH client
 * An SSH server
 * A web browser

Setting up the SSH server to allow port forwarding

Look for the `AllowTcpForwarding` no directive in your SSH server’s `/etc/ssh/sshd_config` file, and change it to `AllowTcpForwarding yes` before restarting the service. If the directive isn’t listed at all, simply add it and ensure that it is set to `yes`.

Setting up the SOCKS server

From your computer’s terminal, run the following:

`ssh -D 31337 user@remotehost.tld`

`-D` tells SSH to listen on port `31337`. This works by allocating a socket to listen to port on the local side, optionally bound to the specified `bind_address`. Whenever a connection is made to this port, the connection is forwarded over the secure channel, and the application protocol is then used to determine where to connect to from the remote machine. Only root can forward privileged ports so be sure to choose any port number greater than 1024. You could choose something lower, but to ensure that there are no conflicts its best to choose a higher numbered port.

This can also be accomplished by defining the `DynamicForward` parameter in your client’s `~/.ssh/config` file.

```
Host remotehost.tld
    HostName 13.37.13.37
    User username
    Port 22
    IdentityFile ~/.ssh/id_rsa
    DynamicForward 31337
```

In this scenario, you can simply type `ssh remotehost.tld` and your SSH client will take care of the rest.

Setting Up Your Browser

The last step is to configure your preferred browser to use the SOCKS server you just created. This guide assumes Firefox is being used but this can be accomplished with most other browsers.

To set up the browser:

* In Firefox, go to the Edit menu and select Preferences.

* Go to Advanced and from there to the Network tab.

* In the Connection area click on Settings.

Make sure the `Remote DNS` option is selected. This can also be specified by typing `about:config` in Firefox and setting the `network.proxy.socks_remote_dns` parameter to `true`

This is crucial because although your web traffic will be encrypted and forwarded, your DNS requests will not if you do not tell also send your DNS queries through the SOCKS connection. You can fix that in Firefox, and make it send the DNS traffic to your tunnel as well. If you do not specify the remote DNS option, an observer on the untrusted LAN will not be able to see any communication to or from the sites that you are visiting, but will be able to see your browser requesting the IP addresses for their associated domain names which is also undesirable.

Once you have everything set up, you should now verify that the websites you visit see the IP address of your SSH server, not the IP address allocated to you by the untrusted network. You can verify this by visiting a site such as ipchecking.com comparing the address that it thinks you are connecting from against that of your SSH server.

Please keep in mind that the steps taken above will only secure network communications sent to or received by your web browser. Many applications have support for SOCKS proxying but will need to be configured on a case by case basis. If you find yourself using untrusted networks frequently, or have many applications that you need to secure communications for then the it might be time to step up to a VPN solution that has the capability to encrypt all traffic sent to or received by your computer’s network interface.

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
