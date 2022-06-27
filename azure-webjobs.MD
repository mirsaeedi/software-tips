# Azure WebJobs: Tips & Tricks

*  WebJobs typically need the `Always On` setting; otherwise a web app can time out after 20 minutes of inactivity. `Always On` is supported only by Basic and higher App Service Plans.
* Use SDK version 5.X. (The official docs are still advising you to use 3.X but they'll be updated soon).
* Beginning with 5.x, Microsoft.Azure.WebJobs.Extensions.Storage has been split by storage service and has migrated the AddAzureStorage() extension method by service type, `AddAzureStorageQueues`, `AddAzureStorageBlobs`, etc.
* Azure Functions is build on top of Azure WebJobs SDK. So, many Azure Functions docs are also applicable to Azure WebJobs.
* WebJobs SDK needs a Storage Account and reads connection string via the `AzureWebJobsStorage` environment variable.
* At minimum, you must call `AddAzureStorageCoreServices` to configure the base functionality. If you don't call it, then some functions like singleton timers won't work.
```C#
static async Task Main()
{
    var builder = new HostBuilder();
    builder.UseEnvironment("development");
    builder.ConfigureWebJobs(b =>
            {
                b.AddAzureStorageCoreServices();
            });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```
* To use Azure Storage integration, install the [`Microsoft.Azure.WebJobs.Extensions.Storage`](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage) package. And call the config method.
```C#
static async Task Main()
{
    var builder = new HostBuilder();
    builder.UseEnvironment("development");
    builder.ConfigureWebJobs(b =>
            {
                b.AddAzureStorageCoreServices();
                b.AddAzureStorageBlobs(); // sets up blob triggers
                b.AddAzureStorageQueues(); // sets up queue triggers;
            });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```
* Do not hardcode the queue names, settings, and cron expressions. In the following, the queue name, `QueueName`, and the cron expression, `TimerSchedule`, are read from configs that are loaded by `ConfigureAppConfiguration` during boot:
```C#
    public async Task Aggregate([TimerTrigger("%TimerSchedule%")] TimerInfo _)
    {
    }

    public async Task ProcessQueueMessage([QueueTrigger("%QueueName%")] QueueMessage queueMessage)
    {
    }
```
* If you want to ensure that only one instance of a function runs even when there are multiple instances of the host web app, you can use the [Singleton attribute](https://docs.microsoft.com/en-us/azure/app-service/webjobs-sdk-how-to#singleton-attribute).
* Timer trigger is by default singleton. If you got multiple machines, only one of them will execute the trigger. This is garaunteed by the WebJobs SDK using [distributed locking via leasing blobs](https://docs.microsoft.com/en-us/azure/app-service/webjobs-sdk-how-to#viewing-lease-blobs).

## Application Insights

* In StartUp.cs, integrate your web jobs with Application Insights to monitor the logs and metrics.
```C#
    builder.ConfigureLogging((context, builder) =>
    {
        // In dev env, use console logging.
        if (context.HostingEnvironment.IsDevelopment() || IsConsoleLoggingEnabled())
        {
            builder.AddConsole();
        }

        // In prod env, use app insights.
        string appInsightsConnectionString = context.Configuration["APPLICATIONINSIGHTS_CONNECTION_STRING"];
        if (context.HostingEnvironment.IsProduction())
        {
            builder.AddApplicationInsightsWebJobs(o => o.ConnectionString = appInsightsConnectionString);
        }
    });
```

## Dependency Injection
In official docs, you see that functions are defined as static in a static class. This approach doesn't allow you to inject dependencies via IoC container. However, Azure WebJobs SDK does [support](https://docs.microsoft.com/en-us/azure/azure-functions/functions-dotnet-dependency-injection) the dependency injection (DI) software design pattern. The only thing you need to do is to create a non-static class to recieve dependecies through constructor injection.
```C#
 public class QueueTriggeredJobs
    {
        private readonly MyService _myService;
        private readonly ILogger<QueueTriggeredJobs> _logger;

        public QueueTriggeredJobs(
            MyService myService,
            ILogger<QueueTriggeredJobs> logger)
        {
            _myService = myService;
            _logger = logger;
        }

        public async Task ProcessQueueMessage([QueueTrigger("%Queue:KeepaTaskExecutor%")] QueueMessage queueMessage)
        {
            _logger.Log("Hello world");
            _myService.Do();
        }
    }
```
Lastly, you got to add the class and its dependencies to the IoC container.

```C#
services.AddScoped<QueueTriggeredJobs>();
services.AddScoped<MyService>();
```

## Queue
* The queue trigger leverage a technique called `Peek lock` to automatically prevents a function from processing a queue message on multiple machine times simultaneously. As messages are dequeued, they are marked as invisible and associated with a timeout managed by the Storage service. When the function starts, it starts processing a message under the following conditions.
    1. If the function is successful, then the function execution completes and the message is deleted.
    2. If the function fails, then the message visibility is reset. After being reset, the message is reprocessed the next time the function requests a new message.
    3. If the function never completes due to a crash, the message visibility expires and the message re-appears in the queue.
* Expanding the last point: if processing a message fails, the message will be retried 5 more times. The default value is configurable via `DequeueCount`. If all retires fail, the messsage goes to [`Poison Queue`](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-queue-trigger?tabs=in-process%2Cextensionv5&pivots=programming-language-csharp#poison-messages).
* The concurrency behavior of queue trigger is explained [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-queue-trigger?tabs=in-process%2Cextensionv5&pivots=programming-language-csharp#concurrency).
* The queue trigger uses a polling algorithm to find out if a message has been pushed to the queue. The algorithm uses the following logic:
    * When a message is found, the runtime waits 100 milliseconds and then checks for another message
    * When no message is found, it waits about 200 milliseconds before trying again.
    * After subsequent failed attempts to get a queue message, the wait time continues to increase until it reaches the maximum wait time, which defaults to one minute.
    * The maximum wait time is configurable via the maxPollingInterval property in the host.json file.

## [BlobTrigger]: Triggering a logic when a blob is added/modified
If you need to trigger a code automatically when a blob is added/modified, you can simply use [BlobTrigger](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob?tabs=in-process%2Cextensionv5%2Cextensionv3&pivots=programming-language-csharp). The trigger runs
a function when blob storage data changes. The following functions runs when a blob in `samples-workitems` is added or modified. The blob name will be bound to the `name` variable.


_Default Strategy_

```C#
public static void Run([BlobTrigger("samples-workitems/{name}")] Stream myBlob, string name)
{
    
}
```

[_EventGrid Strategy_](https://docs.microsoft.com/en-us/azure/azure-functions/functions-event-grid-blob-trigger?tabs=csharp)

```C#
public static void Run([BlobTrigger("samples-workitems/{name}"), Source = BlobTriggerSource.EventGrid] Stream myBlob, string name)
{
    
}
```

_Accessing Blob container_

```C#
public static void Run([BlobTrigger("samples-workitems/{name}"), [Blob("sample-container")] BlobContainerClient blobContainerClient] Stream myBlob, string name)
{
    
}
```

### Notes
* The blob trigger will process all existing blobs in the container when you set up the trigger. If you have a container with many existing blobs and only want to trigger for new blobs, use the Event Grid trigger. (I learnt it the hard way! my code got triggered for all the old blobs 🤣 )
* Instead of `Stream` param, you can use `string` or `byte[]` but it loads all the file content into memory.
* You can filter on blob name. For example, `samples-workitems/images-{name}.png` triggers code for PNG files that their name start with `images`.
* Azure Sdk [Samples](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/storage/Microsoft.Azure.WebJobs.Extensions.Storage.Blobs#examples) worth a read.
* I use BlobTrigger to only recieve update notifications. I don't process the file in the BlobTrigger functton. To process the file, I pushed the file name into queue. So, I can get the benefit of queue processing like load balancing, retry, and poison queue.
