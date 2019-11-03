# Implement Computer Vision
  
## Table of Contents

[Overview](#overview)

[Pre-Requisites](#pre-requisites)

[Creating the LUIS service in the portal-optional](#creating-the-luis-service-in-the-portal-optional)

[Adding intelligence to your applications with LUIS](#adding-intelligence-to-your-applications-with-luis)

[Training the LUIS model](#training-the-luis-model)

## Overview
In this lab, you will build, train and publish a LUIS model to help your bot (which will be created in a future lab) communicate effectively with human users.

> **Note** In this lab, we will only be creating the LUIS model that you will use in a future lab to build a more intelligent bot.

The LUIS functionality and functionality has already been covered in the workshop; for a refresher on LUIS, [read more](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/Home).

Now that we know what LUIS is, we'll want to plan our LUIS app. We already created a basic bot ("PictureBot") that responds to messages containing certain text. We will need to create intents that trigger the different actions that our bot can do, and create entities that require different actions. For example, an intent for our PictureBot may be "OrderPic" and it triggers the bot to provide an appropriate response.

For example, in the case of Search (not implemented here), our PictureBot intent may be "SearchPics" and it triggers Azure Search service to look for photos, which requires a "facet" entity to know what to search for.  You can see more examples for planning your app [here](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/plan-your-app).

Once we've thought out our app, we are ready to [build and train it](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-get-started-create-app).

As a review, these are the steps you will generally take when creating LUIS applications:

  1. [Add intents](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/add-intents)
  2. [Add utterances](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/add-example-utterances)
  3. [Add entities](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/add-entities)
  4. [Improve performance using phrase lists](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/add-features) and [patterns](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-model-intent-pattern)
  5. [Train and test](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/train-test)
  6. [Review endpoint utterances](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/label-suggested-utterances)
  7. [Publish](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)

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


## Creating the LUIS service in the portal-optional

Creating a LUIS service in the portal is optional, as LUIS provides you with a "starter key" that you can use for the labs. However, if you want to see how to create a free or paid service in the portal, you can follow the steps below.  

> **Note** If you ran the pre-req ARM Template, you will already have a cognitive services resource that included the Language Understanding APIs.

1.  Open the [Azure Portal](https://portal.azure.com)

1.  Click **Create a resource**

1.  Enter **Language Understanding** in the search box and choose **Language Understanding**

1.  Click **Create**

1.  For the name, type **{YOURINIT}luisbot**

1.  Select your subscripton and and location similar to your resource group

1.  For the pricing tier, select **F0**

1.  Select your resources group

1.  Click **Create**

**Note** The Luis AI web site does not allow you to control or publish your Azure based cognitive services resources.  You will need to call the APIs in order to train and publish them.

## Adding intelligence to your applications with LUIS

Let's look at how we can use LUIS to add some natural language capabilities. LUIS allows you to map natural language utterances (words/phrases/sentences the user might say when talking to the bot) to intents (tasks or actions the user wants to perform). For our application, we might have several intents: finding pictures, sharing pictures, and ordering prints of pictures, for example. We can give a few example utterances as ways to ask for each of these things, and LUIS will map additional new utterances to each intent based on what it has learned.

> **Warning**: Though Azure services use IE as the default browser, we do not recommend it for LUIS. You should be able to use Chrome or Firefox for all of the labs. Alternatively, you can download either [Microsoft Edge](https://www.microsoft.com/en-us/download/details.aspx?id=48126) or [Google Chrome](https://www.google.com/intl/en/chrome/).

1.  Navigate to [https://www.luis.ai](https://www.luis.ai) (**unless you are located in Europe or Australia***). We will create a new LUIS app to support our bot.

> **Note** If you created a key in a **Europe** region, you will need to create your application at [https://eu.luis.ai/](https://eu.luis.ai/). If you created a key in an **Australia** region, you will need to create your application at [https://au.luis.ai/](https://au.luis.ai/). You can read more about the LUIS publishing regions [here](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions).

1.  Sign in using your Microsoft account. This should be the same account that you used to create the LUIS key in the previous section. 

1.  Click **Create a LUIS app now**. You should be redirected to a list of your LUIS applications.  If prompted, click **Migrate Layer**.  

1.  If this is your first time, you will be asked to agree with service terms of use and select your county. 

- On next step you need to chose recommended option and link your Azure account with LUIS. 
- Finally confirm your settings and you will be forwarded to the LUIS App page.

> **Note**: Notice that there is also an "Import App" next to the "New App" button on [the current page](https://www.luis.ai/applications).  After creating your LUIS application, you have the ability to export the entire app as JSON and check it into source control.  This is a recommended best practice, so you can version your LUIS models as you version your code.  An exported LUIS app may be re-imported using that "Import App" button.  If you fall behind during the lab and want to cheat, you can click the "Import App" button and import the [LUIS model](./code/LUIS/PictureBotLuisModel.json).

1.  From the main page, click the **Create new app** button

1.  Type a name, and click **Done**.  Close the dialog.

![LUIS New App](../images//LuisNewApp.png)

1.  In the top navigation, select the **BUILD** link.  Notice there is one intent called "None".  Random utterances that don't map to any of your intents may be mapped to "None".

![LUIS Dashboard](../images//LuisCreateIntent.png)

We want our bot to be able to do the following things:

+ Search/find pictures
+ Share pictures on social media
+ Order prints of pictures
+ Greet the user (although this can also be done other ways, as we will see later)

Let's create intents for the user requesting each of these.  

1.  Click the **Create new intent** button.

1.  Name the first intent **Greeting** and click **Done**.  

1.  Give several examples of things the user might say when greeting the bot, pressing "Enter" after each one.

![LUIS Greeting Intent](../images//LuisGreetingIntent.png)

Let's see how to create an entity.  When the user requests to search the pictures, they may specify what they are looking for.  Let's capture that in an entity.

1.  Click on **Entities** in the left-hand column and then click **Create new entity**.  

1.  Give it an entity name **facet**

1.  For the entity type select ["Simple"](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-concept-entity-types).  

1.  Click **Done**.

![Add Facet Entity](../images//LuisCreateEntity.png)

1.  Click **Intents** in the left-hand sidebar and then click the **Create new intent** button.  

1.  Give it an intent name of **SearchPics** and then click **Done**.

Just as we did for Greetings, let's add some sample utterances (words/phrases/sentences the user might say when talking to the bot).  People might search for pictures in many ways.  Feel free to use some of the utterances below, and add your own wording for how you would ask a bot to search for pictures.

+ Find outdoor pics
+ Are there pictures of a train?
+ Find pictures of food.
+ Search for photos of kids playing
+ Show me beach pics
+ Find dog photos
+ Show me pictures of men wearing glasses
+ Show me happy baby pics

Once we have some utterances, we have to teach LUIS how to pick out the **search topic** as the "facet" entity. Whatever the "facet" entity picks up is what will be searched. 

1.  Hover and click the word (or click consecutive words to select a group of words) and then select the "facet" entity.

![Labelling Entity](../images//LuisFacet.png)

So your utterances may become something like this when facets are labeled:

![Add Facet Entity](../images//SearchPicsIntentAfter.png)

>**Note** This workshop does not include Azure search, however, this functionality has been left in for the sake of demonstration.

1.  Click **Intents** in the left sidebar and add two more intents:

+ Name one intent **"SharePic"**.  This might be identified by utterances like:

+ Share this pic
+ Can you tweet that?
+ Post to Twitter

+ Create another intent named **"OrderPic"**.  This could be communicated with utterances like:

+ Print this picture
+ I would like to order prints
+ Can I get an 8x10 of that one
+ Order wallets

When choosing utterances, it can be helpful to use a combination of questions, commands, and "I would like to..." formats.

1.  Finally add some sample utterances to the "None" intent. This helps LUIS label when things are outside the scope of your application. Add things like "I'm hungry for pizza", "Search videos", etc. You should have about 10-15% of your app's utterances within the None intent.

## Training the LUIS model

We are now ready to train our model.

1.  Click **Train** in the top right bar.  This builds a model to do utterance --> intent mapping with the training data you've provided. Training is not always immediate. Sometimes, it gets queued and can take several minutes.

1.  Click on **MANAGE** in the top bar. You'll have several options on the left side of the window (Application Information, Keys and Endpoints, Publish Settings, Versions, Collaborators). You can read more about the various publish options [here](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/PublishApp).

1.  Select **Azure resources**, you can use the keys here for your LUIS integration later or simply use the keys from the Azure Portal

1.  In the top bar, click **Publish**. You'll have the option to publish to your "Production" or "Staging" endpoint. 

1.  Select **Production**, and [read about the reasons for the two endpoints](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-version). 

1.  Finally, click **Publish**.

![Publish LUIS App](../images//LuisPublish.png)

Publishing creates an endpoint to call the LUIS model.  The URL will be displayed, which will be explained in a later lab. 

1.  Click **Application Information**, copy the **Application ID**.

1.  Click **Azure Resources**, copy the Key and the Endpoint URL.

**Note** We won't use the whole url, you will only need the **https://{region}.api.cognitive.microsoft.com** part

1.  Click **Test** in the top right bar. Try typing a few utterances and see the intents returned. Familiarize yourself with [interactive testing](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/Train-Test#interactive-testing) and [reviewing endpoint utterances](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-review-endoint-utt) as you may want to do this now or in a future lab.

One quick example is shown below. I have noticed that my model incorrectly assigned "send me a swimming photo" as SharePic, when it should be SearchPics. I reassigned the intent.

![Test LUIS](../images//ReassignIntent.png)

Now I need to retrain my app by selecting the Train button. I then tested the same utterance and compared the results between my recently trained and previously published model. Remember, you'll have to republish your model to see updates in the application that uses the model.

![Reassign Intent](../images//ReassignIntentAfter.png)

You can also [test your published endpoint in a browser](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/PublishApp#test-your-published-endpoint-in-a-browser). Copy the Endpoint URL. To open this URL in your browser, set the URL parameter `&q=` to your test query. For example, append `Find pictures of dogs` to your URL, and then press Enter. The browser displays the JSON response of your HTTP endpoint.

#### Going further

If you still have time, spend time exploring the www.luis.ai site. Select "Prebuilt domains" and see [what is already available for you](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-prebuilt-domains). You can also review some of the [other features](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-feature) and [patterns](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-patterns)
, and check out the [BotBuilder-tools](https://github.com/Microsoft/botbuilder-tools) for creating LUIS models, managing LUIS models, simulating conversations, and more. Later, you may also be interested in [another course that includes how to design LUIS schema](https://aka.ms/daaia).

#### Next Steps
-   [Lab6 - Integrate LUIS with Chat Bot](/lab/916fbac5-930c-4276-9829-1082d9b7043e)

***Congratulations! You have successfully completed the lab. ***
