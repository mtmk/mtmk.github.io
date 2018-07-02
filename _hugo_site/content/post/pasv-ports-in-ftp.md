---
aliases:
    - /ftp/pasv/binary/2015/03/25/pasv-ports-in-ftp.html
title:  "PASV ports in FTP"
date:   2015-03-25
---

I was looking to simulate an embedded device for testing and looking into FTP server implementations a simple bitwise operations question popped up: the way PASV command is standardized requires TCP port numbers to be transferred in two octets represented as integers.

<!--more-->

This is how the server responds to a PASV command:

{{< highlight bash >}}
227 Entering Passive Mode (h1,h2,h3,h4,p1,p2).
{{< /highlight >}}

Which indicates and IP address and a port number. To find the IP address is easy. it's only the first four octets. Port number requires a little math: p1 * 256 + p2. That's all fine and documented quite a bit. But in a server implementation one needs to figure out these octets from a short integer as the port number:

{{< highlight csharp >}}
p1 = port >> 8
p2 = port & 255
{{< /highlight >}}

I just like bitwise operations so much, I though this little experiment deserved a note on the side.

Enjoy
