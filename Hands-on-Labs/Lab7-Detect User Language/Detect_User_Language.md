# Implement Computer Vision
  
## Table of Contents

[Overview](#overview)

[Pre-Requisites](#pre-requisites)

[Retrieve your Cognitive Services url and keys](#retrieve-your-cognitive-services-url-and-keys)

[Add language support to your bot](#add-language-support-to-your-bot)

[Delete the resources](#delete-the-resources)


## Overview
In this lab we are going to integrate language detection ability of cognitive services into our bot.

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


## Retrieve your Cognitive Services url and keys

1.  Open the [Azure Portal](https://portal.azure.com)

1.  Navigate to your resource group, select the cognitive services resource that is generic (aka, it contains all end points).

1.  Under **RESOURCE MANAGEMENT**, select the **Quick Start** tab and record the url and the key for the cognitive services resource

##  Add language support to your bot

1.  If not already open, open your **PictureBot** solution

1.  Right-click the project and select **Manage Nuget Packages**

1.  Click **Browse**

1.  Search for **Microsoft.Azure.CognitiveServices.Language.TextAnalytics**, select it then click **Install**, then click **I Accept**

1.  Open the **Startup.cs** file, add the following using statements:

```csharp
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics;
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
```

1.  Add the following code to the **ConfigureServices** method:

```csharp
services.AddSingleton(sp =>
{
    string cogsBaseUrl = Configuration.GetSection("cogsBaseUrl")?.Value;
    string cogsKey = Configuration.GetSection("cogsKey")?.Value;

    var credentials = new ApiKeyServiceClientCredentials(cogsKey);
    TextAnalyticsClient client = new TextAnalyticsClient(credentials)
    {
        Endpoint = cogsBaseUrl
    };

    return client;
});
```

1.  Open the **PictureBot.cs** file, add the following using statements:

```csharp
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics;
using Microsoft.Azure.CognitiveServices.Language.TextAnalytics.Models;
```

1.  Add the following class variable:

```csharp
private TextAnalyticsClient _textAnalyticsClient;
```

1.  Modify the constructor to include the new TextAnalyticsClient:

```csharp
public PictureBot(PictureBotAccessors accessors, ILoggerFactory loggerFactory,LuisRecognizer recognizer, TextAnalyticsClient analyticsClient)
```

1.  Inside the constructor, initialize the class variable:

```csharp
_textAnalyticsClient = analyticsClient;
```

1.  Navigate to the **OnTurnAsync** method and find the following line of code:

```csharp
var utterance = turnContext.Activity.Text;
var state = await _accessors.PictureState.GetAsync(turnContext, () => new PictureState());
state.UtteranceList.Add(utterance);
await _accessors.ConversationState.SaveChangesAsync(turnContext);
```

1.  Add the following line of code after it

```csharp
//Check the language
var result = _textAnalyticsClient.DetectLanguage(turnContext.Activity.Text);

switch (result.DetectedLanguages[0].Name)
{
    case "English":
        break;
    default:
        //throw error
        await turnContext.SendActivityAsync($"I'm sorry, I can only understand English. [{result.DetectedLanguages[0].Name}]");
        return;
        break;
}
```

1.  Open the **appsettings.json** file and ensure that your cognitive services settings are entered:

```csharp
"cogsBaseUrl": "",
"cogsKey" :  ""
```

1.  Press **F5** to start your bot

1.  Using the Bot Emulator, send in a few phrases and see what happens:

+   Como Estes?
+   Bon Jour!
+   Привет
+   Hello

#### Going further

Since we have already introduced you to LUIS in previous labs, think about what changes you may need to make to support multiple languages using LUIS.  Some helpful articles:

-   [Language and region support for LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-language-support)   

####  Resources

-   [Example: Detect language with Text Analytics](https://docs.microsoft.com/en-us/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-language-detection)
-   [Quickstart: Text analytics client library for .NET](https://docs.microsoft.com/en-us/azure/cognitive-services/text-analytics/quickstarts/csharp)


## Delete the resources
1. Switch to Azure Portal window and goto resource group
2. Delete all resources

#### Next Steps
-   [Lab8 - Test Bots In DirectLine](/lab/2f7669b0-6be6-451a-a05f-c9f14d869388)

***Congratulations! You have successfully completed the lab. ***