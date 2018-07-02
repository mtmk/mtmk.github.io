---
aliases:
    - /2010/09/03/csharp-channels.html
    - /cs-channels
title: C# channels
date: 2010-09-03
---
<a href="http://golang.org/">Go</a> is one of the new languages I came across and found interesting from a few of different angles.

<!--more-->

First of all it is compiled and statically typed somewhat comparable to Java and C#. Second interesting aspect is it's syntax; Although it is basically a C-like syntax there is one thing would catch your eyes first is the variable declarations - they are wrong way round! Well, "that's weird" was my first reaction but when you kind of get used to it, variable name first, type last notation starts to makes (arguable) more sense. After all variable name is more important than it's type when it comes to thinking about your programs logic. Last but not least, the approach to concurrency is very sensible; it has beautiful concurrent constructs like [channels](http://golang.org/doc/effective_go.html#channels) and Go Routines. This concurrency aspect interests me most and decided to do a little experiment in C#, implementing a very simple 'Channel'. (Hence this post and the cheese title)

As far as I understand, the basic idea is quite simple; while achieving concurrency instead of sharing state, pass the state through multiple queues avoiding hard-to-get-right synchronisation mechanisms. Possibly, like what you might have guessed (if you are coming from a computer science background, or read a few articles about how increasingly parallel we need to start making our programs, because the [Moore's Law](http://en.wikipedia.org/wiki/Moore's_law) is not going to last forever and so on) the idea is not new at all. Actually in computer technology terms it's ancient history. I have come across a [Communications of the ACM article](http://cacm.acm.org/magazines/2010/6/92479-the-resurgence-of-parallelism/fulltext) which had a section about Determinate Computation explaining the idea of using blocking tasks communicating through FIFO queues rather than shared memory.

I have been working with C# quite some time now and since I have used blocking queues (which I find very similar to Go Channels) for similar concurrency purposes before. It was a natural flow of thinking that I needed to implement some generic functionality and make it available to my applications in .Net. Although I must stress that; this is just an experiment and it is far from being a complete implementation. There are other crucial features missing to make it really useful and it is not test much and it is not used in production at all.

The [interface](https://github.com/mtmk/cschan/blob/master/cschan/IChannel.cs) is very simple; there are only to methods, Put(item) and Get() : item. And there are two very simple behaviours; 'put' blocks when the capacity is full, 'get' blocks when there is nothing in the channel. For example in client code you can create your threads and share IChannel instances where you can use a set of threads as [producers and consumers](http://en.wikipedia.org/wiki/Producer-consumer_problem) or you can use them for simple notifications.


{{< highlight csharp >}}
var c = new Channel<int>(5);
var t1 = new Thread(() =>
    {
        for (int i = 0; i < 10; i++)
            c.Put(i);
    });

var t2 = new Thread(() =>
    {
        for (int i = 0; i < 10; i++)
            Console.WriteLine(c.Get());
    });

t1.Start();
t2.Start();
t1.Join();
t2.Join();
{{< /highlight >}}

In the above simple example you would always have sequential numbers printed in ascending order e.g. 0, 1, 2, 3, ... This might not sound very interesting but notice I did't have to use any synchronisation mechanisms to achieve a deterministic output.  Same example code can be used in several threads with many channels working together.

Having a quick look at the implementation actually reveals all the tricky synchronisation is handled so I don't have to in my client code. I am basically using a Queue to store items and two Semaphores to block Gets and Puts as the queue runs out and capacity is reached respectively. Here is the constructor:

{{< highlight csharp >}}
private readonly Queue<T> _q = new Queue<T>();
private readonly Semaphore _sPut;
private readonly Semaphore _sGet;

public Channel(int capacity)
{
    _sPut = new Semaphore(capacity, capacity);
    _sGet = new Semaphore(0, capacity);
}
{{< /highlight >}}

There is a bit more to it in the [code repository](https://github.com/mtmk/cschan/blob/master/cschan/Channel.cs) (to handle clean exits and so on) but above snippet shows the guts of it.  Similarly Put and Get methods are fairly simple without the details:

{{< highlight csharp >}}
public void Put(T item)
{
    lock (_guard)
    {
        _sGet.Release();
        _q.Enqueue(item);
    }
}

public T Get()
{
    lock (_guard)
    {
        _sPut.Release();
        return _q.Dequeue();
    }
}
{{< /highlight >}}

As you can see this isn't much different from a [standard blocking queue](http://blogs.msdn.com/b/toub/archive/2006/04/12/blocking-queues.aspx) or [producer/consumer](http://en.wikipedia.org/wiki/Producer-consumer_problem) implementation (only difference is I am blocking on Puts as well -when the queue is full). The real challenge with this implementation would be implementing the Channel class (or Get and Put return types) in such a way so it behaves like a WaitHandle and can be used with multiple channels blocking on Gets or Puts to support more useful scenarios. I suspect this would require some low level unmanaged code. In the [example implementation](https://github.com/mtmk/cschan) you can see I took the initial steps towards this direction by returning a result object from Get and Put methods.
