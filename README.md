# C# with CSharpFritz

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/csharpfritz/csharp_with_csharpfritz/main)

Show notes, slides, and samples from the CSharp with CSharpFritz show.  Jupyter notebooks are available in the [notebooks](notebooks) folder. Notebooks are built using [.NET Interactive](https://github.com/dotnet/interactive). 

Recordings from the stream are available in the [CSharp with CSharpFritz playlist](https://www.youtube.com/playlist?list=PLdo4fOcmZ0oXv32dOd36UydQYLejKR61R) on the YouTube .NET channel. 

## Run the notebooks locally

If you would like to run the notebooks locally, without installing Python and .NET interactive, you can clone this repository and then run docker-compose:
```shell
git clone git@github.com:csharpfritz/csharp_with_csharpfritz.git
cd csharp_with_csharpfritz
docker-compose up
```

The notebooks run by default on network port 8888. 

## Community

This project has adopted the code of conduct defined by the [Contributor Covenant](http://contributor-covenant.org/) to clarify expected behavior in our community. For more information, see the [.NET Foundation Code of Conduct](http://www.dotnetfoundation.org/code-of-conduct).

## License

All content for this video series made available in this repository is covered by the [MIT License](https://github.com/csharpfritz/csharp_with_csharpfritz/blob/main/LICENSE). 



# Async, Await, and Multithreading

As part of the .NET ecosystem, the C# language has access to all of the features of the .NET frameworks and runtime.  This allows us to have access to the threading and memory management features of the .NET runtime.  This is a great feature for writing asynchronous code.  In C#, we can use the **async** and **await** keywords to write asynchronous code.  Combined with the Task Parallel Library, this allows us to write code that is multi-threaded and supports multiple CPU cores.

When a method is called with the async keyword, the method will execute asynchronously and release the current thread while processing takes place.  When the asynchronous method returns, it _typically_ resumes execution on the thread that called the method.  More on that later...  This allows us to write code that is multi-threaded and supports multiple CPU cores.

## Key features of async and await:

- Async code can be used for both I/O bound and CPU bound code, but differently for each scenario
- Async code uses `Task<T>` and `Task` return types to model the work being completed in the background
- The `async` keyword turns a method into an asynchronous method, which allows you to use the `await` keyword in its body
- When the `await` keyword is used, it suspends the calling method and yields control back to its caller until the awaited task completes
- `await` can only be used inside an async method

## CPU bound vs I/O bound

1.  Will your code be "waiting" something from disk or across a network?
  
	 If yes, then your work is I/O bound

2.  Will your code be performing an expensive computation?
	
	 If yes, then your work is CPU bound

## I/O bound Example:  Get data from the network

Let's fetch the .NET homepage from the network using the `HttpClient` class.	We'll use the `async` keyword to make this code asynchronous.  The `await` keyword is used to suspend the calling method until the awaited task completes.  While this code fetches data from the .NET website, the normal code continues to run.

#!csharp

using System.Net.Http;

private readonly HttpClient _Client = new HttpClient();

public async Task<string> GetAsync(string url)
{
	var response = await _Client.GetAsync(url);
	return await response.Content.ReadAsStringAsync();
}

display(await GetAsync("https://dot.net"));

#!markdown

### Explanation

The `Task<string>` return type indicates that this method will return a pointer to the state of the background process and it expects to return from the completed process a `string`.  On line 7, it requests the URL and awaits the response, suspending the calling method until the response is received.  On line 8, it uses the `await` keyword again to suspend the calling method while it reads the content of the response from the network.

If we write our code WITHOUT using `await`, it will continue to run while the network request is being processed.  

#!csharp

var dotNet = GetAsync("https://dot.net");
display($"Is it completed: {dotNet.IsCompleted}");
dotNet.Wait();
display($"Is it completed: {dotNet.IsCompleted}");
display(dotNet.Result);

#!markdown

Here, the `Wait` method is called on the `Task` object returned from the `HttpClient` class.  This method will block the current thread until the task completes.  This is a blocking call, which means that the current thread will not continue until the task completes.

#!markdown

## Example: Multiple Asynchronous Operations

We can shift things a little and run multiple fetch requests in parallel.  We won't `await` the methods, but instead stash the `Task` in variables.  We'll the use the `Task.WhenAll` method to wait for all of the tasks to complete.  

#!csharp

var sw = System.Diagnostics.Stopwatch.StartNew();
var bing = GetAsync("https://www.bing.com");
var dotNet = GetAsync("https://dot.net");

await Task.WhenAll(bing, dotNet);

display($"Bing HTML length: {bing.Result.Length}");
display($"DotNet HTML length: {dotNet.Result.Length}");
display(sw.ElapsedMilliseconds);

#!markdown

## The Task object

Let's explore the Task object a bit.  There's a but we can do there.	We can use the `Task.WhenAll` method to wait for all of the tasks to complete.  This method takes an array of `Task` objects and waits for all of them to complete.  The `Task.WhenAll` method returns a `Task<Task[]>` object.  The `Task<Task[]>` object is a pointer to the state of the background process and it expects to return from the completed process an array of `Task` objects.  

We can also use the `Task` object to continue and run other methods when it completes.  We'll use the `ContinueWith` method to run a method when the `Task` completes.  The `ContinueWith` method takes a method that will run when the `Task` completes.  The `ContinueWith` method returns a `Task` object.  The `Task` object is a pointer to the state of the background process and it expects to return from the completed process a `Task` object. 

#!csharp

var dotNet = GetAsync("https://dot.net")
	.ContinueWith(task =>
{
	display(task.IsCompleted);
	display($"DotNet HTML length: {task.Result.Length}");
});

display($"Checking dotNet complete state: {dotNet.IsCompleted}");
await dotNet;

#!markdown

### Forcing an async method to run synchronously

You may need to run an async method synchronously due to being hosted in a method that is not marked as async.  Try calling like this:

#!csharp

var dotNet = GetAsync("https://dot.net")
	.GetAwaiter().GetResult();
display($"DotNet HTML length: {dotNet.Length}");

var bing = GetAsync("https://bing.com");
bing.Wait();
display($"Bing HTML length: {dotNet2.Result.Length}");

#!markdown

## The Breakfast Example

We can think about executing and interacting with tasks similar to how one would prepare a meal.  There is an excellent sample [originally presented in the .NET Docs concerning preparing breakfast](https://docs.microsoft.com/dotnet/csharp/programming-guide/concepts/async/).  Let's recreate that example here and tinker with how it works.

#!csharp

class Juice {}
class Toast {}
class Bacon {}
class Egg{}
class Coffee{}

Juice PourOJ()
{
	Console.WriteLine("Pouring orange juice");
	return new Juice();
}

void ApplyJam(Toast toast) => 
	Console.WriteLine("Putting jam on the toast");

void ApplyButter(Toast toast) => 
	Console.WriteLine("Putting butter on the toast");

Toast ToastBread(int slices)
{
	for (int slice = 0; slice < slices; slice++)
	{
			Console.WriteLine("Putting a slice of bread in the toaster");
	}
	Console.WriteLine("Start toasting...");
	Task.Delay(3000).Wait();
	Console.WriteLine("Remove toast from toaster");

	return new Toast();
}

Bacon FryBacon(int slices)
{
	Console.WriteLine($"putting {slices} slices of bacon in the pan");
	Console.WriteLine("cooking first side of bacon...");
	Task.Delay(3000).Wait();
	for (int slice = 0; slice < slices; slice++)
	{
			Console.WriteLine("flipping a slice of bacon");
	}
	Console.WriteLine("cooking the second side of bacon...");
	Task.Delay(3000).Wait();
	Console.WriteLine("Put bacon on plate");

	return new Bacon();
}

Egg FryEggs(int howMany)
{
	Console.WriteLine("Warming the egg pan...");
	Task.Delay(3000).Wait();
	Console.WriteLine($"cracking {howMany} eggs");
	Console.WriteLine("cooking the eggs ...");
	Task.Delay(3000).Wait();
	Console.WriteLine("Put eggs on plate");

	return new Egg();
}

Coffee PourCoffee()
{
	Console.WriteLine("Pouring coffee");
	return new Coffee();
}

#!markdown

Making breakfast this way is slow...  and with a cafe full of customers, you're not going to have a good morning.

#!csharp

Coffee cup = PourCoffee();
Console.WriteLine("coffee is ready");

Egg eggs = FryEggs(2);
Console.WriteLine("eggs are ready");

Bacon bacon = FryBacon(3);
Console.WriteLine("bacon is ready");

Toast toast = ToastBread(2);
ApplyButter(toast);
ApplyJam(toast);
Console.WriteLine("toast is ready");

Juice oj = PourOJ();
Console.WriteLine("oj is ready");
Console.WriteLine("Breakfast is ready!");

#!markdown

Let's make breakfast in a more asynchronous way, stepping away from the toaster and the stovetop while the bacon and eggs are cooking.

#!csharp

async Task<Toast> ToastBreadAsync(int slices)
{
		for (int slice = 0; slice < slices; slice++)
		{
				Console.WriteLine("Putting a slice of bread in the toaster");
		}
		Console.WriteLine("Start toasting...");
		await Task.Delay(3000);
		Console.WriteLine("Remove toast from toaster");

		return new Toast();
}

async Task<Bacon> FryBaconAsync(int slices)
{
		Console.WriteLine($"putting {slices} slices of bacon in the pan");
		Console.WriteLine("cooking first side of bacon...");
		await Task.Delay(3000);
		for (int slice = 0; slice < slices; slice++)
		{
				Console.WriteLine("flipping a slice of bacon");
		}
		Console.WriteLine("cooking the second side of bacon...");
		await Task.Delay(3000);
		Console.WriteLine("Put bacon on plate");

		return new Bacon();
}

async Task<Egg> FryEggsAsync(int howMany)
{
		Console.WriteLine("Warming the egg pan...");
		await Task.Delay(3000);
		Console.WriteLine($"cracking {howMany} eggs");
		Console.WriteLine("cooking the eggs ...");
		await Task.Delay(3000);
		Console.WriteLine("Put eggs on plate");
		
		return new Egg();
}


Coffee cup = PourCoffee();
Console.WriteLine("coffee is ready");

Task<Egg> eggsTask = FryEggsAsync(2);
Task<Bacon> baconTask = FryBaconAsync(3);
Task<Toast> toastTask = ToastBreadAsync(2);

Toast toast = await toastTask;
ApplyButter(toast);
ApplyJam(toast);
Console.WriteLine("toast is ready");
Juice oj = PourOJ();
Console.WriteLine("oj is ready");

Egg eggs = await eggsTask;
Console.WriteLine("eggs are ready");
Bacon bacon = await baconTask;
Console.WriteLine("bacon is ready");

Console.WriteLine("Breakfast is ready!");

#!markdown

## The story of ConfigureAwait

When async methods run, you usually want to resume execution on the thread that called the method.  This is the default behavior.  However, if you want to allow execution to resume on a different thread, you can call the `ConfigureAwait` method with a `false` value.  By Default, `ConfigureAwait` is set to  `true` and the method will always resume execution on the thread that called the method. 

As application developers, we usually want our code to return on the same calling thread and this should NOT be re-configured.  If you are building a general purpose library and are taking control of the threading, you can call the `ConfigureAwait` method with a `false` value.  

Typical examples where you don't want to resume execution on the calling thread are:

- GUI applications like Windows Forms or WPF that handle a control action and need to update the UI on the calling thread
- Web Applications need to return to the same thread that called the method in order to respond properly to the client

Stephen Toub from the .NET team has a [great article diving deeper on this topic of ConfigureAwait](https://devblogs.microsoft.com/dotnet/configureawait-faq/).

#!csharp

public async Task MakeEggs()
{
	display(System.Threading.Thread.CurrentThread.ManagedThreadId);
	await Task.Delay(2000).ConfigureAwait(true);
	display("2 seconds later, I'm done making eggs.");
	display(System.Threading.Thread.CurrentThread.ManagedThreadId);
}

await MakeEggs();

#!markdown

## CPU intensive tasks and Task.Run

You can queue work to *potentially* be executed on another thread with the Task.Run method.  This method is similar to the `Task.Factory.StartNew` method, but it does not return a `Task` object.  Instead, it returns a `Task` object that represents the work that was queued.  The `Task` object is a pointer to the state of the background process and it expects to return from the completed process a `Task` object.

#!csharp

void ShowThreadInfo(String s)
{
	Console.WriteLine("{0} thread ID: {1}",
		s, System.Threading.Thread.CurrentThread.ManagedThreadId);
}

ShowThreadInfo("Application");

var t = Task.Run(() => ShowThreadInfo("Task") );
t.Wait();

#!markdown

When you're running a long running task with this method, you will want the ability to cancel processing at some point.  You can do this by passing a `CancellationToken` object that will signal when the task should stop.

You can pass the token to the `Task.Run` method as the last parameter.  The `Task.Run` method will only start the task if the token is not cancelled.

#!csharp

using System.Threading;
using System.Threading.Tasks;

var source = new CancellationTokenSource();
var token = source.Token;

var t = Task.Run(async () =>
{
	while (!token.IsCancellationRequested)
	{
		display("*");
		await Task.Delay(100, token);
	}
	
	display("Task completed (Inner).");
	
}, token);

await Task.Delay(300);
source.Cancel();
// await Task.Delay(1000);
display($"Task is cancelled: {t.IsCanceled}");
// t.Wait();

if (t.IsCompleted)
{
	display("Task is now complete.");
} else {
	display("Task is still running.");
	source.Dispose();
}

#!markdown

## Simple parallel loops using Parallel.ForEach

You can write a for loop to iterate over a collection of data in parallel using the `Parallel.ForEach` method.	This method takes a collection and a method that will be executed on each item in the collection.  The method is executed on a separate thread for each item in the collection, up to the number of allocated threads.  The method is executed in parallel and the loop is executed in parallel.

Let's look at the [Prime number example from the Microsoft Docs](https://docs.microsoft.com/dotnet/standard/parallel-programming/how-to-write-a-simple-parallel-foreach-loop).	We'll use the `Parallel.ForEach` method to calculate the prime numbers up to the number 2 million.

#!csharp

using System.Diagnostics;

bool IsPrime(int number)
{
	if (number < 2)
	{
		return false;
	}

	for (var divisor = 2; divisor <= Math.Sqrt(number); divisor++)
	{
		if (number % divisor == 0)
		{
			return false;
		}
	}
	return true;
}

// Get primes synchronously
IList<int> GetPrimeList(IList<int> numbers) 
	=> numbers.Where(IsPrime).ToList();

IList<int> GetPrimeListWithParallel(IList<int> numbers)
{
	var primeNumbers = new System.Collections.Concurrent.ConcurrentBag<int>();

	display($"Processors: {Environment.ProcessorCount}");
	var options = new ParallelOptions() { 
		MaxDegreeOfParallelism = Environment.ProcessorCount 
	};

	Parallel.ForEach(numbers, options, number =>
	{
		if (IsPrime(number)) primeNumbers.Add(number);
	} );

  return primeNumbers.ToList();
}


var limit = 2_000_000;
var numbers = Enumerable.Range(0, limit).ToList();

var watch = Stopwatch.StartNew();
var primeNumbersFromForeach = GetPrimeList(numbers);
watch.Stop();

var watchForParallel = Stopwatch.StartNew();
var primeNumbersFromParallelForeach = GetPrimeListWithParallel(numbers);
watchForParallel.Stop();

display($"Classical foreach loop | Total prime numbers : {primeNumbersFromForeach.Count} | Time Taken : {watch.ElapsedMilliseconds} ms.");
display($"Parallel.ForEach loop  | Total prime numbers : {primeNumbersFromParallelForeach.Count} | Time Taken : {watchForParallel.ElapsedMilliseconds} ms.");
