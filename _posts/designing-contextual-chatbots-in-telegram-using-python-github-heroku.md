---
layout: single
title: Coding contextual Telegram chatbots with Python, Github & Heroku
date: 2020-04-10 
description: How to design contextual Telgram chatbots with python
tags: [chatbot, python]
---

Telegram bots are third-party applications that run inside the application. Users can interact with bots by sending them messages, commands and inline requests. Contextual chatbots are a category bots that communicate only through inline requests. They are contextual since they maintain a context and the users can only navigate the pre-defined context. These are especially useful in providing customer service. In this post, we will look at how to design such bots in Telegram using Python.

This will consist of two parts. In the first part, we will register our bot with Telegram, set up our local coding environment and write the code for the bot. In the second part, we will discuss how to host the bot on Heroku which is cloud-based service useful for hosting small web-based applications.

### Registering a new bot with Telegram

To register a new bot go to the Telegram app on your phone and follow these steps:  

1. In the search bar search for the “BotFather”. “BotFather” is a bot that will assist you in creating and managing all your bots  
2. Start BotFather and type /help. This brings up all the possible commands that this bot can handle  
3. Type /newbot or click on this command from this list  
4. Follow the instructions for setting up a new bot which is basically choosing a name and username for your bot. \[Tip: Pick a username which is relevant to what your bot can do\]  
5. Once the bot is successfully created the BotFather will return a token to access HTTP API. Save this token securely with you. We will be using this token to authorize our bot from within the python script

### Setting up a new Python virtual environment

It is good practice to create virtual environments for python projects. A virtual environment helps to keep dependencies required by different projects separate. A virtual environment also helps to easily create a _requirements.txt_ file, which is used by Heroku to create its python environment in the cloud.

While there are few choices for virtual environment tools, we will be using virtualenv for our project. Follow these steps to create a Python virtual environment using virtualenv

