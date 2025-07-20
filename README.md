# Telegraf.js Tutorial

This tutorial is for absolute beginners who want to create a Telegram bot using [Telegraf.js](https://telegraf.js.org/).

### Why Telegraf.js and Not [node-telegram-bot-api](https://github.com/yagop/node-telegram-bot-api)?

There are several reasons to choose Telegraf.js, but the key one, especially for beginners like us, is its **middleware-based architecture** (think Express.js or Koa).

### Steps

- [Creating a New Bot with BotFather](#1-get-your-bot-token-from-botfather)
- [More steps coming soon as we build this tutorial together!]

### 1-Get Your Bot Token from BotFather

The first step is to create a bot using BotFather:

1. Open Telegram and search for the official [@BotFather](https://t.me/BotFather) account (look for the blue checkmark).
2. Start a chat and send the `/newbot` command (or select it from the menu).
3. Choose a name, then pick a unique username ending in `bot` (e.g., `@MyUniqueTestBot`).
4. If the username is available, BotFather will provide a **token**.
5. Copy the token and save it securely (e.g., in a `.env` file for later use).

<img src="https://github.com/daniyal-abbassi/telegrafjs-starting-tutorial/blob/main/screenshots/botfather.png" height="500" width="390">

> **Tip**: Never share your bot token publicly! It’s like a password for your bot.

botFather provides several adjastements for you bot, here is a short list of them:

- `/setabouttext` : change about section
- `/setcommands` : change bot's commands
- `/token` : change you token
- `/setdescription` : change middle page text
- `/deletebot` , `/mybots` and much more which we don't cover in this tutorial.

### 2-Hello Telegram!

Let's create our first message:

> I assume we already have Set-up a node.js project

first off install telegraf using `npm`

```
    npm install telegraf --save
```

Then in you bot.js file (name doesn't matter) require telegraf class instance

```js
const { Telegraf } = require("telegraf"); //use curly bracets because it is an instance of telegraf class
```

Now create a new bot with your token and luanch the bot

```js
const bot = new Telegraf("YOUR_BOT_TOKEN"); //it is best-practice to save it .env file and then access it with process.env.BOT_TOKEN
bot.lounch();
```

Now you have a bot instance that can receive **updates** from a user.

> An **update** can be anything from a user: text message, stickers, media files.

    And that's it! now you can try and communicate with your bot

    **Send a message at start**

    When we start the bot we want it to say `Hello Telegram!` to us:

```js
// Complete code
const { Telegraf } = require("telegraf");
require("dotenv").config; //best-practice 
// Create a bot instance
const bot = new Telegraf(process.env.BOT_TOKEN);
// Lauch the bot
bot.launch();
// Reply with Hello Telegram  
bot.start(async (ctx) => await ctx.reply("Hello Telegram!"));
```
