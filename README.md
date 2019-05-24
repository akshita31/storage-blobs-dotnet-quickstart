---
topic: sample
languages:
  - csharp
products:
  - azure
  - azure-storage
  - dotnet-core
name: Transfer objects to and from Azure Blob Storage using .NET
description: A simple sample project to help you get started using Azure Storage with .NET Core and C# as the development language.
services: storage
platforms: dotnet
author: georgewallace
---

# Transfer objects to and from Azure Blob Storage using .NET

This repository contains a simple sample project to help you get started using Azure Storage with .NET Core and C# as the development language.

## Prerequisites

To complete this tutorial:

* Install .NET core 2.1 for [Linux](/dotnet/core/linux-prerequisites?tabs=netcore2x) or [Windows](/dotnet/core/windows-prerequisites?tabs=netcore2x)

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

## Create a storage account using the Azure portal

First, create a new general-purpose storage account to use for this quickstart.

1. Go to the [Azure portal](https://portal.azure.com) and log in using your Azure account.
2. On the Hub menu, select **New** > **Storage** > **Storage account - blob, file, table, queue**.
3. Enter a unique name for your storage account. Keep these rules in mind for naming your storage account:
    * The name must be between 3 and 24 characters in length.
    * The name may contain numbers and lowercase letters only.
4. Make sure that the following default values are set:
    * **Deployment model** is set to **Resource manager**.
    * **Account kind** is set to **General purpose**.
    * **Performance** is set to **Standard**.
    * **Replication** is set to **Locally Redundant storage (LRS)**.
5. Select your subscription.
6. For **Resource group**, create a new one and give it a unique name.
7. Select the **Location** to use for your storage account.
8. Check **Pin to dashboard** and click **Create** to create your storage account.

After your storage account is created, it is pinned to the dashboard. Click on it to open it. Under **Settings**, click **Access keys**. Select the primary key and copy the associated **Connection string** to the clipboard, then paste it into a text editor for later use.

## Configure your storage connection string

To run the application, you must provide the connection string for your storage account. The sample application reads the connection string from an environment variable and uses it to authorize requests to Azure Storage.

After you have copied your connection string, write it to a new environment variable on the local machine running the application. To set the environment variable, open a console window, and follow the instructions for your operating system. Replace `<yourconnectionstring>` with your actual connection string:

# [Windows](#tab/windows)

```cmd
setx storageconnectionstring "<yourconnectionstring>"
```

After you add the environment variable, you may need to restart any running programs that will need to read the environment variable, including the console window. For example, if you are using Visual Studio as your editor, restart Visual Studio before running the sample.

# [Linux](#tab/linux)

```bash
export storageconnectionstring=<yourconnectionstring>
```

After you add the environment variable, run `source ~/.bashrc` from your console window to make the changes effective.

# [macOS](#tab/macos)

Edit your .bash_profile, and add the environment variable:

```bash
export storageconnectionstring=<yourconnectionstring>
```

After you add the environment variable, run `source .bash_profile` from your console window to make the changes effective.

---

## Run the sample

This sample creates a test file in your local **MyDocuments** folder and uploads it to Blob storage. The sample then lists the blobs in the container and downloads the file with a new name so that you can compare the old and new files.

# [Windows](#tab/windows)

If you are using Visual Studio as your editor, you can press **F5** to run.

Otherwise, navigate to your application directory and run the application with the `dotnet run` command.

```console
dotnet run
```

# [Linux](#tab/linux)

Navigate to your application directory and run the application with the `dotnet run` command.

```console
dotnet run
```

# [macOS](#tab/macos)

Navigate to your application directory and run the application with the `dotnet run` command.

```console
dotnet run
```

---

The output of the sample application is similar to the following example:

```output
Azure Blob storage - .NET Quickstart sample

Created container 'quickstartblobs33c90d2a-eabd-4236-958b-5cc5949e731f'

Temp file = C:\Users\myusername\Documents\QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db.txt
Uploading to Blob storage as blob 'QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db.txt'

Listing blobs in container.
https://storagesamples.blob.core.windows.net/quickstartblobs33c90d2a-eabd-4236-958b-5cc5949e731f/QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db.txt

Downloading blob to C:\Users\myusername\Documents\QuickStart_c5e7f24f-a7f8-4926-a9da-9697c748f4db_DOWNLOADED.txt

Press any key to delete the sample files and example container.
```

When you press the **Enter** key, the application deletes the storage container and the files. Before you delete them, check your **MyDocuments** folder for the two files. You can open them and observe that they are identical. Copy the blob's URL from the console window and paste it into a browser to view the contents of the blob.

After you've verified the files, hit any key to finish the demo and delete the test files. Now that you know what the sample does, open the Program.cs file to look at the code.

## Understand the sample code

Next, explore the sample code so that you can understand how it works.

### Try parsing the connection string

The first thing that the sample does is to check that the environment variable contains a connection string that can be parsed to create a [CloudStorageAccount](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount) object pointing to the storage account. To check that the connection string is valid, use the [TryParse](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount.tryparse) method. If **TryParse** is successful, it initializes the *storageAccount* variable and returns **true**.

```csharp --source-file storage-blobs-dotnet-quickstart/Program.cs --region ParseConnectionString --project storage-blobs-dotnet-quickstart/storage-blobs-dotnet-quickstart.csproj --session sample1
// Retrieve the connection string for use with the application. The storage connection string is stored
// in an environment variable on the machine running the application called storageconnectionstring.
// If the environment variable is created after the application is launched in a console or with Visual
// Studio, the shell or application needs to be closed and reloaded to take the environment variable into account.
string storageConnectionString = Environment.GetEnvironmentVariable("storageconnectionstring");

// Check whether the connection string can be parsed.
CloudStorageAccount storageAccount;
if (CloudStorageAccount.TryParse(storageConnectionString, out storageAccount))
{
    // If the connection string is valid, proceed with operations against Blob storage here.
    ...
}
else
{
    // Otherwise, let the user know that they need to define the environment variable.
    Console.WriteLine(
        "A connection string has not been defined in the system environment variables. " +
        "Add an environment variable named 'storageconnectionstring' with your storage " +
        "connection string as a value.");
    Console.WriteLine("Press any key to exit the sample application.");
    Console.ReadLine();
}
```

### Create the container and set permissions

Next, the sample creates a container and sets its permissions so that any blobs in the container are public. If a blob is public, it can be accessed anonymously by any client.

To create the container, first create an instance of the [CloudBlobClient](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient) object, which points to Blob storage in your storage account. Next, create an instance of the [CloudBlobContainer](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer) object, then create the container.

In this case, the sample calls the [CreateAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.createasync) method to create the container. A GUID value is appended to the container name to ensure that it is unique. In a production environment, it's often preferable to use the [CreateIfNotExistsAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.createifnotexistsasync) method to create a container only if it does not already exist and avoid naming conflicts.

> [!IMPORTANT]
> Container names must be lowercase. For more information about naming containers and blobs, see [Naming and Referencing Containers, Blobs, and Metadata](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata).

```csharp --source-file storage-blobs-dotnet-quickstart/Program.cs --region CreateContainerAndSetPermissions --project storage-blobs-dotnet-quickstart/storage-blobs-dotnet-quickstart.csproj --session sample1
// Create the CloudBlobClient that represents the Blob storage endpoint for the storage account.
CloudBlobClient cloudBlobClient = storageAccount.CreateCloudBlobClient();

// Create a container called 'quickstartblobs' and append a GUID value to it to make the name unique.
CloudBlobContainer cloudBlobContainer = cloudBlobClient.GetContainerReference("quickstartblobs" + Guid.NewGuid().ToString());
await cloudBlobContainer.CreateAsync();

// Set the permissions so the blobs are public.
BlobContainerPermissions permissions = new BlobContainerPermissions
{
    PublicAccess = BlobContainerPublicAccessType.Blob
};
await cloudBlobContainer.SetPermissionsAsync(permissions);
```

### Upload blobs to the container

Next, the sample uploads a local file to a block blob. The code example gets a reference to a **CloudBlockBlob** object by calling the [GetBlockBlobReference](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.getblockblobreference) method on the container created in the previous section. It then uploads the selected file to the blob by calling the [​Upload​From​FileAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob.uploadfromfileasync) method. This method creates the blob if it doesn't already exist, and overwrites it if it does.

```csharp --source-file storage-blobs-dotnet-quickstart/Program.cs --region UploadBlobs --project storage-blobs-dotnet-quickstart/storage-blobs-dotnet-quickstart.csproj --session sample1
// Create a file in your local MyDocuments folder to upload to a blob.
string localPath = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
string localFileName = "QuickStart_" + Guid.NewGuid().ToString() + ".txt";
sourceFile = Path.Combine(localPath, localFileName);
// Write text to the file.
File.WriteAllText(sourceFile, "Hello, World!");

Console.WriteLine("Temp file = {0}", sourceFile);
Console.WriteLine("Uploading to Blob storage as blob '{0}'", localFileName);

// Get a reference to the blob address, then upload the file to the blob.
// Use the value of localFileName for the blob name.
CloudBlockBlob cloudBlockBlob = cloudBlobContainer.GetBlockBlobReference(localFileName);
await cloudBlockBlob.UploadFromFileAsync(sourceFile);
```

### List the blobs in a container

The sample lists the blobs in the container using the [​List​BlobsSegmentedAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobssegmentedasync) method. In the case of the sample, only one blob has been added to the container, so the listing operation returns just that one blob.

If there are too many blobs to return in one call (by default, more than 5000), then the **​List​BlobsSegmentedAsync** method returns a segment of the total result set and a continuation token. To retrieve the next segment of blobs, you provide in the continuation token returned by the previous call, and so on, until the continuation token is null. A null continuation token indicates that all of the blobs have been retrieved. The sample code shows how to use the continuation token for the sake of best practices.

```csharp --source-file storage-blobs-dotnet-quickstart/Program.cs --region ListBlobs --project storage-blobs-dotnet-quickstart/storage-blobs-dotnet-quickstart.csproj --session sample1
// List the blobs in the container.
Console.WriteLine("List blobs in container.");
BlobContinuationToken blobContinuationToken = null;
do
{
    var results = await cloudBlobContainer.ListBlobsSegmentedAsync(null, blobContinuationToken);
    // Get the value of the continuation token returned by the listing call.
    blobContinuationToken = results.ContinuationToken;
    foreach (IListBlobItem item in results.Results)
    {
        Console.WriteLine(item.Uri);
    }
} while (blobContinuationToken != null); // Loop while the continuation token is not null.

```

### Download blobs

Next, the sample downloads the blob created previously to your local file system using the [​Download​To​FileAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.downloadtofileasync) method. The sample code adds a suffix of "_DOWNLOADED" to the blob name so that you can see both files in local file system.

```csharp --source-file storage-blobs-dotnet-quickstart/Program.cs --region DownloadBlob --project storage-blobs-dotnet-quickstart/storage-blobs-dotnet-quickstart.csproj --session sample1
// Download the blob to a local file, using the reference created earlier.
// Append the string "_DOWNLOADED" before the .txt extension so that you can see both files in MyDocuments.
destinationFile = sourceFile.Replace(".txt", "_DOWNLOADED.txt");
Console.WriteLine("Downloading blob to {0}", destinationFile);
await cloudBlockBlob.DownloadToFileAsync(destinationFile, FileMode.Create);  
```

### Clean up resources

The sample cleans up the resources that it created by deleting the entire container using [Cloud​Blob​Container.​DeleteAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.deleteasync). You can also delete the local files if you like.

```csharp --source-file storage-blobs-dotnet-quickstart/Program.cs --region CleanUpResources --project storage-blobs-dotnet-quickstart/storage-blobs-dotnet-quickstart.csproj --session sample1
Console.WriteLine("Press the 'Enter' key to delete the sample files, example container, and exit the application.");
Console.ReadLine();
// Clean up resources. This includes the container and the two temp files.
Console.WriteLine("Deleting the container");
if (cloudBlobContainer != null)
{
    await cloudBlobContainer.DeleteIfExistsAsync();
}
Console.WriteLine("Deleting the source, and downloaded files");
File.Delete(sourceFile);
File.Delete(destinationFile);
```

## Resources for developing .NET applications with blobs

See these additional resources for .NET development with Blob storage:

### Binaries and source code

- Download the NuGet package for the latest version of the [.NET client library](https://www.nuget.org/packages/Microsoft.Azure.Storage.Blob/) for Azure Blob Storage.
- View the [Microsoft Azure Storage Blob SDK for .NET source code](https://github.com/Azure/azure-storage-net/tree/master/Blob) on GitHub.

### Client library reference and samples

- See the [.NET API reference](https://docs.microsoft.com/dotnet/api/overview/azure/storage) for more information about the .NET client library.
- Explore [Blob storage samples](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=dotnet&term=blob) written using the .NET client library.

## Next steps

In this quickstart, you learned how to upload, download, and list blobs using .NET.

To learn how to create a web app that uploads an image to Blob storage, continue to:

> [!div class="nextstepaction"]
> [Upload and process an image](storage-upload-process-images.md)

- To learn more about .NET Core, see [Get started with .NET in 10 minutes](https://www.microsoft.com/net/learn/get-started/).
- To explore a sample application that you can deploy from Visual Studio for Windows, see the [.NET Photo Gallery Web Application Sample with Azure Blob Storage](https://azure.microsoft.com/resources/samples/storage-blobs-dotnet-webapp/).

## More information

The [Azure storage documentation](https://docs.microsoft.com/azure/storage/) includes a rich set of tutorials and conceptual articles, which serve as a good complement to the samples.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.