* open your terminal in macOS or Linux and create a new project directory `mkdir telegram-bot`
* cd into the directory `cd telegram-bot`
* check that virtualenv is installed in your system by typing `virtualenv -v`. If virtualenv is installed this command will return the installation path. If it is not installed you can install it using the methods mentioned in this [link](https://virtualenv.pypa.io/en/latest/installation.html)

* create a new virtual environment and activate it

```shell
virtualenv venv
source venv/bin/activate
```

While you can use any name for your environment, venv is a convention that is commonly used. Once the environment is active proceed to install the required python packages for the project

We will be using the [python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot) library for this project. This library provides a pure Python interface for the [Telegram Bot API](https://core.telegram.org/bots/api). 

Install it by running `python -m pip install -U python-telegram-bot`

### Initialising a Git repository in the project folder

A git repository needs to be initialised in the project folder as Heroku uses a local git repository to deploy the code in their cloud service. While typically the Heroku remote is added to the local git repository to push the commit, for this project we will add a remote from Github, and further connect the Github repository to the Heroku app.

To initialise a local git repository, in your project folder run `git init`

Also, create a .gitignore file by running the command `touch .gitignore`
The .gitignore file lists the name of those files that should be ignored from all commits which will be updated later.

Now we will be creating a new repository in Github. Visit your Github account and create a new repository. You can either make the repository public or private. We will assume that the name of Github repository you have created is the same as your project folder. Once the repository is created copy its URL.

Now we need to connect our local repository with our newly created Github repository.
Connect your local repository to the newly created Github repo by running the command `git remote add origin URL-to-your-Github-repository`

Once the connection to the remote repository has been established run `git pull origin master`. This will update the local repository with the README file and now we can push commits to the Github repo.

### Writing the Python app

Hurray! Now we can finally start writing the Python script for the bot. This bot will have a very simple design. There is this very famous scikit-learn [machine learning map](https://scikit-learn.org/stable/tutorial/machine_learning_map/index.html) available which can help data scientists decide on which estimator to try depending on the data. While the original map is extensive for this tutorial we only adapt the classification segment (other segments are clustering, regression and dimensionality reduction).

![](/assets/images/classification.png){: .center-image .img-responsive}

The script given its size is impossible to host here completely. Please click on this [link](https://gist.github.com/escapist21/52346423fdeb0b1c49d6ccbb66a9cfe2) to access the Github gist for this code. 

Create a new file in your project folder named by running `touch telegram_bot.py` , open it with your favorite code editor and paste the code from the above link in this file.

### Explaining the code

Here we will look at parts of the code and try to understand what is happening

```python
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, CallbackQueryHandler, ConversationHandler
from telegram import InlineKeyboardButton, InlineKeyboardMarkup  
import os
```

First, we import the necessary classes from the python-telegram-bot library. The first line imports the various handlers that will be required for this program. You are requested to read up about what these handlers do. The second line imports the modules associated with creating the inline keyboard. Since this is a contextual chatbot, only the inline keyboards will be used to communicate with the bot.

| ![the bot with the inline keyboard markup](/assets/images/bot_screenshot.jpeg){: .center-image .img-responsive} |
| :--: |
| *the bot with the inline keyboard markup* |

```python
# States  
FIRST, SECOND = range(2)

# Callback data  
ONE, TWO, THREE, FOUR, FIVE = range(5)
```

“States” is a list of all conversation steps that will be later used in setting up the _ConversationHandler_. “Callback data” is a list of callbacks that are returned from the inline keyboard upon pressing on it. The “States” along with the “Callback data” will be returned from various functions that will form our conversation logic. Here we just initialize these variables for later use.

```python
def start(update, context):  
    fname = update.message.from_user.first_name

    # Build inline keyboard  
    keyboard1 = [InlineKeyboardButton("<100K samples", callback_data=str(ONE))]  
    keyboard2 = [InlineKeyboardButton(">100K samples", callback_data=str(TWO))]
    
    # create reply keyboard markup  
    reply_markup = InlineKeyboardMarkup([keyboard1, keyboard2])

    # send message with text and appended inline keyboard  
    update.message.reply_text(
        "Hello {}.Let's figure out the best classifer for your data\n\n"
        "How many samples do you have?".format(fname), reply_markup)
    )  
    # tell ConversationHandler that we are in state 'FIRST' now  
    return FIRST  
.  
.  
.

def end(update,context):  
    """Returns 'ConversationHandler.END', which tells the CoversationHandler that the conversation is over"""  
    query = update.callback_query  
    query.answer()  
    query.edit_message_text(  
        "Goodbye, and all the best\n\nIf you need my help again click on /start"  
    )  
    return ConversationHandler.END
```

the start() function is the first function that is called from within the _ConversationHandler_ that we will define later. It is activated in response to the /start command. Once the _ConversationHandler_ is activated it uses the “State” and “Callback data” variable returned from each function to control the conversation flow.

Following the start() function are the other functions that are called as part of the conversation each having a unique inline keyboard, appended message text and returning a “State” and “Callback data”. Refer to the script to study these functions.

These functions are followed by the end() function. The end function is called when the logic flow within the _ConversationHandler_ reaches the end of the conversation. It returns the _ConversationHandler.END_ signal, which effectively terminates the conversation.

```python
def main():  
    #setting to appropriate values  
    TOKEN = "YOUR ACCESS TOKEN"  
    # set up updater  
    updater = Updater(token=TOKEN, use_context=True)  
    # set up dispatcher  
    dispatcher = updater.dispatcher  
    #print a message to terminal to log successful start  
    print("Bot started")

    # Set up ConversationHandler with states FIRST and SECOND  
    conv_handler = ConversationHandler(  
        entry_points=[CommandHandler(command='start', callback=start)],  
        states={  
            FIRST: [CallbackQueryHandler(linear_svc, pattern='^' + str(ONE) + '$'),  
                   .  
                   .  
                   .  
                   ],  
            SECOND: [CallbackQueryHandler(end, pattern='^' + str(ONE) + '$'),  
                   .  
                   .  
                   .  
                   ]  
                },  
            fallbacks=[CommandHandler(command='start', callback=start)]  
        )

    # add ConversationHandler to dispatcher  
    dispatcher.add_handler(conv_handler)  
      
    # start the bot  
    updater.start_polling()

    # run the bot until Ctrl+C is pressed  
    updater.idle()
```

The _Updater_ class continuously fetches new updates from Telegram and passes it to the _Dispatcher_ class. Once the _Updater_ object is created, it is used to create a _Dispatcher_ object and they are then linked together in a queue. Different types of Handlers can be registered with the Dispatcher, which then sorts all the updates received from Telegram according to the registered Handlers. For example, here we add a _ConversationHandler_ to the dispatcher.

The _ConversationHandler_ manages four collections of other handlers. In this example, three such collections are used namely _entry_points_, _states_ and _fallbacks_.

The _entry_point_ collection is a list used to initiate the conversation. In this example, a _CommandHandler_ class is used which responds to the ‘start’ command.

The _states_ collection is a dictionary containing the different conversation steps and one or more associated handlers. In this example, all the conversation steps are associated with _CallbackQueryHandler_ since all our updates from the app are in the form of callbacks associated with pressing an inline keyboard button.

![](/assets/images/flowchart.jpeg){: .center-image .img-responsive}

The flowchart is used to design the conversation steps as defined in the _states_ collection.

The _fallbacks_ collection is a list that is used if the user currently in conversation returns an update which is not of the expected type. For example, a text update is sent when the expected update was a command. This prevents the bot from breaking.

Click on this [link](https://vimeo.com/405917388) to watch a video of the bot in action

Run the python file from the terminal `python telegram_bot.py` and your bot will spring to life.  

### Creating a new app in Heroku

Heroku is a cloud application platform that operates as a PaaS (Platform as a Service). It enables developers to build, run and operate applications entirely in the cloud. If you do not have a free Heroku developer account, you can create one by following this link.
Log in to your Heroku account and your app dashboard will be presented to you. The app dashboard showcases all your apps and lets you manage them. To create a new app click on ‘New’ on the top right corner and select ‘Create new app’ from the dropdown

![](/assets/images/heroku_dashboard.png){: .img-responsive}

In the next step, enter a name for the app and click ‘Create app’

![](/assets/images/heroku_create_new_app.png){: .img-responsive}

Once the app is successfully created, you will be taken to the app management screen

![](/assets/images/heroku_app_management.png){: .img-responsive}

This page allows you to deploy your app, view logs etc. among other operations. We will return to this page later on.


### Setting up  webhook

A webhook is an HTTP push API, which is used to deliver data to other applications in real-time. This is an improvement over typical HTTP push APIs since there is no need for frequent pushing to emulate the real-time feel. The Telegram bot API provides inbuilt methods to set up a webhook.
To set up the webhook open the python script where you have been writing the bot logic and append the following code in the main function only. The rest of the file remains as it is.

```python
# all code before the main function stays as it is

def main():
    # setting to appropriate values
    TOKEN = "YOUR ACCESS TOKEN"
    APPNAME = "HEROKU APP NAME"
    
    # set PORT to be used with Heroku
    PORT = os.environ.get('PORT')

    # set up updater
    updater = Updater(token=TOKEN, use_context=True)
    # set up dispatcher
    dispatcher = updater.dispatcher
    # a print message to log successful initiation of the bot
    # this is for self
    print("Bot started")
    
    # ConversationHandler
    .
    .
    
    # add conversation handler to dispatcher
    dispatcher.add_handler(conv_handler)
    
    # starting webhook and setting it up with heroku app
    updater.start_webhook(listen="0.0.0.0",
                            port=(PORT),
                            url_path=TOKEN)
    
    updater.bot.setWebhook(
      "https://{}.herokuapp.com/{}".format(APPNAME, TOKEN)
      )

    # start the bot
    # updater.start_polling()

    # run the bot until pressed ctrl+c
    # updater.idle()


if __name__ == "__main__":
    main()
```

##### Explaining the appended code
* Set the name of the app that you created in Heroku in the last section to the APPNAME variable

* Start a webhook by calling the `updater.start_webhook()` method and set the arguments to the values as shown in the example

* Set the webhook by calling the `updater.set.Webhook()` method and pass the URL of the Heroku app as shown in the example

* Comment out the `updater.start_polling()` method call. This is no longer required since webhook has been set. Also comment out `updater.idle()` method call.

### Pushing the code to Github repository
We are now ready to push our code to the Github repo. But first three additional files need to be created.

* _requirements.txt_: This file lists all the python packages that will be installed by Heroku in its environment. We have been already working within a virtual environment, and to create this file just type the following command from the terminal
`pip freeze > requirements.txt`

* _Procfile_: All Heroku apps include a Procfile. The Procfile specifies the commands that are to be executed by the app on start up. It typically follows `<process type>: <command>`. Create the Procfile by running `touch Procfile`, open it and add `web: python telegram_bot.py`

* _.gitignore_: This file maintains a list of all the files that do not need to be pushed to the Github repo. Typically this will contain all local environment files. Since we have been using a virtualenv named venv our file should list
`venv/`
`.venv`
`venv.bak`

Once these three files have been created, issued these commands from your terminal

```shell
git add.
git commit -m "first commit"
git push -u origin master
```
All the files in your local git repo will be now available in your remote Github repository.

### Deploying the app on Heroku
Now return to your Heroku app page. Under the ‘**Deploy**’ tab, three deployment methods will be listed. We will select the **Github method**.

Give the necessary permissions to Heroku to access your Github profile. Once the authorization has been set up, search for the name of your repository from the search bar and click on ‘**Connect**’.

Once the repository has been connected to the Heroku app, click on ‘**Deploy branch**’ under the ‘**Manual deploy**’ section, pointing to the ‘**main**’ branch. The app will now build itself and if all the steps are followed correctly, your bot will be hosted.

**Please note** that visiting your app URL will return a ‘**404: page not found**’ error but that is alright since the script does not necessarily return anything. If problems are encountered while starting the app, visiting the URL will display the error message.

So, that’s it for this tutorial. Happy coding!