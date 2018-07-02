---
aliases:
    - /aspnet-cgi-handler
    - /2009/06/06/aspnet-cgi-handler.html
title: Asp.Net CGI handler
date: 2009-06-06
---
I created <a href="https://github.com/mtmk/aspnet-cgi-handler">Asp.Net CGI Handler</a>
to enable some of the old Perl scripts to run under Asp.Net.

<!--more-->

CGI Handler is a Asp.Net HttpHandler implementation which can run programs as
<a href="http://www.w3.org/CGI/">CGIs</a>.

Code is meant to be an example; there are no binaries or releases.
If you want to use it, you will need to copy the CgiHttpHandler.cs
into your project and modify a few things to get it running.

Enjoy!

