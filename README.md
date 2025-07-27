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
- [Send and Receive Media](#send-and-receive-media)
- [Chains Of Middlewares](#chains-of-middlewares)
- [State, Sessions and Wizards](#state-session-and-wizards)
- [Personal Project](#personal-project)

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
**How they work?**. When a user presses an inline button, the bot receives a special `callback_query` update instead of a text message.

**Usage:**
First import `Markup` helper which provided by Telegraf.
It takes an array of arrays (or object), where each inner array(or object) represents a row of buttons.
We use this syntax `Markup.button.callback(text, callback_data)` as second argument to `ctx.reply` or other `replyWith...` methods.:

```js
// ... Previous lines
bot.command("menu", async (ctx) => {
  // pass 'Markup.inlineKeyboard' as second argument
  await ctx.reply(
    "Main Menu:",
    Markup.inlineKeyboard([
      // first row of btns
      [
        Markup.button.callback("🎵 Song File", "song_file"), // inner array first btn
        Markup.button.callback("🎶 Song MP3", "song_mp3"), // inner array second btn
      ],
      // second row of btns
      [
        Markup.button.callback("🇬🇧 English", "english"),
        Markup.button.callback("🇮🇷 Persian", "persian"),
      ],
    ])
  );
});
```

**How to handle a button press?** To handle each `callback_query` generated by a button press, the `bot.action()` method is used. worth mention that we should use `ctx.answerCbQuery()` to stop a button press loading status:

```js
//... (bot setup and /menu command from above)

// Handler for the 'song_file' callback
bot.action("song_file", (ctx) => {
  // Prevent btn from loading
  ctx.answerCbQuery("Fetching stats...");

  // Send a new message with the stats
  ctx.reply("You select song file!");
});

// Handler for the 'persion' callback
bot.action("persion", (ctx) => {
  ctx.answerCbQuery(); // Stop loading
  ctx.reply("you select persion, such a beautifull chose!");
});

bot.launch();
```

**Changing Keyboard Message** : Let's say you wanna change the message's text after selecting a yes or no button:

```js
//... (bot setup)

bot.command("confirm", async (ctx) => {
  await ctx.reply(
    "Confirmation button",
    Markup.inlineKeyboard([
      [
        Markup.button.callback("Yes", "yes_btn"),
        Markup.button.callback("No", "no_btn"),
      ],
    ])
  );
});
// Handle yes btn
bot.action("yes_btn", async (ctx) => {
  await ctx.editMessageText("message text is chenged...right?");
  //stoping the loading status
  ctx.answerCbQuery();
});
// Handle no btn
bot.action("no_btn", async (ctx) => {
  // inline-keyboard changed to only one btn
  await ctx.editMessageText(
    "you select no",
    Markup.inlineKeyboard([
      [Markup.button.callback("Finished of keyboards", "finish")],
    ])
  );
  ctx.answerCbQuery();
});

bot.launch();
```

**Custom Reply Keyboards:** Unlike inline keyboards, which are part of a message, custom reply keyboards replace the user's standard text keyboard with a set of predefined buttons:

```js
//...previous lines
bot.command("status", async (ctx) => {
  // Use 'Markup.keyboard()' for reply keyboards
  await ctx.reply(
    "status? ",
    Markup.keyboard([
      ["happy", "sasd"],
      ["angry", "anxios"],
      ["absurd", "meloncholic"],
    ])
      .oneTime()
      .resize()
  ); // Use 'oneTime()' with 'resize()' for better stying
});
// instead of 'bot.action()' we should use 'bot.hears()' here
bot.hears("angry", async (ctx) => {
  await ctx.reply("oh..so you are angry huh??");
});
```

### Send And Receive Media

A key feature of Telegram bots is their ability to send and receive various types of media.It's very usefull to know:
To process files we should use `bot.on()` method with appropriate message filter. good to know that files will stored on Telegram's servers, therefor they have an ID wich is very useful. as you can guess When a file is received, the context object(`ctx`) will contain information about it such as `file_id`.

You can receive media and access to its data:

```js
// ... previous lines
// FILE HANDLING
bot.on(message("document"), async (ctx) => {
  const document = await ctx.message.document; // 'ctx' contains file's information
  const document_id = document.file_id; // file's ID
  const document_name = document.file_name; // file's name
  const document_size = document.file_size; // files's size
  console.log(
    `File name is: ${document_name} - size: ${document_size} - id is : ${document_id}`
  );
  await ctx.reply(
    `File name is: ${document_name} - size: ${document_size} - id is : ${document_id}`
  );
});

bot.lauch();
```

You can send Media as well, Telegraf provides a family of `replyWith` methods, these methods accept different types of sources for the file content:

- By `file_id` (String): The most efficient method for sending a file that is already on Telegram's servers.

- By URL (String): Telegram will download the file from the public URL and send it to the user.

- By Local File/Buffer/Stream: Using the `Input` helper class, files can be sent from a local path, a `Buffer`, or a readable stream.

```js
// ...previous lines
// SEND FILE
bot.command("media", async (ctx) => {
  // Send photo with URL + Caption
  await ctx.replyWithPhoto("https://picsum.photos/200/300/", {
    caption: "this is cation",
  });
  // Send from Local file Path
  await ctx.replyWithDocument(Input.fromLocalFile("./path/to/file.pdf"));
  // Send with id - assuming stickerId was stored previously
  const stickerId = "lkasdCASDFlkjf...";
  await ctx.replyWithSticker(stickerId);
});

bot.lauch();
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

### State, Session and Wizards

Untill now our bot was only dealing with one user, What if there ware more then one user? how should bot **remember** which is who? There are different approaches for it which are depends on how you want your bot to remember and act:

> Note: because I stuggled too much to grasp these concepts, I will dive deep into it.

**Terminalogy & Code**

- What are **state** and **session**? By default, interactions with a Telegram bot are **stateless**. The bot has no memory of previous messages from a user; each **update** is processed in isolation. To create meaningful conversations, the bot needs a way to store and retrieve data associated with a specific user or chat. This is achieved through **sessions**.

```js
const { Telegraf, session } = require("telegraf");
// require'dotenv'
require("dotenv").config;
//create a bot instace with our token
const bot = new Telegraf(process.env.BOT_TOKEN);
// Use and Configure our session middleware
bot.use(
  session({
    initial: () => ({ counter: 0 }),
  })
);

bot.on("text", async (ctx) => {
  ctx.session.counter++; //increment the number of messages for each user
  await ctx.reply(`Message counter is: ${ctx.session.counter}`);
});

// Start the Bot
bot.launch();
```

It's suffisiont but all session data is lost whenever tho bot's process is terminated or re-started, for more data persistance we should use `telegraf-session-local` package, It stores session data in a local JSON file, providing a simple yet effective persistence layer:

```js
const { Telegraf } = require("telegraf");
// Import 'telegraf-session-local'
const LocalSession = require("telegraf-session-local");

const bot = new Telegraf(process.env.BOT_TOKEN);
// Configure session so it use session_db.json file
const localSession = new LocalSession({ database: "session_db.json" });
// Use our session middleware
bot.use(localSession.middleware());

bot.on("text", async (ctx) => {
  ctx.session.counter = ctx.session.counter || 0;
  ctx.session.counter++; //keep trach of msg numbers for each user

  await ctx.reply(`Message Count: ${ctx.session.counter}`);
  return next();
});

// Remove from database
bot.command("remove", async (ctx) => {
  await ctx.reply("Session data removed");
  ctx.session = null;
});
```

> You can also use MongoDB which I didn't learned it yet, if you know and how free time, you can add it!

- What are **WizardScene**s and **Stage**s? to put it simply, we are just fine with using above methods BUT when it comes to managin complex, multi-step conversations(like a user registration flow or ) that you want prevent users from accidentally triggering other bot functions while in the middle of a dialog, here is where **WizardScene** comes in handy.A **Scene** functions as a modal state machine for the user. When a user enters a **Scene**, the bot's global command and hears handlers are temporarily disabled for that user. This creates an isolated conversational environment where only the listeners defined within the **current scene** step are active. You may ask: okay, but how to do that? before that you need to grasp these Core Components:
  **Core Components and Setup**
  Implementing a wizard involves three core components:

1.  **`WizardScene`**: Defines the sequence of steps (functions) in the conversation.

2.  **`Stage`**: A container that registers all the scenes available to the bot.

3.  **`session`**: Scenes rely on sessions to keep track of the user's current scene and step.
    > Note:The setup requires registering the middleware in a specific order: `session` must be registered **before** the `stage` middleware.

**The "Magic Formula" for a Wizard Scene**

1. **Import necessary modules:** `Telegraf`, `Scenes`, `session`.

2. **Define the steps:** Create an instance of `Scenes.WizardScene`, providing a unique ID and a series of handler functions, one for each step of the conversation.

3. **Create and register the stage:** Create a `Scenes.Stage` instance, passing an array of all your scenes.

4. **Register middleware:** Use `bot.use(session())` followed by `bot.use(stage.middleware())`.

5. **Create an entry point:** Use a command like `bot.command('register',...)` to trigger `ctx.scene.enter('your-wizard-id')`.

**Navigating & Managing the flow**

- **Entering:** `ctx.scene.enter('scene-id')` starts the wizard.

- **Advancing:** `return ctx.wizard.next()` moves to the next step handler.

- **Leaving:** `return ctx.scene.leave()` exits the scene and clears its state.

- **State:** The `ctx.wizard.state` object is used to store data collected across steps. This state is automatically cleared when the scene is left, which is a major advantage over using the global `ctx.session` for temporary conversational data.

Now I assume You can comprehend below code:

```js
// Import Scenes
const { Telegraf, Scenes, session, Markup } = require("telegraf");

// Step 1: Define the wizard scene and its steps
const registrationWizard = new Scenes.WizardScene(
  "registration-wizard",
  // Step 1: Ask for name
  (ctx) => {
    ctx.reply("Welcome to registration! What is your name?");
    // Initialize the state object for this wizard session
    ctx.wizard.state.userData = {};
    return ctx.wizard.next();
  },
  // Step 2: Ask for email
  (ctx) => {
    // Validate and store the name from the previous step
    if (ctx.message.text.length < 2) {
      ctx.reply("Please enter a valid name.");
      return; // Stay on the current step
    }
    ctx.wizard.state.userData.name = ctx.message.text;

    ctx.reply("Got it. Now, what is your email address?");
    return ctx.wizard.next();
  },
  // Step 3: Confirm and leave
  async (ctx) => {
    // Validate and store the email
    // A simple regex for email validation
    if (!/^\S+@\S+\.\S+$/.test(ctx.message.text)) {
      ctx.reply("Please enter a valid email address.");
      return; // Stay on the current step
    }
    ctx.wizard.state.userData.email = ctx.message.text;

    const { name, email } = ctx.wizard.state.userData;
    await ctx.reply(
      `Thanks, ${name}! Your registration is complete. Your email is ${email}.`
    );

    // The wizard is finished, leave the scene
    return ctx.scene.leave();
  }
);

const bot = new Telegraf(process.env.BOT_TOKEN);

// Step 2: Create a stage and register the wizard
const stage = new Scenes.Stage();

// Step 3: Register session and stage middleware (order is important)
bot.use(session());
bot.use(stage.middleware());

// Step 4: Create a command to enter the wizard
bot.command("register", (ctx) => {
  ctx.scene.enter("registration-wizard");
});

bot.launch();
```

### Personal Project 

Now that we learn how to create our own bot, it's time to create something and learn in practice, funny thing here is that you wouldn't learn anything if don't fail! so try to fail.

I wil add my project here. Soon... 