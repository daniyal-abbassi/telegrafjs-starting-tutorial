# Telegraf.js Tutorial

This tutorial is for absolute beginners who want to create a Telegram bot using [Telegraf.js](https://telegraf.js.org/).

### Why Telegraf.js and Not [node-telegram-bot-api](https://github.com/yagop/node-telegram-bot-api)?

There are several reasons to choose Telegraf.js, but the key one, especially for beginners like us, is its **middleware-based architecture** (think Express.js or Koa).

### Steps

- [Creating a New Bot with BotFather](#1-get-your-bot-token-from-botfather)
- [First message: Hello Telegram](#2-hello-telegram)
- [Different types of message](#3-handling-different-types-of-message)
- [The Telegraf Context (ctx): The Heart of The Bot](#4-the-telegraf-context-ctx-the-heart-of-the-bot)
  **Advance topics**
- [Keyboards](#keyboards)
- [Chains Of Middlewares](#chains-of-middlewares)

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

### 4-The Telegraf Context (ctx): The Heart of The Bot

The **Context object** (conventionally named `ctx`) is the central nervous system of every interaction.`ctx` is created for each incoming update, It contains not only the update data but also a direct interface to the Telegram API and a set of convenient shortcut methods:

- `ctx.update` has access to low-level information about the entire update.
- `ctx.message` access to the message object, which contains details like the text, date, and any attached media.
- `ctx.from` has access to information about the user who sent the update, such as their ID, first name, and username.
- `ctx.chat` containing information about the chat where the update comming from , including its ID and type (e.g., 'private' or 'group')
- `ctx.telegram` has full access to Telegram API, You can make amy raw API call like:
  `await ctx.telegram.deleteMessage(ctx.chat.id,ctx.message.message_id);`

It's goot to know that methods like `ctx.reply()` which we ware using are actually **shortcuts**!
For example for sending a message:

```js
// without using shortcut
await ctx.telegram.sendMessage(ctx.message.chat.id, "Hello");
// with shorcut
await ctx.reply("Hello");
```

Here are some usefull shortcuts:

- `ctx.reply(text, [extra])`: Sends a text message.

- `ctx.replyWithPhoto(source, [extra])`: Sends a photo.

- `ctx.replyWithDocument(source, [extra])`: Sends a document.

- `ctx.replyWithSticker(source, [extra])`: Sends a sticker.

- `ctx.replyWithLocation(latitude, longitude, [extra])`: Sends a map location.

- `ctx.leaveChat()`: Makes the bot leave the current chat.

> **Note**: You can work with you bot by now. next sections are more advanced stuff!.

### Keyboards
Let's move further and explore in-line keyboard which infact are very intersting!

**What are Keyboards?** Inline keyboards are buttons that are attached directly to a message sent by the bot.
**Why use them?** They provide a clean and intuitive way for users to interact, making them ideal for menus, confirmation dialogs, and navigation systems. 
**How they work?**. When a user presses an inline button, the bot receives a special ```callback_query``` update instead of a text message.

**Usage:**
First import ``Markup`` helper which provided by Telegraf.
It takes an array of arrays (or object), where each inner array(or object) represents a row of buttons.
We use this syntax ``Markup.button.callback(text, callback_data)`` as second argument to ``ctx.reply`` or other ``replyWith...`` methods.:
```js
// ... Previous lines
bot.command('menu', async (ctx) => {
    // pass 'Markup.inlineKeyboard' as second argument
    await ctx.reply('Main Menu:', Markup.inlineKeyboard([
        // first row of btns
        [
            Markup.button.callback('🎵 Song File', 'song_file'), // inner array first btn
            Markup.button.callback('🎶 Song MP3', 'song_mp3') // inner array second btn
        ],
        // second row of btns
        [
            Markup.button.callback('🇬🇧 English', 'english'),
            Markup.button.callback('🇮🇷 Persian', 'persian')
        ]
    ]));
});
```
**How to handle a button press?** To handle each ``callback_query`` generated by a button press, the ``bot.action()`` method is used. worth mention that we should use ``ctx.answerCbQuery()`` to stop a button press loading status:
```js
//... (bot setup and /menu command from above)

// Handler for the 'song_file' callback
bot.action('song_file', (ctx) => {
  // Prevent btn from loading
  ctx.answerCbQuery('Fetching stats...');
  
  // Send a new message with the stats
  ctx.reply('You select song file!');
});

// Handler for the 'persion' callback
bot.action('persion', (ctx) => {
  ctx.answerCbQuery(); // Stop loading
  ctx.reply('you select persion, such a beautifull chose!');
});

bot.launch();

```
**Changing Keyboard Message** : Let's say you wanna change the message's text after selecting a yes or no button: 
```js
//... (bot setup)

bot.command('confirm',async (ctx) => {
    await ctx.reply('Confirmation button',Markup.inlineKeyboard([
        [
            Markup.button.callback('Yes','yes_btn'),
            Markup.button.callback('No','no_btn')
        ]
    ]))
})
// Handle yes btn
bot.action('yes_btn',async (ctx) => {
    await ctx.editMessageText('message text is chenged...right?');
    //stoping the loading status
    ctx.answerCbQuery()
})
// Handle no btn
bot.action('no_btn',async (ctx) => {
    // inline-keyboard changed to only one btn
    await ctx.editMessageText('you select no',Markup.inlineKeyboard([
        [
            Markup.button.callback('Finished of keyboards','finish')
        ]
    ]));
    ctx.answerCbQuery();
})

bot.launch();
```
### Chains Of Middlewars

You can modify and add propertis to `ctx` object which is usefull for sessions and database.
here we add a role and responed time to `ctx` object :

```js
// Middleware to add a custom property to the context
bot.use((ctx, next) => {
  ctx.state.role = "admin"; // Add a 'role' property to the state
  ctx.customData = { startTime: Date.now() }; // Add a custom property directly
  return next(); // Pass control to the next middleware
});

bot.command("test", (ctx) => {
  const responseTime = Date.now() - ctx.customData.startTime;
  // Access the properties set in the middleware
  ctx.reply(`Hello ${ctx.state.role}! Response time: ${responseTime}ms`);
});

bot.launch();
```
