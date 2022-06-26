# Azure WebJobs: Tips & Tricks

## Triggering a logic when a blob is added/modified
If you need to trigger a code automatically when a blob is added/modified, you can simply use [BlobTrigger](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob?tabs=in-process%2Cextensionv5%2Cextensionv3&pivots=programming-language-csharp). The trigger runs
a function when blob storage data changes.
