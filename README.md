# Assistant
This repository contains the code of the tutorial of connecting Rasa-powered assistant to Google Assistant. In this project, I am going to show you how can create custom action and enable your Google Assistant to handle deeper and more natural conversations by integrating it with Rasa Stack.
This project contains the following files:
**place_finder** - a directory which contains a pre-built Rasa assistant called Search Restaurant. This assistant is used in this tutorial to demonstrate the integration to Google Assistant.
**action.json** - a custom Google Assistant action configuration file.
**ga_connector.py** - a custom Rasa-Google Assistant connector. If you follow the tutorial using your own assistant, add this connector to your project directory.

## Outline
    1. The Rasa AI assistant
    2. Initializing the Google Assistant custom skill
    3. Creating the custom Google Assistant action
    4. Creating the Rasa Connector for Google Assistant
    5. Putting all the pieces together
    
## Building blocks
Here are the tools and software which you will need to create a Rasa-powered Google Assistant:
    • Google Assistant smart speaker or a Google Assistant app — a tool to test the custom Google Assistant skill
    • Google Actions console — the platform for initializing custom Google Assistant actions
    • Google Actions SDK — a software pack designed to build custom Google Assistant skills
    • Rasa NLU and Rasa Core — open source conversational AI software stack
    • Ngrok — a tunnel to expose a locally running application to the outside world

You can find the step-by-step tutorial here.

## step 1 : Rasa custom action
  This project consists of the following files:

**data/nlu_data.md** is a file which contains the training data for Rasa NLU model. This data consists of the example user queries alongside the corresponding intents and entities.
**config.yml** file contains the configuration of the Rasa NLU training pipeline. It defines how the user inputs will be parsed, how the features will be extracted and what machine learning model will be used to train the model.
**data/stories.md** is a file which contains training data for the Rasa Core model. This data consists of stories — actual conversations between a user and an assistant written in a Rasa format, where user inputs are expressed as corresponding intents and entities, and the responses of the assistant are expressed as actions.
**domain.yml** is a file which contains the configuration of the assistant’s domain. It consists of the following pieces:
intents and entities that are defined in Rasa NLU training data examples;
slots which work like placeholders for saving the details that the assistant has to remember throughout the conversation;
templates which define the text responses that an assistant should return when the corresponding utterances are predicted
actions which the assistant should be able to predict and execute.
**actions.py** is a file which contains the code of the custom action where the assistant makes an API call to retrieve the details about the user’s current location and search for the specified place within the user requested radius. The assistant then uses the returned details to generate the response and stores some of the retrieved details as slots for the assistant to use in the later stages of the conversation.
**endpoints.yml** contains the configuration of custom actions webhook.
**credentials.yml** — a file to store your Google Places API key.

All these files are ready for you to use, all you have to do is provide your Google Places API key to credentials.yml file and train the models. Train the NLU model using the command below which will call Rasa NLU train function, pass training data and processing pipeline configuration files, and save the model inside the models/current/nlu_model directory:

python -m rasa_nlu.train -c config.yml --data data/nlu_data.md -o models --fixed_model_name nlu_model --project current --verbose

Once the NLU model is trained, train the Rasa Core model using the command below which will call Rasa Core train function, pass stories data and domain files and save the trained model inside the models/current/dialogue directory:

python -m rasa_core.train -d domain.yml -s data/stories.md -o models/current/dialogue --epochs 100

By training Rasa NLU and Rasa Core models you create the brain of your AI assistant.

## Step 2: Initializing the Google Assistant custom action

The custom Google Assistant action will package your Rasa-powered agent as a custom skill and will enable you to use Google Assistant’s speech-to-text service to enable the voice communication with your assistant.First, you have to initialize the Google Assistant custom action. Here is how you can do it:

1. Go to Google actions console and select ‘Add/import project’
2. Set the name of your project and pick the language you want your assistant to use. Click ‘Create project’.
3. Once the project is created, you will be prompted to a page where you can choose the category of your project. Scroll down to the bottom of the page and choose ‘Actions SDK’ option. By doing that, you tell your Google Assistant that you will implement your own custom action with a custom NLP and dialogue management system.
4. Inside the window which was opened after selecting Actions SDK you will find Google Actions SDK guide and a gactions function which you will use later to deploy the custom action. Copy the provided function or take note of the — project parameter value and click OK.
5. Now you can pick a project category or skip this step by selecting ‘Skip’ at the top of the page.
6. In the project configurations page, you can specify how you want your custom skill to be invoked. You can do it by clicking on ‘Decide how your Action is invoked’ on a ‘Quick setup’ pane.
7. Here you can select the Display name — the name which Google Assistant looks for to start your custom skill. For example, if you set the display name to ‘Search Restaurant’, then to start this custom skill you will have to say something like ‘Hey Google, talk to a Search Restaurant’. You can also choose the voice that you would like your Google Assistant to use and once you are happy with the configurations select ‘Save’.

