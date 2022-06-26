# Azure WebJobs: Tips & Tricks

## Triggering a logic when a blob is added/modified
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
