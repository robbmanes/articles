---
title: Running CoreDNS as a DNS Server in a Container
published: true
description: How to manually configure CoreDNS to serve your own DNS zones and 
tags: dns, coredns, container, docker
date_of_publish: 02-09-2019
---
#  Running CoreDNS as a DNS Server in a Container 
This article is published at:
- [dev.to](https://dev.to/robbmanes/running-coredns-as-a-dns-server-in-a-container-1d0)

If you've ever needed to or wanted to set up your own DNS server, then this is for you.  I recently found myself in possession of a Raspberry Pi, and instead of relying on my home router for DHCP and DNS, I decided to serve both from containers on the Pi, so I could resolve all of my hosts with their respective names when I VPN back to my network.  I intended to have all other requests forwarded to another server that weren't in my local network, but still service local systems with my customized hostnames.

Lately I find myself working more and more with containers and OpenShift, a Kubernetes distribution from Red Hat (disclaimer, I work for Red Hat), and in upstream Kubernetes one of the DNS servers provided is [CoreDNS](https://coredns.io/).  I've been playing around with it for a while and thought I'd make this tutorial concerning how to launch it yourself, using the [CoreDNS provided container image](https://hub.docker.com/r/coredns/coredns) on Docker Hub.

I'd also like to note we're using a very, very small fraction of CoreDNS functionality here.  One of the outstanding things about CoreDNS is its customizability with plugins, and its direct integration with Kubernetes via said plugins makes it extremely powerful indeed.  Nonetheless, if DNS servers are new to you, or `bind` or `unbound` scares you a bit, maybe give CoreDNS for your personal needs.

Without further ado, here's how I got it working.

First, pull the container image down locally from Docker Hub.  If you're using Docker, you can do so like this:
```
# docker pull coredns/coredns
```

Afterwards, we'll need to configure a `Corefile`, which serves as the CoreDNS daemon's configuration file; there are many options that can be passed in here (which can be seen in the CoreDNS manual), but I will go through the example I used.

```
# cat ~/containers/coredns$ cat Corefile 
.:53 {
	forward . 8.8.8.8 9.9.9.9
	log
	errors
}

example.com:53 {
	file /root/example.db
	log
	errors
}
```

Let's go through the options of the `Corefile` one-by-one.  It is important to note that each bracketed section denotes a DNS "zone", which sets the behavior of CoreDNS based on what is being resolved.

First, note the initial bracketed section.  It begins with a `.:53`, indicating that this zone is a global (with "." indicating all traffic), and it is listening on port 53 (udp by default).  The parameters we set in here will apply to all incoming DNS queries that do not specify a specific zone, like a query to resolve `github.com`.  We see on the next line, that we forward such requests to a secondary DNS server for resolution; in this case, all requests to this zone will be simply forwarded to Google's DNS servers at `8.8.8.8` and `9.9.9.9`.

Second, we have another zone which is specified for `example.com`, also listening on UDP port 53.  Any queries for hosts belonging in this zone will refer to a file database (similar to how bind does) to do a lookup there; more on that momentarily.  As an example, a query to "server.example.com" will bypass the global zone of "." and fall into the zone which is servicing "example.com", and using the `file` directive the database file will be referenced to find the proper record.

That's all there is to this `Corefile`, for a simple forwarding DNS server which also serves local clients with hostnames.  Now we have to make that DNS database file we referenced, `example.db`, and fill it with our hosts.

Although this isn't a DNS primer, I will go over how this file works.  There are two main records at play here, and I'll discuss a third; they are SOA, A, and CNAME DNS records which will make up our DNS configuration.

Initially, we *must* configure an `SOA` record, or a "Start of Authority" record.  This is the initial record used by this DNS server in this zone to declare its authority to the client which is making a query, and we must begin the file with it.  Here is an example SOA record which can be used in this file:
```
example.com.		IN	SOA	dns.example.com. robbmanes.example.com. 2015082541 7200 3600 1209600 3600
```

To go over each section individually:

- `example.com.` refers to the zone in which this DNS server is responsible for.
- `SOA` refers to the type of record; in this case, a "Start of Authority"
- `dns.example.com` refers to the name of this DNS server
- `robbmanes.example.com` refers to the email of the administrator of this DNS server.  Note that the `@` sign is simply noted with a period; this is not a mistake, but how it is formatted.
- `2015082541` refers to the serial number.  This can be whatever you like, so long as it is a serial number that is not reused in this configuration or otherwise has invalid characters.  There are usually rules to follow concerning how to set this, notably by setting a valid date concerning the last modifications, like `2019020822` for February 08, 2019, at 22:00 hours.
- `7200` refers to the Refresh rate in seconds; after this amount of time, the client should re-retrieve an SOA.
- `3600` is the Retry rate in seconds; after this, any Refresh that failed should be retried.
- `1209600` refers to the amount of time in seconds that passes before a client should no longer consider this zone as "authoritative".  The information in this SOA expires ater this time.
- `3600` refers to the Time-To-Live in seconds, which is the default for all records in the zone.

Once we've written our `SOA` to our liking, we can add additional records for each of our hosts we wish to resolve.  I assign my IP addresses with static DHCP leases for certain MAC addresses, so to do so I first added the DNS server, so it can resolve to itself:
```
dns.example.com.	IN	A	192.168.1.2
```

An `A` record indicates a name, in this case `dns.example.com`, which can be canonically mapped directly to an IP address, `192.168.1.2`.  If I add another `A` record:
```
host.example.com.	IN	A	192.168.1.10
```

I can then assign a `CNAME` record to it, which will serve as an "alias" of sorts, directing traffic back to `host.example.com`:
```
server.example.com.	IN	CNAME	host.example.com.
```

You can add as many entries here as you like, or look up different types of records to suit your needs.  I ended up with something basic, like so:

```
example.com.		IN	SOA	dns.example.com. robbmanes.example.com. 2015082541 7200 3600 1209600 3600
gateway.example.com.	IN	A	192.168.1.1
dns.example.com.	IN	A	192.168.1.2
host.example.com.	IN	A	192.168.1.3
server.example.com	IN	CNAME	host.example.com
```

Afterwards, when we're done with our DNS zone file and our `Corefile`, we can stick them in the same directory and prepare to export them to a newly-running coredns container.  I stuck both of these files in a directory of `~/containers/coredns/`:
```
$ pwd
/home/robb/containers/coredns

$ ls
Corefile  example.db
```

To run the container, the `coredns` binary looks in the immediate directory its in for any file named `Corefile`, and uses it as configuration.  Unfortunately, in the `coredns/coredns` image we pulled from Docker Hub, it is located in the root directory of `/`, which can't be mounted as a volume.  We'll need to manually pass our `Corefile` and ensure that the `file` directive in our zone of `example.com:53` is a direct path in the container to the DNS zone database file.  To do this, I mapped them to `/root` in the container and passed the `-conf` option which allows a user to specify the path to a Corefile; this is the command I used to launch my CoreDNS container:
```
# docker run -d --name coredns --restart=always --volume=/home/robb/containers/coredns/:/root/ -p 53:53/udp coredns/coredns -conf /root/Corefile
```

Afterwards, I made sure my container was running without issues by checking the logs and `docker ps -a`:
```
# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                        NAMES
8a6c9a5c0538        coredns/coredns     "/coredns -conf /roo…"   About an hour ago   Up About an hour    53/tcp, 0.0.0.0:53->53/udp   coredns

# docker logs coredns
.:53
example.com.:53
2019-02-09T04:50:24.060Z [INFO] CoreDNS-1.3.1
2019-02-09T04:50:24.061Z [INFO] linux/arm, go1.11.4, 6b56a9c
CoreDNS-1.3.1
linux/arm, go1.11.4, 6b56a9c
```

We can then query our server with `dig` from a client in the same subnet to make sure it's working as intended.  My DNS container is running on a host with an IP of `192.168.1.2`:
```
$ dig @192.168.1.2 host.example.com

; <<>> DiG 9.11.3-1ubuntu1.3-Ubuntu <<>> @192.168.1.2 host.example.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 30400
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 9faccdae88cb6576 (echoed)
;; QUESTION SECTION:
;host.example.com		IN	A

;; ANSWER SECTION:
host.example.com	0	IN	A	192.168.1.3

;; Query time: 3 msec
;; SERVER: 192.168.1.2#53(192.168.1.2)
;; WHEN: Fri Feb 08 23:01:22 MST 2019
;; MSG SIZE  rcvd: 93
```

And just like that, we have an easy-to-configure and maintain CoreDNS configuration running!  Thusfar, I have been using `docker restart` to reload the container and re-read the `Corefile` and DNS zone file, but you should absolutely be aware of the [reload](https://coredns.io/plugins/reload/) plugin to CoreDNS that removes the need to restart the container.

That's a very basic introduction to CoreDNS, and I hope you get some good usage out of this great DNS daemon.