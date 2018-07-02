---
title: "Finally Switched to Hugo"
date: 2018-06-25
---

I have been looking into using [Hugo](https://gohugo.io) for a long time.
Finally managed to make the transition. Also I decided not to use a
ready made theme and created my simple layout and CSS using
[Skeleton](http://getskeleton.com) as my CSS framework.

<!--more-->

[Syntax highlighting](https://gohugo.io/content-management/syntax-highlighting/) is an other area I am keen to use, since I usually have at least one or two code samples in my posts.

Here are two sample HTML and C# blocks.

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

All the source is available in [my GitHub Pages repo](https://github.com/mtmk/mtmk.github.io/tree/master/_hugo_site). Feel free to compy and experiment with it.

Finally this is how you would generate and publish your site (also not to myself)

{{< highlight bash >}}
  git clone mtmk.github.io
  cd mtmk.github.io
  cd _hugo_site
  hugo -d ..
  cd ..
  git add .
  git commit -m 'New content'
  git push
{{< /highlight >}}

Enjoy!
