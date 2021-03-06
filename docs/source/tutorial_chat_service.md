# Using Chat Services

This is a tutorial for connecting your trained ParlAI agents to various chat services in order to allow your models to talk to humans. Humans using chat services (like Facebook Messenger) can be viewed as another type of agent in ParlAI and communicate in the standard observation/act dict format.

We currently support the following chat services:

1. **Browser**
2. **Facebook Messenger**
3. **Terminal**
4. **Web Sockets**

You can find more information on how to set up these services below.

Please read here [here](https://github.com/facebookresearch/ParlAI/tree/master/parlai/chat_service) for information on how to set up a new chat service.

## Overview

As stated, humans messaging on chat services can be viewed as a type of agent in ParlAI, communicating with models via observations and action dicts. See [here](https://github.com/facebookresearch/ParlAI/blob/master/parlai/chat_service/core/agents.py) for the basic implementation. Human agents are placed in worlds with ParlAI agent(s) and possibly other humans, and the world defines how each of these agents interacts.

The chat environment is defined by the **task**. A task typically consists of an `Overworld`, which can spawn subtasks (subworlds) or serve as a "main menu", allowing people to pick from multiple conversation options. The task definition resides in a config file, `config.yml`, in which all all available worlds and any additional commandline arguments.

Here is an example config file for a Messenger chatbot:

```
tasks:
  default:
    onboard_world: MessengerBotChatOnboardWorld
    task_world: MessengerBotChatTaskWorld
    timeout: 1800
    agents_required: 1
task_name: chatbot
world_module: parlai.chat_service.tasks.chatbot.worlds
overworld: MessengerOverworld
page_id: 1 # Configure Your Own Page
max_workers: 30
opt:  # Additional model opts go here
  debug: True
  model: legacy:seq2seq:0
  model_file: models:convai2/seq2seq/convai2_self_seq2seq_model
  override:
    model: legacy:seq2seq:0
```

After following the set-up instructions (detailed below), this task could be run with the following command from the `parlai/chat_service/services/messenger` directory:
```
python run.py --config-path ../../tasks/chatbot/config.yml
```


## Example Tasks

As an example, the [Overworld Demo](https://github.com/facebookresearch/ParlAI/blob/master/parlai/chat_service/tasks/overworld_demo/) displays three separate tasks connected together by an overworld.

- The `echo` task is a simple example of an echo bot, and shows the functionality and flow of a simple single-person world.
- The `onboard data` task is an example that shows how an onboarding world can collect information that is later exposed in the active task world.
- The `chat` task is an example of a task that requires multiple users, and shows how many people can be connected together in an instance of a world and then returned to the overworld upon completion of a task.

In addition to the overworld demo, the following example tasks are provided:

- [Generic Chatbot](https://github.com/facebookresearch/ParlAI/blob/master/parlai/chat_service/tasks/chatbot/): Allow conversations with any ParlAI models, for instance the [PersonaChat](https://github.com/facebookresearch/ParlAI/tree/master/projects/personachat) model.
- [QA Data Collection](https://github.com/facebookresearch/ParlAI/blob/master/parlai/chat_service/tasks/qa_data_collection/): collect questions and answers from people, given a random Wikipedia paragraph from SQuAD.

### Creating your Own Task

To create your own task, start with reading the tutorials on the provided examples, and then copy and modify the example `worlds.py` and `config.yml` files to create your task.

**A few things to keep in mind:**

1. A conversation ends when a call between `parley` calls to `episode_done` returns True.
2. Tasks with an overworld should return the name of the world that they want to queue a user into from the ``parley`` call in which the user makes that selection to enter a world.
3. Tasks with no overworld will immediately attempt to put a user into the queue for the default task onboarding world or actual task world (if no onboarding world exists), and will do so again following the completion of a world (via `episode_done`).
3. To collect the conversation, data should be collected during every `parley` and saved during the `world.shutdown` call. You must inform the user of the fact that the data is being collected as well as your intended use.
4. Finally, if you wish to use and command line arguments as you would in ParlAI, specify those in the `opt` section of the config file.


## Available Chat Services


### Browser
This allows you to participate in a ParlAI world as an agent using a local browser.
This extends the `websocket` chat service implementation to run a server locally,
which you can send and receive messages using a browser.

#### Setup

1. Run: `python parlai/chat_service/services/browser_chat/run.py --config-path path/to/config.yml --port PORT_NUMBER`
2. Run: `python client.py --port PORT_NUMBER`
3. Interact

**Example command:** `python parlai/chat_service/services/browser_chat/run.py --config-path parlai/chat_service/tasks/chatbot/config.yml --port 10001`

If no port number is specified in `--port` then the default port used will be `34596`. If specifying, ensure both port numbers match on client and server side.


### Facebook Messenger

This allows you to chat with a ParlAI model on Facebook Messenger.

<p align="center"><img width="80%" src="_static/img/messenger-example.png" /></p>


#### Setup

- ParlAI's Messenger functionality requires a free heroku account which can be obtained [here](https://signup.heroku.com/). Running any ParlAI Messenger operation will walk you through linking the two.

- Running and testing a bot on the [Facebook Messenger Platform](https://developers.facebook.com/docs/messenger-platform) for yourself will require following the guide to set up a [Facebook App](https://developers.facebook.com/docs/messenger-platform/getting-started/app-setup) for Messenger. Skip the set up your webhook step, as ParlAI will do it for you.

- When the guide asks you to configure your webhook URL, you're ready to run the task. This can be done by running the `run.py` file in with python.

- After the heroku server is setup, the script will print out your webhook URL to the console, this should be used to continue the tutorial. The default verify token is `Messenger4ParlAI`. This URL should be added in the Webhook section. The webhook subscription fields should also be edited to subscribe to the `messages` field.

- On the first run, the page will ask you for a "Page Access Token," which is also referred to on the messenger setup page. Paste this in to finish the setup. You should now be able to communicate with your ParlAI world by messaging your page.

- To open up your bot for the world to use, you'll need to submit your bot for approval from the [Developer Dashboard](https://developers.facebook.com/apps/).

**Note:** When running a new task from a different directory, the webhook url will change. You will need to update this in the developer console from the webhook settings using "edit subscription." Your Page Access token should not need to be changed unless you want to use a different page.

Additional flags can be used (you can also specify these in the `config.yml` file):

- `--password <value>` requires that a user sends the message contained in `value` to the bot in order to access the rest of the communications.
- `--force-page-token` forces the script to request a new page token from you, allowing you to switch what page you're running your bot on.
- `--verbose` and `--debug` should be used before reporting problems that arise that appear unrelated to your world, as they expose more of the internal state of the messenger manager.

**Other things to keep in mind when creating your Messenger tasks:**
- Your world can utilize the complete set of [Facebook Messenger Templates](https://developers.facebook.com/docs/messenger-platform/send-messages/templates) by putting the formatted data in the 'payload' field of the observed action.
- Quick replies can be attached to any action, the `MessengerOverworld` of the [Overworld Demo](https://github.com/facebookresearch/ParlAI/blob/master/parlai/chat_service/tasks/overworld_demo/) displays this functionality.


### Terminal

This allows you to participate in a ParlAI world as an agent using the terminal.
This extends the `websocket` chat service implementation to run a server locally,
which you can send and receive messages from using the terminal.

#### Setup

1. Run: `python parlai/chat_service/services/terminal_chat/run.py --config-path path/to/config.yml --port PORT_NUMBER`
2. Run: `python client.py --port PORT_NUMBER`
3. Interact

**Example command:** `python parlai/chat_service/services/terminal_chat/run.py --config-path parlai/chat_service/tasks/chatbot/config.yml --port 10001`


If no port number is specified in `--port` then the default port used will be `34596`. If specifying, ensure both port numbers match on client and server side.

### Web Sockets

See **Browser** above for an example implementation of a websockets-based chat service. You can view the code [here](https://github.com/facebookresearch/ParlAI/tree/master/parlai/chat_service/services/browser_chat).

### Adding a New Chat Service

For full instructions on adding a new chat service to ParlAI, please read [here](https://github.com/facebookresearch/ParlAI/tree/master/parlai/chat_service/README.md).
