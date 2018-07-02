---
aliases:
  - /mysqlexception-unable-to-connect-to-any-of-the-specified-mysql-hosts
  - /2009/01/02/mysqlexception-unable-to-connect-to-any-of-the-specified-mysql-hosts.html
title: "MySqlException: Unable to connect to any of the specified MySQL hosts"
date: 2009-01-02
---

Usually in batch type of MySql client code you might have seen this exception before:
MySqlException: Unable to connect to any of the specified MySQL hosts
(or similar, this one is specific to MySql .Net client)

<!--more-->

Of course it might be what it says, but actually message can be a little misleading if it happens in
some batch jobs (with huge number of connects and disconnects) but nowhere else.
After a little digging and debugging in the MySql .Net client library code it became obvious 
that it was a low level TCP socket issue. Because of the high number of connects and disconnects,
OS (WinXP/2003 in this case) network layer was not finding enough user ports to assign to the server
(by design TCP keeps the connection in a TIME_WAIT state before disposing the connection).

Little googling quickly relieved a few solutions:
First one is simple. If you can alter the code, the obvious solution is to use pooling or keep the
connection around while querying the database. This would be much more efficient as well.
Other suggestion is to mingle with the OS variables. On Microsoft Windows XP / Server 2003 these
are registry variables MaxUserPort and TcpTimedWaitDelay under
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters.
On Vista and server 2008 MaxUserPort have different meaning and can be set using netsh utility.


Here are my registry settings on XP/2003:

    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\MaxUserPort 0xFFFF (DWORD)
    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\MaxUserPort\TcpTimedWaitDelay 60 (DWORD)

You need to create them. By default they don't exists.

On Vista/2008 you can use netsh to change it to something like:

    netsh int ipv4 set dynamicport tcp start=10000 num=50000

Here is example code to reproduce the issue:(In Ruby, for fun)

{{< highlight ruby >}}
require "mysql"
c = 1;
20000.times do
  c += 1
  puts(c)
  dbh = Mysql.real_connect("localhost", "root", "", "test")
  dbh.query("select 1")
  dbh.close()
end
{{< /highlight >}}

You should see exceptions in about 3900 iteration in XP/2003 and 16000 in Vista/2008,
 if they have the default values for their TCP parameters.
 
 I have not tested this with any other TCP servers or OSes, but I would expect the issue and the solutions to be similar.
 
 Happy hacking..
 
 
 References:
 
 <a href="http://support.microsoft.com/kb/328476">Description of TCP/IP settings that you may have to adjust when
 SQL Server connection pooling is disabled</a>
 
 <a href="http://technet.microsoft.com/en-us/commerceserver/bb608743.aspx">Commerce Server 2002 Best Practices</a>
 
 <a href="http://technet.microsoft.com/en-us/library/aa995661.aspx">Microsoft Exchange Server Analyzer: MaxUserPort value is too low</a>

 <a href="http://support.microsoft.com/kb/929851">The default dynamic port range for TCP/IP has changed
 in Windows Vista and in Windows Server 2008</a>

 <a href="http://blogs.technet.com/swi/archive/2008/07/08/ms08-037-more-entropy-in-the-dns-resolver.aspx">MS08-037 : More entropy for the DNS resolver</a>
