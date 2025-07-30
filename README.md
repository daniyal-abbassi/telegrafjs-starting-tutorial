# Telegraf.js Tutorial

This tutorial is for absolute beginners who want to create a Telegram bot using [Telegraf.js](https://telegraf.js.org/).

### Why Telegraf.js and Not [node-telegram-bot-api](https://github.com/yagop/node-telegram-bot-api)?

There are several reasons to choose Telegraf.js, but the key one, especially for beginners like us, is its **middleware-based architecture** (think Express.js or Koa).

### Steps

- [Creating a New Bot with BotFather](#1get-your-bot-token-from-botfather)
- [First message: Hello Telegram](#2hello-telegram)
- [Different types of message](#3handling-different-message-types)
- [The Telegraf Context (ctx): The Heart of The Bot](#4the-telegraf-context-ctx-the-heart-of-the-bot)

    **Advance topics**

- [Keyboards](#keyboards)
- [Send and Receive Media](#send-and-receive-media)
- [Chains Of Middlewares](#chains-of-middlewars)
- [State, Sessions and Wizards](#state-session-and-wizards)
- [Personal Project](#personal-projects)

### 1.Get Your Bot Token from BotFather

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

### 2.Hello Telegram!

Let's make our bot send its first message.

> I assume we already have Set-up a node.js project

First, install telegraf using `npm`

```
    npm install telegraf --save
```
We also installed `dotenv` to handle our secret bot token.
Then in you bot.js file (the name doesn't matter) require telegraf class instance

```js
//// Destructuring: we're importing the Telegraf class from the telegraf package.
const { Telegraf } = require("telegraf");
```

Now create a new bot with your token and luanch the bot

```js
// It's best practice to save your token in a .env file
// and access it using process.env.BOT_TOKEN
const bot = new Telegraf("YOUR_BOT_TOKEN"); 
bot.lounch(); // This starts your bot
```

You now have a bot that can receive updates from users.

> An **update** can be anything from a user: text message, stickers, media files.

And that's it! now you can try and communicate with your bot `node bot.js`

**Send a message at start**

When a user first starts a chat with our bot, we want it to say `Hello Telegram!` :

```js
// Complete code
const { Telegraf } = require("telegraf");
require("dotenv").config; // Loads environment variables from a .env file

// Create a bot instance
const bot = new Telegraf(process.env.BOT_TOKEN);

// Reply with "Hello, Telegram!" when a user starts the bot
bot.start(async (ctx) => await ctx.reply("Hello Telegram!"));

// Lauch the bot
bot.launch();
```

> We will discuss about `ctx` later. For now, just know it's how you interact with Telegram.

**Send a message on `/help` command**

```js
bot.help(async (ctx) => await ctx.reply("I'm a simple help message!"));
```

**Reply with a specific message**
The `bot.hears()` method lets you listen for specific words or phrases.
```js
bot.hears("hi", async (ctx) => await ctx.reply("Oh, hello to you too!"));
```

You can also use a regular expression (RegExp) to detect patterns. The `i` flag makes it case-insensitive.

```js
bot.hears(
  /cat|kitten/i,
  async (ctx) => await ctx.reply("Did someone say cat? 😻")
);
```

You can even capture parts of the message using groups in your RegExp. The captured parts are available in `ctx.match`.

```js
bot.hears(/^\/auth (\w+) (.+)$/, async (ctx) => {
  const username = ctx.match[1]; // from the first capture group: (\w+)
  const password = ctx.match[2]; // from the second capture group: (.+)
  await ctx.reply(`Username is: ${username} And Password is: ${password}`);
});

/*
For the input '/auth johnDoe 12345678', the ctx.match array looks like this:
[
  '/auth johnDoe 12345678', // The full matched string
  'johnDoe',                // The first captured group
  '12345678',               // The second captured group
  index: 0,
  input: '/auth johnDoe 12345678',
  groups: undefined
]
*/
```

**Reply with a command**
The `bot.command()` method is the standard way to handle commands like `/mycommand`.

```js
bot.command("CommandText", async (ctx) => await ctx.reply("Command invoked"));
```

### 3.Handling Different Message Types

What if you want your bot to react to something other than text?

- A user sending a sticker, photo, video, or voice message.
- A user joining or leaving a group.
- And much more...

The `bot.on()` method is the most general-purpose handler. It listens for specific update types directly from the Telegram API. These types include not only text messages but also non-text events like a user sending a sticker, photo, document, or audio file.

We'll use `message` filter from `telegraf/filters` to specify the subtype of message to handle.

```js
//...Previous lines
const { message } = require("telegraf/filters"); // import the filter

// Listen for any sticker sent by the user
bot.on(message("sticker"), async (ctx) => await ctx.reply("Nice sticker! 👍"));

// Listen for any media types like photo
bot.on(
  message("photo"),
  async (ctx) => await ctx.reply("Is that you in this photo !?")
);

// Listen for any text message (this is a general text handler)
// It will only run if other handlers like bot.command() or bot.hears() don't catch the message first.
bot.on(message("text"), async (ctx) => {
  const recievedText = ctx.message.text;
  // Your logic here
  if (recievedText.includes("secret")) {
    await ctx.reply("Shhh, it's a secret!");
  }
});
```

### 4.The Telegraf Context (ctx): The Heart of The Bot

The **Context object** (conventionally named `ctx`) is the central nervous system of every interaction.`ctx` is created for each incoming update, It contains not only the update data but also a direct interface to the Telegram API and a set of convenient shortcut methods:

- `ctx.update` has access to low-level information about the entire update.
- `ctx.message` access to the message object, which contains details like the text, date, and any attached media.
- `ctx.from` has access to information about the user who sent the update, such as their ID, first name, and username.
- `ctx.chat` containing information about the chat where the update comming from , including its ID and type (e.g., 'private' or 'group')
- `ctx.telegram` has full access to Telegram API, You can make amy raw API call like:
  `await ctx.telegram.deleteMessage(ctx.chat.id,ctx.message.message_id);`

It's goot to know that methods like `ctx.reply()` which we've been using, are actually **shortcuts**!

For example for sending a message:
```js
// without using a shortcut
await ctx.telegram.sendMessage(ctx.message.chat.id, "Hello");
// with the shorcut
await ctx.reply("Hello");
```

Here are some other usefull shortcuts:

- `ctx.reply(text, [extra])`: Sends a text message.

- `ctx.replyWithPhoto(source, [extra])`: Sends a photo.

- `ctx.replyWithDocument(source, [extra])`: Sends a document.

- `ctx.replyWithSticker(source, [extra])`: Sends a sticker.

- `ctx.replyWithLocation(latitude, longitude, [extra])`: Sends a map location.

- `ctx.leaveChat()`: Makes the bot leave the current chat.

> **Note**: You can build a pretty capable bot with just what you've learned so far. The next sections cover more advanced topics!

### Keyboards

Let's explore inline keyboards, which are an incredibly useful feature.

**What are Keyboards?** Inline keyboards are buttons that are attached directly to a message sent by the bot.
**Why use them?** They provide a clean and intuitive way for users to interact, making them ideal for menus, confirmation dialogs, and navigation systems.
**How they work?**. When a user presses an inline button, the bot receives a special `callback_query` update instead of a text message.

**Usage:**
First, we import the `Markup` helper from Telegraf. To create an inline keyboard, we use `Markup.inlineKeyboard()`, which takes an array of arrays. Each inner array represents a row of buttons. We pass this as the second argument to `ctx.reply()` or other `replyWith...` methods.

```js
// ... Previous lines
bot.command("menu", async (ctx) => {
  // Use Markup.inlineKeyboard() to create the keyboard
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

**How to handle a button press?** We use the `bot.action()` method to handle the `callback_query` from a button press. The string inside `bot.action()` must match the `callback_data` you defined in `Markup.button.callback()`. It's good practice to call `ctx.answerCbQuery()` to let Telegram know the button press was received, which stops the loading animation on the button.

```js
//... (bot setup and /menu command from above)

// Handler for the 'song_file' action
bot.action("song_file", (ctx) => {
  // Prevent btn from loading
  ctx.answerCbQuery("Fetching stats...");

  // Send a new message with the stats
  ctx.reply("You select song file!");
});

// Handler for the 'lang_fa' action
bot.action("persion", (ctx) => {
  ctx.answerCbQuery(); // Stop loading
  ctx.reply("you select persion, such a beautifull chose!");
});

bot.launch();
```

**Changing Keyboard Message** : You can edit the original message after a button is pressed using `ctx.editMessageText()`.

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
      .oneTime() // The keyboard disappears after one use
      .resize() // Makes the buttons fit the screen better
  ); 
});
// instead of 'bot.action()' we should use 'bot.hears()' here
bot.hears("angry", async (ctx) => {
  await ctx.reply("oh..so you are angry huh??");
});
```

### Send And Receive Media

A key feature of Telegram bots is their ability to handle media. It's very useful to know how to do this.
To process incoming files, we use the `bot.on()` method with the appropriate message filter. Files are stored on Telegram's servers and are identified by a unique `file_id`. When a file is received, the `ctx` object contains all its information.

You can receive media and access its data like this:

```js
// ... previous lines
// FILE HANDLING
bot.on(message("document"), async (ctx) => {
  const document = await ctx.message.document; // 'ctx' contains file's info
  const document_id = document.file_id; 
  const document_name = document.file_name; 
  const document_size = document.file_size; // files's size in bytes
  console.log(
    `File name is: ${document_name} - size: ${document_size} - id is : ${document_id}`
  );
  await ctx.reply(
    `File name is: ${document_name} - size: ${document_size} - id is : ${document_id}`
  );
});

bot.lauch();
```

You can also send media. Telegraf provides a family of `replyWith...` methods that accept different source types:

- By `file_id` (String): The most efficient method for sending a file that is already on Telegram's servers.

- By URL (String): Telegram will download the file from the public URL and send it.

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
You can "chain" middlewares together using `bot.use()`. This is powerful because each middleware can modify the `ctx` object before passing it to the next handler. This is useful for things like analytics, authorization, or pre-populating data.

Here, we'll create a simple middleware to log response times and add a `role` to the context state. :

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

By default, interactions with a Telegram bot are **stateless**. The bot has no memory of previous messages from a user; each update is processed in isolation. To create meaningful, multi-step conversations, the bot needs to remember data between messages. This is achieved through sessions.

> Note: Because I found these concepts challenging at first, I'll explain them in more detail.

**Terminalogy & Code**

- What are **state** and **session**? A **session** stores data for a specific user or chat. The data is attached to the `ctx` object as `ctx.session` and persists across multiple updates from that user. The data itself is often called the **state**.

Let's build a simple message counter for each user.
```js
const { Telegraf, session } = require("telegraf");
require("dotenv").config;
const bot = new Telegraf(process.env.BOT_TOKEN);

// Use and configure the session middleware
// This uses an in-memory store by default
bot.use(
  session({
    initial: () => ({ counter: 0 }), // Initial session data
  })
);

bot.on("text", async (ctx) => {
  ctx.session.counter++; // Increment the counter for this user
  await ctx.reply(`Message counter is: ${ctx.session.counter}`);
});

bot.launch();
```

This is sufficient for testing, but all session data is lost whenever the bot restarts. For data persistence, we can use a package like `telegraf-session-local`. It stores session data in a local **JSON file**.

First, install it: `npm install telegraf-session-local`

```js
const { Telegraf } = require("telegraf");
// Import 'telegraf-session-local'
const LocalSession = require("telegraf-session-local");

const bot = new Telegraf(process.env.BOT_TOKEN);

// Configure the session to use a local JSON file for storage
const localSession = new LocalSession({ database: "session_db.json" });
// Use our session middleware
bot.use(localSession.middleware());

bot.on("text", async (ctx) => {
  ctx.session.counter = ctx.session.counter || 0;
  ctx.session.counter++; //keep trach of msg numbers for each user

  await ctx.reply(`Message Count: ${ctx.session.counter}`);
  return next();
});

// Command to clear session data for a user
bot.command("remove", async (ctx) => {
  ctx.session = null; // Setting session to null removes it
  await ctx.reply("Session data removed.");
});
```

> For production bots, you might use a database like Redis or MongoDB for session storage. I haven't learned that yet, but if you're familiar with it and have some free time, feel free to contribute a section!

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

With that in mind, let's walk through the code for a simple registration wizard:

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

### Personal Projects

Now that we've learned the building blocks, it's time to create something and learn by doing. The funny thing is, you won't truly learn until you try, fail, and fix things. So go ahead and try to build something!

## Project Idea: An AI ChatBot 🤖 (a basic one!)

Let's build something fun! We'll create a simple chatbot that uses Google's Gemini AI to respond to user messages. This is a great way to see how easily you can add powerful AI capabilities to your bot.

## 1.What You'll Need
- A **Telegram Bot Token** (you already have this from the first section).
- A **Google AI API Key**. You can get one for free from [Google AI Studio](https://aistudio.google.com/app/apikey). Just create a new API key in your project.
## 2.Project Setup & Installation
Create a new folder for your project and initialize it:
```bash
mkdir ai-bot
cd ai-bot
npm init -y
```
Now, let's install the necessary packages: `telegraf` for the bot, `@google/genai` for the AI, and `dotenv` to manage our secret keys.
```bash
npm install telegraf @google/genai dotenv
```
Next, create a `.env` file in your project folder to store your keys securely. **Never** share this file or commit it to Git.
```Ini
# .env file
BOT_TOKEN="YOUR_TELEGRAM_BOT_TOKEN_HERE"
GEMINI_API_KEY="YOUR_GOOGLE_AI_API_KEY_HERE"
```
## 3.The Code
Create a new file called `bot.js`. Here is the complete code with explanations.

```js
// load the .env file
require("dotenv").config();
// import telegraf
const { Telegraf } = require("telegraf");
// import gemini
const { GoogleGenAI } = require("@google/genai");
// --- CONFIGURE GEMINI ---
const ai = new GoogleGenAI(process.env.GEMINI_API_KEY);

// create bot
const bot = new Telegraf(process.env.BOT_TOKEN);
// a handler for /start command
bot.start(async (ctx) => {
  await ctx.reply("Send me a message");
});
// handler for text message
bot.on("text", async (ctx) => {
  const userInput = ctx.message.text;
  try {
    // show typing indicator to the user
    await ctx.telegram.sendChatAction(ctx.chat.id, "typing");
    // USE AI
   const geminiResponse = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: userInput,
    config: {
      thinkingConfig: {
        thinkingBudget: 0, // disable thinking ability - more spped and less token usage
      }
    }
   })
    ctx.reply(geminiResponse.text);
  } catch (error) {
    console.error("AI error: ", error);
    ctx.reply("Sorry, ai is gone at the moment, do the thinking yourself...");
  }
});
// launch the bot
bot.launch();
console.log("The Bot is Running...");
// stop handling
process.once("SIGINT", () => bot.stop("SIGING"));
process.once("SIGTERM", () => bot.stop("SIGTERM"));

```
`sendChatAction`: This provides a great user experience, showing that the bot has received the message and is "thinking."

## Song Lyrics Project
Soon...