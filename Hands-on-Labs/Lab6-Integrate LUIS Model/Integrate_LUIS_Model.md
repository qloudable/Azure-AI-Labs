# Implement Computer Vision
  
## Table of Contents

[Overview](#overview)

[Pre-Requisites](#pre-requisites)

[Adding natural language understanding](#adding-natural-language-understanding)

[Adding LUIS to PictureBot MainDialog](#adding-luis-to-picturebot-maindialog)

[Testing natural speech phrases](#testing-natural-speech-phrases)

[Delete the resources](#delete-the-resources)

## Overview
Our bot is now capable of taking in a user's input and responding based on the user's input. Unfortunately, our bot's communication skills are brittle. One typo, or a rephrasing of words, and the bot will not understand. This can cause frustration for the user. We can greatly increase the bot's conversational abilities by enabling it to understand natural language with the LUIS model we built in "Lab5 - Implement LUIS Model".

We will have to update our bot in order to use LUIS.  We can do this by modifying "Startup.cs" and "PictureBot.cs."

**Note:** This lab is meant for an Artificial Intelligence (AI) Engineer or an AI Developer on Azure. 

**Architecture:**
<img src="https://raw.githubusercontent.com/qloudable/Azure-AI-Labs/master/images/AI_Immersion_Arch.png" alt="image-alt-text" width="100%">

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

This lab builds on "Lab2 - Create An Intelligent Chat Bot". It is recommended that you do that lab in order to be able to implement logging as covered in this lab. If you have not, reading carefully through all the exercises and looking at some of the code or using it in your own applications may be sufficient, depending on your needs.

>NOTE: If you intend to use the code in the Finished folder, you MUST replace the app specific information with your own app IDs and endpoints.

## Adding natural language understanding

#### Adding LUIS to Startup.cs

1.  If not already open, open your **PictureBot** solution in Visual Studio

> **NOTE** You can also start with the **{GitHubPath}/Lab7-Integrate_LUIS/code/Starter/PictureBot/PictureBot.sln** solution if you did not start from Lab 1.
> Be sure to replace all the appsettings values

1.  Open **Startup.cs** and locate the `ConfigureServices` method. We'll add LUIS here by adding an additional service for LUIS after creating and registering the state accessors.

Below:

```csharp
services.AddSingleton((Func<IServiceProvider, PictureBotAccessors>)(sp =>
{
    .
    .
    .
    return accessors;
});
```

Add:

```csharp
// Create and register a LUIS recognizer.
services.AddSingleton(sp =>
{
    var luisAppId = Configuration.GetSection("luisAppId")?.Value;
    var luisAppKey = Configuration.GetSection("luisAppKey")?.Value;
    var luisEndPoint = Configuration.GetSection("luisEndPoint")?.Value;

    // Get LUIS information
    var luisApp = new LuisApplication(luisAppId, luisAppKey, luisEndPoint);

    // Specify LUIS options. These may vary for your bot.
    var luisPredictionOptions = new LuisPredictionOptions
    {
        IncludeAllIntents = true,
    };

    // Create the recognizer
    var recognizer = new LuisRecognizer(luisApp, luisPredictionOptions, true, null);
    return recognizer;
});
```

1.  Modify the **appsettings.json** to include the following properties, be sure to fill them in with your LUIS instance values:

```json
"luisAppId": "",
"luisAppKey": "",
"luisEndPoint": ""
```

> **Note** The Luis endpoint url for the .NET SDK should be something like **https://{region}.api.cognitive.microsoft.com** with no api or version after it.

## Adding LUIS to PictureBot MainDialog

1.  Open **PictureBot.cs.**. The first thing you'll need to do is initialize the LUIS recognizer, similar to how you did for `PictureBotAccessors`. Below the commented line `// Initialize LUIS Recognizer`, add the following:

```csharp
private LuisRecognizer _recognizer { get; } = null;
```

1.  Navigate to the **PictureBot** constructor:

```csharp
public PictureBot(PictureBotAccessors accessors, ILoggerFactory loggerFactory /*, LuisRecognizer recognizer*/)
```

Now, maybe you noticed we had this commented out in your previous labs, maybe you didn't. You have it commented out now, because up until now, you weren't calling LUIS, so a LUIS recognizer didn't need to be an input to PictureBot. Now, we are using the recognizer.

1.  Uncomment the input requirement (parameter `LuisRecognizer recognizer`), and add the following line below `// Add instance of LUIS Recognizer`:

```csharp
_recognizer = recognizer ?? throw new ArgumentNullException(nameof(recognizer));
```

Again, this should look very similar to how we initialized the instance of `_accessors`.

As far as updating our `MainDialog` goes, there's no need for us to add anything to the initial `GreetingAsync` step, because regardless of user input, we want to greet the user when the conversation starts.

1.  In `MainMenuAsync`, we do want to start by trying Regex, so we'll leave most of that. However, if Regex doesn't find an intent, we want the `default` action to be different. That's when we want to call LUIS.

Within the `MainMenuAsync` switch block, replace:

```csharp
default:
    {
        await MainResponses.ReplyWithConfused(stepContext.Context);
        return await stepContext.EndDialogAsync();
    }
```

With:

```csharp
default:
{
    // Call LUIS recognizer
    var result = await _recognizer.RecognizeAsync(stepContext.Context, cancellationToken);
    // Get the top intent from the results
    var topIntent = result?.GetTopScoringIntent();
    // Based on the intent, switch the conversation, similar concept as with Regex above
    switch ((topIntent != null) ? topIntent.Value.intent : null)
    {
        case null:
            // Add app logic when there is no result.
            await MainResponses.ReplyWithConfused(stepContext.Context);
            break;
        case "None":
            await MainResponses.ReplyWithConfused(stepContext.Context);
            // with each statement, we're adding the LuisScore, purely to test, so we know whether LUIS was called or not
            await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
            break;
        case "Greeting":
            await MainResponses.ReplyWithGreeting(stepContext.Context);
            await MainResponses.ReplyWithHelp(stepContext.Context);
            await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
            break;
        case "OrderPic":
            await MainResponses.ReplyWithOrderConfirmation(stepContext.Context);
            await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
            break;
        case "SharePic":
            await MainResponses.ReplyWithShareConfirmation(stepContext.Context);
            await MainResponses.ReplyWithLuisScore(stepContext.Context, topIntent.Value.intent, topIntent.Value.score);
            break;
        default:
            await MainResponses.ReplyWithConfused(stepContext.Context);
            break;
    }
    return await stepContext.EndDialogAsync();
}
```

Let's briefly go through what we're doing in the new code additions. First, instead of responding saying we don't understand, we're going to call LUIS. So we call LUIS using the LUIS Recognizer, and we store the Top Intent in a variable. We then use `switch` to respond in different ways, depending on which intent is picked up. This is almost identical to what we did with Regex.

> **Note** If you named your intents differently in LUIS than instructed in the dode accompanying [Lab 6](../Lab6-Implement_LUIS/02-Implement_LUIS.md), you need to modify the `case` statements accordingly.

Another thing to note is that after every response that called LUIS, we're adding the LUIS intent value and score. The reason is just to show you when LUIS is being called as opposed to Regex (you would remove these responses from the final product, but it's a good indicator for us as we test the bot).

## Testing natural speech phrases

1.  Press **F5** to run the app. 

1.  Switch to your Bot Emulator. Try sending the bots different ways of searching pictures. What happens when you say "send me pictures of water" or "show me dog pics"? Try some other ways of asking for, sharing and ordering pictures.

If you have extra time, see if there are things LUIS isn't picking up on that you expected it to. Maybe now is a good time to go to luis.ai, [review your endpoint utterances](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/label-suggested-utterances), and retrain/republish your model.

> **Fun Aside**: Reviewing the endpoint utterances can be extremely powerful.  LUIS makes smart decisions about which utterances to surface.  It chooses the ones that will help it improve the most to have manually labeled by a human-in-the-loop.  For example, if the LUIS model predicted that a given utterance mapped to Intent1 with 47% confidence and predicted that it mapped to Intent2 with 48% confidence, that is a strong candidate to surface to a human to manually map, since the model is very close between two intents.

#### Going further

If you're having trouble customizing your LUIS implementation, review the documentation guidance for adding LUIS to bots [here](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-v4-luis?view=azure-bot-service-4.0&tabs=cs).

>Get stuck or broken? You can find the solution for the lab up until this point under [code/Finished](./code/Finished). You will need to insert the keys for your Azure Bot Service in the `appsettings.json` file. We recommend using this code as a reference, not as a solution to run, but if you choose to run it, be sure to add the necessary keys (in this section, there shouldn't be any).

**Extra Credit**

If you wish to attempt to integrate LUIS bot including Azure Search, building on the prior supplementary LUIS model-with-search [training] (https://github.com/Azure/LearnAI-Bootcamp/tree/master/lab01.5-luis), follow the following trainings: [Azure Search](https://github.com/Azure/LearnAI-Bootcamp/tree/master/lab02.1-azure_search), and [Azure Search Bots](https://github.com/Azure/LearnAI-Bootcamp/blob/master/lab02.2-building_bots/2_Azure_Search.md).


## Delete the resources
1. Switch to Azure Portal window and goto resource group
2. Delete all resources

#### Next Steps
-   [Lab7 - Detect User Language in Chat Bot](/lab/dbeefcdd-ecd9-4f3e-b8a5-9fdfc0b0b3c9)

***Congratulations! You have successfully completed the lab. ***