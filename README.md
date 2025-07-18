# Telegraf.js Tutorial 

This tutorial is for absolute beginners who want to create a Telegram bot using [Telegraf.js](https://telegraf.js.org/). Whether you're new to coding or just new to bots, we’ll walk through the process step-by-step to get you started!

### Why Telegraf.js and Not [node-telegram-bot-api](https://github.com/yagop/node-telegram-bot-api)?
There are several reasons to choose Telegraf.js, but the key one, especially for beginners like us, is its **middleware-based architecture** (think Express.js or Koa).

### Steps
- [Creating a New Bot with BotFather](#1-get-your-bot-token-from-botfather)
- [More steps coming soon as we build this tutorial together!]

### 1-Get Your Bot Token from BotFather
The first step is to create your bot using BotFather:

1. Open Telegram and search for the official [@BotFather](https://t.me/BotFather) account (look for the blue checkmark).
2. Start a chat and send the `/newbot` command (or select it from the menu).
3. Choose a name for your bot, then pick a unique username ending in `bot` (e.g., `@MyUniqueTestBot`).
4. If the username is available, BotFather will provide a **token**.
5. Copy the token and save it securely (e.g., in a `.env` file for later use).

<img src="https://github.com/daniyal-abbassi/telegrafjs-starting-tutorial/blob/main/screenshots/botfather.png" height="500" width="390">

> **Tip**: Never share your bot token publicly! It’s like a password for your bot.