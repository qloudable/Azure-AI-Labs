# Implement Computer Vision
  
## Table of Contents

[Overview](#overview)

[Pre-Requisites](#pre-requisites)

[Intercepting and analyzing messages](#intercepting-and-analyzing-messages)

[Logging to Azure Storage](#logging-to-azure-storage)

[Logging utterances to a file](#logging-utterances-to-a-file)

## Overview
This workshop demonstrates how you can perform logging using Microsoft Bot Framework and store aspects of chat conversations. After completing these labs, you should be able to:

- Understand how to intercept and log message activities between bots and users
- Log utterances to file storage

**Note:** This lab is meant for an Artificial Intelligence (AI) Engineer or an AI Developer on Azure. 

**Architecture:**
<img src="https://raw.githubusercontent.com/qloudable/Azure-AI-Labs/master/images/AI_Immersion_Arch.png" alt="image-alt-text">

In the continuation of this lab throughout the course, we'll show you how to query your data, and then build a Bot Framework bot to query it. Finally, we'll extend this bot with LUIS to automatically derive intent from your queries and use those to direct your searches intelligently.

**Some Key points;**

**We recommend using Chrome or Edge as the broswer. Also set your browser zoom to 80%**


- All screen shots are examples ONLY. Screen shots can be enlarged by Clicking on them

- Do NOT use resource name and other data from screen shots. Only use information provided in the content section of the lab

- Mac OS Users should use ctrl+C / ctrl+V to copy and paste inside the Azure Console

- Login credentials are provided later in the guide (scroll down). Every User MUST keep these credentials handy.

**Tenant Id**
**Subscripiton Id**
**Resource Group**
**User Name**
**Password**

This lab is part of course "Design & Implement Azure AI Solution". We recommend you to take the labs in following order: 

- [Lab1 - Implement Computer Vision API](/lab/3f89185f-0da6-4f68-b66b-19daaba28141)

- [Lab2 - Create An Intelligent Chat Bot](/lab/9ed93b70-1fe5-47a3-835a-82fa346fd429)

- [Lab3 - Log Chat Bot Conversations](/lab/69e9e2ab-055d-48f6-a1f9-bd24244309a2)

- [Lab4 - Implement QnA Maker](/lab/35a6c467-5a2c-4736-8c63-f2aabf2d13df)

- [Lab5 - Implement LUIS Model](/lab/99f1db25-5698-48ea-9cf6-a8d0dbca4fbb)

- [Lab6 - Integrate LUIS with Chat Bot](/lab/916fbac5-930c-4276-9829-1082d9b7043e)

- [Lab7 - Detect User Language in Chat Bot](/lab/dbeefcdd-ecd9-4f3e-b8a5-9fdfc0b0b3c9)

- [Lab8 - Test Bots In DirectLine](/lab/2f7669b0-6be6-451a-a05f-c9f14d869388)

## Pre-Requisites

This lab is meant for an Artificial Intelligence (AI) Engineer or an AI Developer on Azure. To ensure you have time to work through the exercises, there are certain requirements to meet before starting the labs for this course.

You should ideally have some previous exposure to Visual Studio. We will be using it for everything we are building in the labs, so you should be familiar with [how to use it](https://docs.microsoft.com/en-us/visualstudio/ide/visual-studio-ide) to create applications. Additionally, this is not a class where we teach code or development. We assume you have some familiarity with C# (intermediate level - you can learn [here](https://mva.microsoft.com/en-us/training-courses/c-fundamentals-for-absolute-beginners-16169?l=Lvld4EQIC_2706218949) and [here](https://docs.microsoft.com/en-us/dotnet/csharp/quick-starts/)), but you do not know how to implement solutions with Cognitive Services.


## Intercepting and analyzing messages

In this lab, we'll look at some different ways that the Bot Framework allows us to intercept and log data from conversations that the bot has with users. To start we will utilize the In Memory storage, this is good for testing purposes, but not ideal for production environments.

Afterwards, we'll look at a very simple implementation of how we can write data from conversations to files in Azure. Specifically, we'll put messages users send to the bot in a list, and store the list, along with a few other items, in a temporary file (though you could change this to a specific file path as needed)

#### Using the Bot Framework Emulator

Let's take a look and what information we can glean, for testing purposes, without adding anything to our bot.

1.  Open your **PictureBot.sln** in Visual Studio. 

> **Note** You can use the **Starter** solution if you did not do Lab 3.

1.  Press **F5** to run your bot

1.  Open the bot in the Bot Framework Emulator and have a quick conversastion:

1.  Review the bot emulator debug area, notice the following:

- If you click on a message, you are able to see its associated JSON with the "Inspector-JSON" tool on the right. Click on a message and inspect the JSON to see what information you can obtain.

- The "Log" in the bottom right-hand corner, contains a complete log of the conversation. Let's dive into that a little deeper.

    - The first thing you'll see is the port the Emulator is listening on

    - You'll also see where ngrok is listening, and you can inspect the traffic to ngrok using the "ngrok traffic inspector" link. However, you should notice that we will bypass ngrok if we're hitting local addresses. **ngrok is included here informationally, as remote testing is not covered in this workshop**

    - If there is an error in the call (anything other than POST 200 or POST 201 reponse), you'll be able to click it and see a very detailed log in the "Inspector-JSON". Depending on what the error is, you may even get a stack trace going through the code and attempting to point out where the error has occurred. This is greatly useful when you're debugging your bot projects.

    - You can also see that there is a `Luis Trace` when we make calls out to LUIS. If you click on the `trace` link, you're able to see the LUIS information. You may notice that this is not set up in this particular lab.

![Emulator](../images/emulator.png)

You can read more about testing, debugging, and logging with the emulator [here](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0).

## Logging to Azure Storage

The default bot storage provider uses in-memory storage that gets disposed of when the bot is restarted. This is good for testing purposes only. If you want to persist data but do not want to hook your bot up to a database, you can use the Azure storage provider or build your own provider using the SDK. 

1.  Open the **Startup.cs** file.  Since we want to use this process for every message, we'll use the `ConfigureServices` method in our Startup class to add storing information to an Azure Blob file. Notice that currently we're using:

```csharp
IStorage dataStore = new MemoryStorage();
```

As you can see, our current implementation is using in-memory storage. Again, this memory storage is recommended for local bot debugging only. When the bot is restarted, anything stored in memory will be gone.

1.  Replace the current `IStorage` line with the following to change it to a FileStorage based persistance:

```csharp
var blobConnectionString = Configuration.GetSection("BlobStorageConnectionString")?.Value;
var blobContainer = Configuration.GetSection("BlobStorageContainer")?.Value;
IStorage dataStore = new Microsoft.Bot.Builder.Azure.AzureBlobStorage(blobConnectionString, blobContainer);
```

1.  Switch to the Azure Portal, navigate to your blob storage account

1.  From the **Overview** tab, click **Containers**

1.  Check if a **chatlog** container exists, if it does not click **+Container**:

    -   For the name, type **chatlog**, click **OK**

1.  If you haven't already done so, click **Access keys** and record your connection string

1.  Open the **appsettings.json** and add your blob connection string details:

```json
"BlobStorageConnectionString": "",
"BlobStorageContainer" :  "chatlog"
```

1.  Press **F5** to run the bot. 

1.  In the emulator, go through a sample conversation with the bot.

> **Note** If you dont get a reply back, check your Azure Storage Account connection string

1.  Switch to the Azure Portal, navigate to your blob storage account

1.  Click **Containers**, then open the **ChatLog** container

1.  Select the chat log file, it should start with **emulator...**, then select **Edit blob**.  What do you see in the files? What don't you see that you were expecting/hoping to see?

You should see something similar to this:

```json
{"$type":"System.Collections.Concurrent.ConcurrentDictionary`2[[System.String, System.Private.CoreLib],[System.Object, System.Private.CoreLib]], System.Collections.Concurrent","DialogState":{"$type":"Microsoft.Bot.Builder.Dialogs.DialogState, Microsoft.Bot.Builder.Dialogs","DialogStack":{"$type":"System.Collections.Generic.List`1[[Microsoft.Bot.Builder.Dialogs.DialogInstance, Microsoft.Bot.Builder.Dialogs]], System.Private.CoreLib","$values":[{"$type":"Microsoft.Bot.Builder.Dialogs.DialogInstance, Microsoft.Bot.Builder.Dialogs","Id":"mainDialog","State":{"$type":"System.Collections.Generic.Dictionary`2[[System.String, System.Private.CoreLib],[System.Object, System.Private.CoreLib]], System.Private.CoreLib","options":null,"values":{"$type":"System.Collections.Generic.Dictionary`2[[System.String, System.Private.CoreLib],[System.Object, System.Private.CoreLib]], System.Private.CoreLib"},"instanceId":"f80db88d-cdea-4b47-a3f6-a5bfa26ed60b","stepIndex":0}}]}},"PictureBotAccessors.PictureState":{"$type":"Microsoft.PictureBot.PictureState, PictureBot","Greeted":"greeted","Search":"","Searching":"no"}}
```

## Logging utterances to a file

For the purposes of this lab, we are going to focus on the actual utterances that users are sending to the bot. This could be useful to determine what types of conversations and actions users are trying to complete with the bot.

We can do this by updating what we're storing in our `UserData` object in the **PictureState.cs** file and by adding information to the object in **PictureBot.cs**:

1.  Open **PictureState.cs**

1.  **after** the following code:

```csharp
public class UserData
{
    public string Greeted { get; set; } = "not greeted";
```

add:

```csharp
// A list of things that users have said to the bot
public List<string> UtteranceList { get; private set; } = new List<string>();
```

In the above, we're simple creating a list where we'll store the list of messages that users send to the bot.

In this example we're choosing to use the state manager to read and write data, but you could alternatively [read and write directly from storage without using state manager](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-v4-storage?view=azure-bot-service-4.0&tabs=csharpechorproperty%2Ccsetagoverwrite%2Ccsetag).

> If you choose to write directly to storage, you could set up eTags depending on your scenario. By setting the eTag property to `*`, you could allow other instances of the bot to overwrite previously written data, meaning that the last writer wins. We won't get into it here, but you can [read more about managing concurrency](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-v4-storage?view=azure-bot-service-4.0&tabs=csharpechorproperty%2Ccsetagoverwrite%2Ccsetag#manage-concurrency-using-etags).

The final thing we have to do before we run the bot is add messages to our list with our `OnTurn` action. 

1.  In **PictureBot.cs**, **after** the following code:

```csharp
public override async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
        {
            if (turnContext.Activity.Type is "message")
            {
```

add:

```csharp
var utterance = turnContext.Activity.Text;
var state = await _accessors.PictureState.GetAsync(turnContext, () => new PictureState());
state.UtteranceList.Add(utterance);
await _accessors.ConversationState.SaveChangesAsync(turnContext);
```

> **Note** We have to save the state if we modify it

The first line takes the incoming message from a user and stores it in a variable called `utterance`. The next line adds the utterance to the existing list that we created in PictureState.cs.

1.  Press **F5** to run the bot.

1.  Have another conversation with the bot. Stop the bot and check the latest blob persisted log file. What do we have now?

```json
{"$type":"System.Collections.Concurrent.ConcurrentDictionary`2[[System.String, System.Private.CoreLib],[System.Object, System.Private.CoreLib]], System.Collections.Concurrent","DialogState":{"$type":"Microsoft.Bot.Builder.Dialogs.DialogState, Microsoft.Bot.Builder.Dialogs","DialogStack":{"$type":"System.Collections.Generic.List`1[[Microsoft.Bot.Builder.Dialogs.DialogInstance, Microsoft.Bot.Builder.Dialogs]], System.Private.CoreLib","$values":[{"$type":"Microsoft.Bot.Builder.Dialogs.DialogInstance, Microsoft.Bot.Builder.Dialogs","Id":"mainDialog","State":{"$type":"System.Collections.Generic.Dictionary`2[[System.String, System.Private.CoreLib],[System.Object, System.Private.CoreLib]], System.Private.CoreLib","options":null,"values":{"$type":"System.Collections.Generic.Dictionary`2[[System.String, System.Private.CoreLib],[System.Object, System.Private.CoreLib]], System.Private.CoreLib"},"instanceId":"f80db88d-cdea-4b47-a3f6-a5bfa26ed60b","stepIndex":0}}]}},"PictureBotAccessors.PictureState":{"$type":"Microsoft.PictureBot.PictureState, PictureBot","Greeted":"greeted","UtteranceList":{"$type":"System.Collections.Generic.List`1[[System.String, System.Private.CoreLib]], System.Private.CoreLib","$values":["help"]},"Search":"","Searching":"no"}}
```

>Get stuck or broken? You can find the solution for the lab up until this point under [/code/PictureBot-FinishedSolution-File](./code/PictureBot-FinishedSolution-File). You will need to insert the keys for your Azure Bot Service and your Azure Storage settings in the `appsettings.json` file. We recommend using this code as a reference, not as a solution to run, but if you choose to run it, be sure to add the necessary keys.

#### Next Steps
-  [Lab4 - Implement QnA Maker](/lab/35a6c467-5a2c-4736-8c63-f2aabf2d13df)

***Congratulations! You have successfully completed the lab. ***