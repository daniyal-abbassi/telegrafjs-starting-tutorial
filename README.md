# Telegraf.js Tutorial

This tutorial is for absolute beginners who want to create a Telegram bot using [Telegraf.js](https://telegraf.js.org/).

### Why Telegraf.js and Not [node-telegram-bot-api](https://github.com/yagop/node-telegram-bot-api)?

There are several reasons to choose Telegraf.js, but the key one, especially for beginners like us, is its **middleware-based architecture** (think Express.js or Koa).

### Steps

- [Creating a New Bot with BotFather](#1-get-your-bot-token-from-botfather)
- [First message: Hello Telegram](#2-hello-telegram)
- [Different types of message](#3-handling-different-types-of-message)

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

> We will discuss about `ctx` later.

**Send a message on `/help` command**

```js
bot.help(async (ctx) => await ctx.reply("I'm a help message!"));
```

**Reply with a specific message**

```js
bot.hears("hi", async (ctx) => await ctx.reply("polite...Hi There"));
```

You can also use regex to detect a specific text in message like cat:

```js
bot.hears(
  /cat|kitten/i,
  async (ctx) => await ctx.reply("Did someone say cat?")
);
```

Also you can use regex with `ctx.match` to store data and use them:

```js
bot.hears(/^\/auth (\w+) (.+)$/, async (ctx) => {
  const username = ctx.match[1]; // extract username from matched array
  const password = ctx.match[2];
  await ctx.reply(`Username is: ${username} And Password is: ${password}`);
});
/*
ctx.match array is like this: 
[
  '/auth johnDoe 12345678',
  'jognDoe',
  '12345678',
  index: 0,
  input: '/auth johnDoe 12345678',
  groups: undefined
]
**/
```

**Reply with a command**

```js
bot.command("CommandText", async (ctx) => await ctx.reply("Command invoked"));
```

### 3-Handling different types of message

What if you want your bot to respond base on your specific actions or message type? Like:

- Sending it a sticker, photo, video, voice ect...
- Sending it a chat ID, a user's ID, group specifc action ect...
- And much more...

The `bot.on()` method is the most general-purpose handler. It listens for specific update types directly from the Telegram API. These types include not only text messages but also non-text events like a user sending a sticker, photo, document, or audio file.

We'll use `message` filter from `telegraf/filters` to specify the subtype of message to handle.

```js
//...Previous lines
const { message } = require("telegraf/filters"); // import the filter

// Listen for any sticer sent by the user
bot.on(message("sticker"), async (ctx) => await ctx.reply("Nice Sticker!!!"));

// Listen for any medio types like photo
bot.on(
  message("photo"),
  async (ctx) => await ctx.reply("Is that you in this photo !?")
);

// Listen for any text message (general text handler)
// Use it when 'bot.command()' or 'bot.hears()' couldn't caught your text
bot.on(message("text"), async (ctx) => {
  const recievedText = ctx.message.text;
  // Your logic here
  if (recievedText.includes("shit")) {
    await ctx.reply("Language!!!");
  }
});
```
