# Implement Computer Vision
  
## Table of Contents

[Overview](#overview)

[Pre-Requisites](#pre-requisites)

[QnA Maker Setup](#qna-maker-setup)

[Create a KnowledgeBase](#create-a-knowledgebase)

[Publish and Test your Knowledge base](#publish-and-test-your-knowledge-base)

[Download the Bot Source code-optional)](#download-the-bot-source-code-optional)

[Delete the resources](#delete-the-resources)


## Overview
In this lab we will explore the QnA Maker for creating bots that connect to a pre-trained knowledgebase. Using QnAMaker, you can upload documents or point to web pages and have it pre-populate a knowledge base that can feed a simple bot for common uses such as frequently asked questions.

**Note:** This lab is meant for an Artificial Intelligence (AI) Engineer or an AI Developer on Azure. 

**Architecture:**
<img src="https://raw.githubusercontent.com/qloudable/Azure-AI-Labs/master/images/AI_Immersion_Arch.png" alt="image-alt-text">

In the continuation of this lab throughout the course, we'll show you how to query your data, and then build a Bot Framework bot to query it. Finally, we'll extend this bot with LUIS to automatically derive intent from your queries and use those to direct your searches intelligently.

## Pre-Requisites

1. Azure Portal Credentials (User, Password, Tenant Id, Subscription Id, and Resource Group)  
  **Note:** These details Will be provided to you while taking the lab. 

3. Familiarity with Azure Portal Console: https://docs.microsoft.com/en-us/azure/azure-portal/

3. Exposure to Visual Studio: https://docs.microsoft.com/en-us/visualstudio/get-started/visual-studio-ide?view=vs-2019

4. Familiarity with C# (intermediate level): https://docs.microsoft.com/en-us/dotnet/csharp/tutorials/intro-to-csharp/

##  QnA Maker Setup
Use following credentials to login into Azure Portal
Username: {{User Name}}
Password: {{Password}}

**Note** Tenant Id, Subscription Id, and Resource Group can be found under Access Details. 

1.  Open the [Azure Portal](https://portal.azure.com)

1.  Click **Add a new resource**

1.  Type **QnA Maker**, select **QnA Maker**

1.  Click **Create**

1.  Type a name

1.  Select the **S0** tier for the resource pricing tier.  We aren't using the free tier because we will upload files that are larger than 1MB later.

1.  Select your resource group

1.  For the search pricing tier, select the **F** tier

1.  Enter an appname, it must be unique

1.  Click **Create**.  This will created the following resources in your resource group:

-   App Service Plan
-   App Service
-   Application Insights
-   Search Service
-   Cognitive Service instance of type QnAMaker

## Create a KnowledgeBase

1.  Open the [QnA Maker site](https://qnamaker.ai)

1.  In the top right, click **Sign in**.  Login using the Azure credentials for your new QnA Maker resource

1.  In the top navigation area, click **Create a knowledge base**

1.  Skip **STEP 1** as you have already created the resource

1.  Select your Azure AD and Subscription tied to your QnA Maker resource, then select your newly created QnA Maker resource

1.  For the name, type **Microsoft FAQs**

1.  For the URL, enter below url and click **Add URL**

```
https://raw.githubusercontent.com/qloudable/Azure-AI-Labs/master/Hands-on-Labs/Lab4-Implement%20QnA%20Maker/media/Manage%20Azure%20Blob%20Storage.docx

```

1.  For the URL, enter below url and click **Add URL**

```
https://raw.githubusercontent.com/qloudable/Azure-AI-Labs/master/Hands-on-Labs/Lab4-Implement%20QnA%20Maker/media/surface-pro-4-user-guide-EN.pdf

```

**Note** You can find out more about the supported file types and data sources [here](https://docs.microsoft.com/en-us/azure/cognitive-services/QnAMaker/concepts/data-sources-supported)

1.  For the **Chit-chat**, select **Witty**

1.  Click **Create your KB**

## Publish and Test your Knowledge base

1.  Review the knowledge base QnA pairs, you should see ~200 different pairs based on the two documents we fed it

1.  In the top menu, click **PUBLISH**.  

1.  On the publish page, click **Publish**.  Once the service is published, click the **Create Bot** button on the success page
![Success Page](https://raw.githubusercontent.com/qloudable/Azure-AI-Labs/master/Hands-on-Labs/Lab4-Implement%20QnA%20Maker/media/successpage.png)

1.  If prompted, login as the account tied to your lab resource group.

1.  On the bot service creation page, fix any naming errors, then click **Create**.

**Note**  Recent change in Azure requires dashes ("-") be removed from some resource names

1.  Once the bot resource is created, navigate to the new **Web App Bot**, then click **Test in Web Chat**
![Powershell App](https://raw.githubusercontent.com/qloudable/Azure-AI-Labs/master/Hands-on-Labs/Lab4-Implement%20QnA%20Maker/media/success2.png)

1.  Ask the bot any questions related to a Surface Pro 4 or managing Azure Blog Storage:

+   How do I add memory?
+   How long does it take to charge the battery?
+   How do I hard reset?
+   What is a blob?

1.  Ask it some questions it doesn't know, such as:

+   How do I bowl a strike?

## Download the Bot Source code-optional

1.  Under **Bot management**, select the **Build** tab

1.  Click **Download Bot source code**, when prompted click **Yes**.  

1.  Azure will build your source code, when complete, click **Download Bot source code**, if prompted, select **Yes**

1.  Open powershell from app menu and go to Download folder using command:

```
cd c:\downalods

```

1.  Unzip the downloaded file using following command

```powershell

Expand-Archive -LiteralPath {downloaded-file}.Zip -DestinationPath {downloaded-file}

```

1.  Navigate to QnABot.sln file uder {downloaded-file}  using cd command in powershell

1. Run below command to open solution in visual studio

```
.\QnABot.sln

```

1.  Open the **Startup.cs** file, you will notice nothing special has been added here

1.  Open the **Bots/{BotName}.cs** file, notice this is where the QnA Maker is being initalized:

```csharp
var qnaMaker = new QnAMaker(new QnAMakerEndpoint
{
    KnowledgeBaseId = _configuration["QnAKnowledgebaseId"],
    EndpointKey = _configuration["QnAAuthKey"],
    Host = GetHostname()
},
null,
httpClient);
```

and then called:

```csharp
var response = await qnaMaker.GetAnswersAsync(turnContext);
if (response != null && response.Length > 0)
{
    await turnContext.SendActivityAsync(MessageFactory.Text(response[0].Answer), cancellationToken);
}
else
{
    await turnContext.SendActivityAsync(MessageFactory.Text("No QnA Maker answers were found."), cancellationToken);
}
```
Run the bot locally by clicking on IIS Express option in toolbar of visual studio and note down the hostname.

To connect locally running bot, open Bot Framework Emulator from App Menu

then click on Open Bot button on welcome screen and enter bot url, App Id, and Password

**Note**: App Id & Password you can find in appsettings.json in the solution (See below picture for reference). URL would be formed using hostname noted before and it should look like "https://localhost:3978/api/messages"

![Setting](https://raw.githubusercontent.com/qloudable/Azure-AI-Labs/master/Hands-on-Labs/Lab4-Implement%20QnA%20Maker/media/settings.png)
 
As you can see, it is very simple to add a generated QnA Maker to your own bots with just a few lines of code.


####  Resources

-   [What is the QnA Maker service?](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/overview/overview)

## Delete the resources
1. Switch to Azure Portal window and goto resource group
2. Delete all resources