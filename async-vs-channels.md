# C# async vs GO routines + channels

**Scenario** : You need a function that can do work, e.g. download files in the background and not block the main thread. The most common use for this is to not block a app's UI, or to do work in parallel. For example; create an ansychronous function that can download files in the background.

In C# this would be achieved by creating an async function. In GO you use goroutines and channels.

<table width="100%">
<tr>
<th><img width="441" height="1">C#</th>
<th><img width="441" height="1">GO</th>
</tr>
<tr>
<td>

```csharp
static Task<int> downloadFile(string name)
{
    var t = Task<int>.Run(() => {
    var r = new Random();
    Thread.Sleep(r.Next(3000));
    Console.WriteLine($"downloaded {name}");
    return r.Next(2000);
    });
    return t;
}
```

</td>
<td>

```go
func downloadFile(name string) <-chan int32 {
	r := make(chan int32)
	go func(name string) {
		defer close(r)
		time.Sleep(time.Second * 3)
		fmt.Println("downloaded " + name)
		r <- rand.Int31n(100)
	}(name)
	return r
}
```

</td>
</tr>
</table>

Some introduction paragraph about pattern Blah goes here.

<table width=100%>
<tr>
<th><img width="441" height="1">C#</th>
<th><img width="441" height="1">GO</th>
</tr>
<tr valign="top">
<td valign="top">
<pre>

```csharp
public static async Task AsyncVsChannels()
{
    Console.WriteLine("downloading 3 files");
    (var a, var b, var c) = (
        downloadFile("a.txt"),
        downloadFile("b.txt"),
        downloadFile("c.txt")
    );

    Console.WriteLine("Not blocked.");
    Thread.Sleep(500);
    Console.WriteLine("Wait for all;");

    // wait for all dowloads
    Task.WaitAll(a, b, c);

    // write results
    Console.WriteLine("{0:d},{1:d},{2:d}",
        await a,
        await b,
        await c
    );
}
```

</pre>
</td>
<td valign=top>

```go
func AsyncVsChannels()
{
	fmt.Println("downloading 3 files")
	aCh, bCh, cCh :=
		downloadFile("a.txt"),
		downloadFile("b.txt"),
		downloadFile("c.txt")

	fmt.Println("Not blocked.")
	time.Sleep(time.Second * 1)
	fmt.Println("Wait for all;")

    // wait for all downloads
    // write results
	fmt.Println(<- aCh, <-bCh, <- cCh)
}
```

</td>
</tr>
</table>

Running the code above produces the following output

```go
downloading 3 files
Not blocked.
Wait for all;
downloaded c.txt
downloaded b.txt
downloaded a.txt
1567,359,14
```

## Take aways?

-   C# makes a complicated process look too simple.
-   GO forces you to learn a lot about concurrency even with this very common and what "looks" trivial example, and this will stay with you for life.

## resources

-   [use Go Channels as Promises and Async/Await](https://levelup.gitconnected.com/use-go-channels-as-promises-and-async-await-ee62d93078ec)