## Step 3: Creating a custom Google Assistant action

Custom Google Assistant action will be responsible for only two things — understanding that a user invoked the custom action and passing all user inputs to Rasa after the skill is invoked. Google Assistant will have to do once intents are matched after performing speech-to-text on user voice inputs.
Here is how to do it:
1. Download Google gactions CLI and place it inside your project directory.
2. Inside that directory, initialize Google Assistant custom action configuration file by running the following command:
gactions init
3. Open the actions.json file which was initialized after running the command above. This file has the bare bones configuration of the custom skill invocation action. It consists of three main parts:
name — it defines the type of the intent
intent — corresponds to what the action is about
fulfillment — a webhook which the Google Assistant should send the request to once the corresponding intent is matched

All custom Google Assistant skills need an invocation action, so all you have to do is to provide the necessary details to the configuration above:

- conversationName which will work as a connector between the intent and a corresponding fulfillment. For the skill invocation action, a conversationName could be something like ‘welcome’
- queryPatterns — an example phrase that a user might say. For the skill invocation action, a query pattern could be something like ‘Talk to Search Restaurant’.
- 
Once the invocation action is configured, all you have to add is one another action which will pass all user inputs to Rasa Stack after the custom skill is invoked. For this, you will have to define a new intent with the name ‘TEXT’ which will capture all user inputs after speech-to-text service is performed and pass it to Rasa Stack using the webhook which you will define in the next step.
In order to pass user inputs to Rasa, Google Assistant needs a webhook url which will be used to establish the connection between the two. In the next step, you will write a custom connector to connect Rasa to Google Assistant.

## Step 4: Creating Rasa Connector for Google Assistant

By default, Rasa comes with a bunch of prebuilt connectors to the most popular messaging platforms, but since Google Assistant connector is not yet included, you will learn how to create one yourself! Here are the steps how to do it:

1. In your project directory create a new file called ‘ga_connector.py’. In this file, you will write the code.
Now it’s time to put all the pieces together and test the assistant.

## Step 5: Putting all the pieces together

All that is left to do is to start the Rasa Core server which will use the custom connector to connect to Google Assistant. You can do it by using **run_app.py** which will load the Rasa Core agent using Rasa NLU model as an interpreter and will star the webhook which will listen for incoming user inputs:
It will start the server on the port 5004 and establish the GoogleConnector webhook.
Since your Rasa assistant runs locally and Google Assistant runs on the cloud, you will need a tunneling service to expose your locally running Rasa assistant to the outside world. Ngrok is a good choice for that, so start ngrok on the same port as the Rasa agent is running on — 5004. Now you can connect it to Google Assistant by passing your webhook URL to fulfilments of your Google Assistant custom action. To do that, copy the url generated by ngrok (it will be different than the one we got for the example below), attach the ‘/webhooks/google_assistant/webhook’ suffix to it, go back to action.json file and provide the url to the conversations part of the configuration file:

If your Rasa assistant includes custom actions (just like Search Restaurant does), make sure to start the actions server by running the following command:

python -m rasa_core_sdk.endpoint --actions actions

Now, all that is left to do is to deploy your custom Google Assistant skill and enable testing. You can deploy the custom Google Assistant skill by running the command below. Here, PROJECT_ID is a value of the — project parameter which you took the note of in step 2 of this tutorial.

gactions update --action_package action.json --project PROJECT_ID

Once the action is uploaded, enable testing of the action by running the following command:

gactions test --action_package action.json --project PROJECT_ID

Tip: if you forgot to take note of your project id, you can find it inside the projects settings on Google Actions console:

**Congratulations! You have just built a custom Google Assistant skill with Rasa Stack!**

Now you can go ahead and test it on your Google Home smart speaker or your Google Assistant app (make sure to be logged in with the same Google account you used to develop the custom skill on Google Actions console)!

**Let us know how you are getting on!**

If you have any question post your question on Rasa Community Forum.

**Resources**

- Full code of this tutorial
- Rasa website
- Rasa Community Forum
- Rasa Stack docs
- Rasa GitHub repositories
