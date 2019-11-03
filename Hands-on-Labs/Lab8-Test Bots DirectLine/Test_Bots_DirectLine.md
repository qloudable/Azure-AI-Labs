# Implement Computer Vision
  
## Table of Contents

[Overview](#overview)

[Pre-Requisites](#pre-requisites)

[Publish Your Bot](#publish-your-bot)

[Setting up the Direct Line channel](#setting-up-the-direct-line-channel)

[Create a console application](#create-a-console-application)

[Using HTTP Get to retrieve messages](#using-http-get-to-retrieve-messages)

[Delete the resources](#delete-the-resources)


## Overview
Communication directly with your bot may be required in some situations. For example, you may want to perform functional tests with a hosted bot. Communication between your bot and your own client application can be performed using the [Direct Line API](https://docs.microsoft.com/en-us/bot-framework/rest-api/bot-framework-rest-direct-line-3-0-concepts).

Additionally, sometimes you may want to use other channels or customize your bot further. In that case, you can use Direct Line to communicate with your custom bots.

Microsoft Bot Framework Direct Line bots are bots that can function with a custom client of your own design. Direct Line bots are similar to normal bots. They just don't need to use the provided channels. Direct Line clients can be written to be whatever you want them to be. You can write an Android client, an iOS client, or even a console-based client application.

This hands-on lab introduces key concepts related to Direct Line API.

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

This lab starts from the assumption that you have built and published the bot from "Lab2 - Create An Intelligent Chat Bot". It is recommended that you do that lab in order to be successful in the ones that follow. If you have not, reading carefully through all the exercises and looking at some of the code or using it in your own applications may be sufficient, depending on your needs.

We'll also assume that you've completed "Lab3 - Log Chat Bot Conversations", but you should be able to complete the labs without completing the logging labs.


## Publish Your Bot

#### Collecting the Keys

Over the course of the last few labs, we have collected various keys. You will need most of them if you are using the starter project as a starting point.

>_Keys_
>
>- Cognitive Services API Url:
>- Cognitive Services API key:
>- LUIS API Endpoint:
>- LUIS API Key:
>- LUIS API App ID:
>- Bot App Name:
>- Bot App ID:
>- Bot App Password:
>- Azure Storage Connection String:
>- Cosmos DB Url:
>- Cosmos DB Key:
>- DirectLine Key:

1.  Open your **PictureBot** solution

1.  Right-click the project and select **Publish**

1.  In the publish dialog, select **Select Existing**, then click **Publish**

1.  If prompted, login using the account you have used through the labs

1.  Select the subscription you have been using

1.  Expand the resource group and select the picture bot app service we created in Lab 3

1.  Click **OK**

> **Note** Depending on the path you took to get to this lab, you may need to publish a second time otherwise you may get the echo bot service otherwise when you test below.  Republish the bot, only this time change the publish settings to remove existing files.  

## Setting up the Direct Line channel

1.  In the portal, locate your published PictureBot **web app bot** and navigate to the **Channels** tab. 

1.  Select the Direct Line icon (looks like a globe). You should see a **Default Site** displayed. 

1.  For the **Secret keys**, click **Show** and store one of the secret keys in Notepad, or wherever you are keeping track of your keys. 

You can read detailed instructions on [enabling the Direct Line channel](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-connect-directline?view=azure-bot-service-4.0) and [secrets and tokens](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-3.0#secrets-and-tokens).

## Create a console application

We'll create a console application to help us understand how Direct Line can allow us to connect directly to a bot. The console client application we will create operates in two threads. The primary thread accepts user input and sends messages to the bot. The secondary thread polls the bot once per second to retrieve any messages from the bot, then displays the messages received.

> **Note** The instructions and code here have been modified from the [best practices in the documentation](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-direct-line?view=azure-bot-service-4.0&tabs=cscreatebot%2Ccsclientapp%2Ccsrunclient).

1.  If not already open, open your **PictureBot** solution in Visual Studio. 

1.  Right-click on the solution in Solution Explorer, then select **Add > New Project**. 

1.  Search for **Console App (.NET Core)**, select it and click **Next**

1.  For the name, type **PictureBotDL**

1.  Select **Create**.

**Step 2** - Add NuGet packages to PictureBotDL

1.  Right-click on the PictureBotDL project and select **Manage NuGet Packages**. 

1.  Within the **Browse** tab , search and install/update the following:

+   Microsoft.Bot.Connector.DirectLine
+   Microsoft.Rest.ClientRuntime

1.  Open **Program.cs**

1.  Replace the contents of **Program.cs** (within PictureBotDL) with the following:

```csharp
using System;
using System.Diagnostics;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Bot.Connector.DirectLine;
using Newtonsoft.Json;
using Activity = Microsoft.Bot.Connector.DirectLine.Activity;

namespace PictureBotDL
{
    class Program
    {
        // ************
        // Replace the following values with your Direct Line secret and the name of your bot resource ID.
        //*************
        private static string directLineSecret = "YourDLSecret";
        private static string botId = "YourBotServiceName";

        // This gives a name to the bot user.
        private static string fromUser = "PictureBotSampleUser";

        static void Main(string[] args)
        {
            StartBotConversation().Wait();
        }


        /// <summary>
        /// Drives the user's conversation with the bot.
        /// </summary>
        /// <returns></returns>
        private static async Task StartBotConversation()
        {
            // Create a new Direct Line client.
            DirectLineClient client = new DirectLineClient(directLineSecret);

            // Start the conversation.
            var conversation = await client.Conversations.StartConversationAsync();

            // Start the bot message reader in a separate thread.
            new System.Threading.Thread(async () => await ReadBotMessagesAsync(client, conversation.ConversationId)).Start();

            // Prompt the user to start talking to the bot.
            Console.Write("Conversation ID: " + conversation.ConversationId + Environment.NewLine);
            Console.Write("Type your message (or \"exit\" to end): ");

            // Loop until the user chooses to exit this loop.
            while (true)
            {
                // Accept the input from the user.
                string input = Console.ReadLine().Trim();

                // Check to see if the user wants to exit.
                if (input.ToLower() == "exit")
                {
                    // Exit the app if the user requests it.
                    break;
                }
                else
                {
                    if (input.Length > 0)
                    {
                        // Create a message activity with the text the user entered.
                        Activity userMessage = new Activity
                        {
                            From = new ChannelAccount(fromUser),
                            Text = input,
                            Type = ActivityTypes.Message
                        };

                        // Send the message activity to the bot.
                        await client.Conversations.PostActivityAsync(conversation.ConversationId, userMessage);
                    }
                }
            }
        }


        /// <summary>
        /// Polls the bot continuously and retrieves messages sent by the bot to the client.
        /// </summary>
        /// <param name="client">The Direct Line client.</param>
        /// <param name="conversationId">The conversation ID.</param>
        /// <returns></returns>
        private static async Task ReadBotMessagesAsync(DirectLineClient client, string conversationId)
        {
            string watermark = null;

            // Poll the bot for replies once per second.
            while (true)
            {
                // Retrieve the activity set from the bot.
                var activitySet = await client.Conversations.GetActivitiesAsync(conversationId, watermark);
                watermark = activitySet?.Watermark;

                // Extract the activies sent from our bot.
                var activities = from x in activitySet.Activities
                                 where x.From.Id == botId
                                 select x;

                // Analyze each activity in the activity set.
                foreach (Activity activity in activities)
                {
                    // Display the text of the activity.
                    Console.WriteLine(activity.Text);

                    // Are there any attachments?
                    if (activity.Attachments != null)
                    {
                        // Extract each attachment from the activity.
                        foreach (Attachment attachment in activity.Attachments)
                        {
                            switch (attachment.ContentType)
                            {
                                // Display a hero card.
                                case "application/vnd.microsoft.card.hero":
                                    RenderHeroCard(attachment);
                                    break;

                                // Display the image in a browser.
                                case "image/png":
                                    Console.WriteLine($"Opening the requested image '{attachment.ContentUrl}'");
                                    Process.Start(attachment.ContentUrl);
                                    break;
                            }
                        }
                    }

                }

                // Wait for one second before polling the bot again.
                await Task.Delay(TimeSpan.FromSeconds(1)).ConfigureAwait(false);
            }
        }


        /// <summary>
        /// Displays the hero card on the console.
        /// </summary>
        /// <param name="attachment">The attachment that contains the hero card.</param>
        private static void RenderHeroCard(Attachment attachment)
        {
            const int Width = 70;
            // Function to center a string between asterisks.
            Func<string, string> contentLine = (content) => string.Format($"{{0, -{Width}}}", string.Format("{0," + ((Width + content.Length) / 2).ToString() + "}", content));

            // Extract the hero card data.
            var heroCard = JsonConvert.DeserializeObject<HeroCard>(attachment.Content.ToString());

            // Display the hero card.
            if (heroCard != null)
            {
                Console.WriteLine("/{0}", new string('*', Width + 1));
                Console.WriteLine("*{0}*", contentLine(heroCard.Title));
                Console.WriteLine("*{0}*", new string(' ', Width));
                Console.WriteLine("*{0}*", contentLine(heroCard.Text));
                Console.WriteLine("{0}/", new string('*', Width + 1));
            }
        }
    }
}
```
> **Note** this code was slightly modified from [the documentation](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-direct-line?view=azure-bot-service-4.0&tabs=cscreatebot%2Ccsclientapp%2Ccsrunclient#create-the-console-client-app) to include a few things we'll use in the next lab sections.

1.  In **Program.cs**, update the direct line secret and bot id to your specific values.

Spend some time reviewing this sample code. It's a good exercise to make sure you understand how we're connecting to our PictureBot and getting responses.

**Step 4** - Run the app

1.  Right-click on the **PictureBotDL** project and select **Set as Startup Project**. 

1.  Next, press **F5** to run the app 

1.  Have a conversation with the bot using the commandline application

![Console App](../images//consoleapp.png)

> **Note**  If you do not get a response, it is likely you have an error in your bot.  Test your bot locally using the Bot Emulator, fix any issues, then republish it.

Quick quiz - how are we displaying the Conversation ID? We'll see in the next sections why we might want this.

## Using HTTP Get to retrieve messages

Because we have the conversation ID, we can retrieve user and bot messages using HTTP Get. If you are familiar and experienced with Rest Clients, feel free to use your tool of choice.

In this lab, we will go through using Postman (a web-based client) to retrieve messages.

**Step 1** - Download Postman

1.  [Download the native app for your platform](https://www.getpostman.com/apps). You may need to create a simple account.

**Step 2**

Using Direct Line API, a client can send messages to your bot by issuing HTTP Post requests. A client can receive messages from your bot either via WebSocket stream or by issuing HTTP GET requests. In this lab, we will explore HTTP Get option to receive messages.

We'll be making a GET request. We also need to make sure the header contains our header name (**Authorization**) and header value (**Bearer YourDirectLineSecret**). Addtionally, we'll make the call to our existing conversation in the console app, by replaceing {conversationId} with our current Conversation ID in our request: `https://directline.botframework.com/api/conversations/{conversationId}/messages`.

Postman makes this very easy for us to configure:

- Change the request to type "GET" using the drop down.
- Enter your request URL with your Conversation ID.
- Change the Authorization to type "Bearer Token" and enter your Direct Line Secret key in the "Token" box.

![Bearer Token](../images//bearer.png)

1.  Open **Postman**

1.  For the type, ensure that **GET** is selected

1.  For the url, type **https://directline.botframework.com/api/conversations/{conversationId}/messages**.  Be sure to replace the converstationId with your specific converstation id

1.  Click **Authorization**, for the type, select **Bearer Token**

1.  Set the value to **{Your Direct Line Secret}**

1.  Finally, select **Send**. 

1.  Inspect your results. 

1.  Create a new conversation with your console app and be sure to search for pictures. 

1.  Create a new request using Postman using the new converstation id. 

1.  Inspect the response returned.  You should find the image url displayed within the images array of the response

![Images Array Example](../images//imagesarray.png)

#### Going further

Have extra time? Can you leverage curl (download link: https://curl.haxx.se/download.html) from the terminal to retrieve conversations (like you did for Postman)?

> Hint: your command may look similar to `curl -H "Authorization:Bearer {SecretKey}" https://directline.botframework.com/api/conversations/{conversationId}/messages -XGET`

####  Resources

-   [Direct Line API](https://docs.microsoft.com/en-us/bot-framework/rest-api/bot-framework-rest-direct-line-3-0-concepts)

## Delete the resources
1. Switch to Azure Portal window and goto resource group
2. Delete all resources

***Congratulations! You have successfully completed the lab. ***