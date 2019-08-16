---
title: "Confusing error: could not bind IPv4 socket: Cannot assign requested address"
categories:
  - blog
tags:
  - Linux
  - Troubleshooting
  - Tech
---

## Postgres service failing with error: "could not bind IPv4 socket: Cannot assign requested address"

Detailed Error:

{% highlight bash %}
-- Unit postgresql.service has begun starting up.
Aug 14 13:21:25 localmachine pg_ctl[31257]: 2019-08-14 13:21:25 IST LOG:  could not bind IPv4 socket: Cannot assign requested address
Aug 14 13:21:25 localmachine pg_ctl[31257]: 2019-08-14 13:21:25 IST HINT:  Is another postmaster already running on port 5432? If not, wait a few seconds and retry.
Aug 14 13:21:25 localmachine pg_ctl[31257]: 2019-08-14 13:21:25 IST WARNING:  could not create listen socket for "localhost"
Aug 14 13:21:25 localmachine pg_ctl[31257]: 2019-08-14 13:21:25 IST FATAL:  could not create any TCP/IP sockets
Aug 14 13:21:26 localmachine pg_ctl[31257]: pg_ctl: could not start server
Aug 14 13:21:26 localmachine pg_ctl[31257]: Examine the log output.
Aug 14 13:21:26 localmachine systemd[1]: postgresql.service: control process exited, code=exited status=1
Aug 14 13:21:26 localmachine systemd[1]: Failed to start PostgreSQL database server.
-- Subject: Unit postgresql.service has failed
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
--
-- Unit postgresql.service has failed.
--
-- The result is failed.
Aug 14 13:21:26 localmachine systemd[1]: Unit postgresql.service entered failed state.
Aug 14 13:21:26 localmachine systemd[1]: postgresql.service failed.
{% endhighlight bash %}

There are multiple scenarios and places to check in this case. Got lost once because of this. All all troubleshooting steps where pointing towards checking on `/etc/hosts` file and whether any other services are running on port used by `PostgreSQL`.

Checked and confirmed no other services are running on localhost port `5432` ( 127.0.0.1:5432 ) :

{% highlight bash %}
netstat -ntualp|grep "127.0.0.1:5432"
{% endhighlight bash %}

A usual default `hosts` file should look like one below:

{% highlight bash %}
[root@localmachine ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
{% endhighlight bash %}

To confirm `localhost` is able to resolve, test using `ping`:

{% highlight bash %}
[root@localmachine ~]# ping -c2 localhost
PING localhost (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.125 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.071 ms

--- localhost ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.071/0.098/0.125/0.027 ms
{% endhighlight bash %}

This `systemctl restart postgresql` was failing with same error !!

I tried changing `listen = 127.0.0.1` instead of `localhost`. And it was working and restarted `PostgreSQL` service without any issue! At this point it was confusing. Couldn't find any relevant logging anywhere.

Changing `localhost` to `IP` was not a solution for me. It's a workaround.

Checking further could see `PostgreSQL` was connecting to `localhost.xx.yy.zz.com`. It means it was getting resolving `localhost` from somewhere else.

What step a SysAdmin can check on the next step :) tried with `nslookup`:

{% highlight bash %}
nslookup localhost
Server:         10.24.237.5
Address:        10.24.237.5#53

Name:   localhost.xx.yy.com      <<< some wrong FQDN
Address: 10.161.201.64           <<< some wrong IP
{% endhighlight bash %}

`localhost` was resolving as `localhost.xx.yy.com` which is a `DNS` thing. Will explain this in a later post ;)

So, at this point my conclusion was, a `PostgreSQL` local servie should be respecting `/etc/hosts` entries and should be able to find `127.0.0.1` as the answer for `localhost`.

What if `PostgreSQL` don't have access to `hosts` file? !!! Further googling helped me to check `/etc/nsswitch.conf` which controls these check-list.

{% highlight bash %}
[root@localmachine ~]# grep hosts: /etc/nsswitch.conf
#hosts:     db files nisplus nis dns
hosts:      files dns myhostname            <<< files:- this entry should be there for using /etc/hosts file.
{% endhighlight bash %}

It was there, then what?

`PostgreSQL` was ignoring or not able to access `/etc/hosts` file due to `WRONG PERMISSIONS`! and  But NOOO related log entries found :(

{% highlight bash %}
[root@satellite ~]# ls -l /etc/hosts
-rw-------. 1 root root 240 Aug 14 08:04 /etc/hosts     <<<  600 permission. Only owner/root able to access.
{% endhighlight bash %}

Yes, `Postgres` or any other service was not able to access `hosts` file.

Changed permissions using:

{% highlight bash %}
[root@localmachine ~]# chmod 644 /etc/hosts
You have new mail in /var/spool/mail/root

[root@localmachine ~]# ls -al /etc/hosts
-rw-r--r--. 1 root root 240 Aug 14 08:04 /etc/hosts
[root@localmachine ~]#
{% endhighlight bash %}

Restarted the service: `systemctl restart postgresql`.

Yes !.. Made it at last. A small, simple permission issue made all these ... :)
