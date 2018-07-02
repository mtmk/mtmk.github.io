---
title: "Finally Switched to Hugo"
date: 2018-06-25
---

I have been looking into using Hugo for a long time. Finally managed to make
the transition. Also I decided not to use a ready made theme and created my simple layout.

<!--more-->

http://getskeleton.com

https://gohugo.io

{{< highlight html >}}
<section id="main">
  <div>
    <h1 id="title">{{ .Title }}</h1>
    {{ range .Data.Pages }}
      {{ .Render "summary"}}
    {{ end }}
  </div>
</section>
{{< /highlight >}}

{{< highlight csharp >}}
public class Program
{
  public static int Main(string args)
  {
    System.Console.WriteLine("Hello world!");
  }
}
{{< /highlight >}}