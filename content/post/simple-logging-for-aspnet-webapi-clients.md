---
aliases:
  - /c#/asp.net/webapi/logging/log4net/2015/02/28/simple-logging-for-aspnet-webapi-clients.html
title:  "Simple logging for ASP.NET Web API clients"
date:   2015-02-28
---
This is a quick and dirty example of how you can enable a simple form of logging for clients
using ASP.NET Web API. I used log4net and an extremely simple controller to achieve this.

<!--more-->

Controller has only a couple of tricks to get the body of the POST as it is regardless of the content type. The rest is pretty straight forward.

{% highlight c# %}
public class LogController : ApiController
{

    public async Task<string> Post()
    {
        string message = await Request.Content.ReadAsStringAsync();

        string logger = GetHeaderOrDefault("X-Logger", "main");
        string logLevel = GetHeaderOrDefault("X-LogLevel", "debug");

        ILog log = LogManager.GetLogger("client." + logger);

        if (logLevel == "error") log.Error(message);
        else if (logLevel == "warn") log.Warn(message);
        else if (logLevel == "info") log.Info(message);
        else log.Debug(message);

        return "OK " + DateTime.Now;
    }
}
{% endhighlight %} 

Another simple trick is with the log4net configuration. This is to keep the log file separate from the rest of the logging in the application:

{% highlight xml %}
<?xml version="1.0" encoding="utf-8" ?>
<log4net>

  <appender name="RollingFileClient" type="log4net.Appender.RollingFileAppender">
    <file value="logs\client.log" />
    <appendToFile value="true" />
    <rollingStyle value="Size" />
    <maxSizeRollBackups value="10" />
    <maximumFileSize value="10MB"/>
    <staticLogFileName value="true" />
    <lockingModel type="log4net.Appender.FileAppender+MinimalLock" />
    <layout type="log4net.Layout.PatternLayout">
      <conversionPattern value="%d %-5p %c - %m%n" />
    </layout>
  </appender>
  
  <logger name="client">
    <level value="DEBUG" />
    <appender-ref ref="RollingFileClient"/>
  </logger>
    
</log4net>
{% endhighlight %}

You might find the rolling file appender interesting too which makes sure the log files won't go out of hand in the long run.

Full working example is available [on GitHub under mtmk](https://github.com/mtmk/simple-aspnet-webapi-logger).

Enjoy!
