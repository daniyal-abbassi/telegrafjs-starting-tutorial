# telegraf.js Tutorial For absolute beginners

This tutorial is for absolute beginners who wants to create a telegram bot using [telegraf.js](https://telegraf.js.org/).

### Why telegraf.js and not [node-telegram-bot-api]((https://github.com/yagop/node-telegram-bot-api)) ?
There are multiple reasons which I don't dive in to, BUT the most important feature (specially for new learners like myself) is that the core Architecture of ```telegraf.js``` is **Middleware-based** (like Express/Koa) which is more handy and more familier to use.

### Steps
- [Creating new Bot with BotFather]


### step-1: Get our Token from BotFather 
The first step well... is to actually create a bot with botFather:
- Open Telegram and Search for @BotFather account(notice blue checkmark), and start a new chat.
- type ```/newbot``` command (or simply choose it with menu options)
- Choose a Name and then Choose a Username (uniqe, and ends with bot) e.g. ```my_uniqe_test_bot```
- If our chosen name is available, BotFather will give us a Token
- Copy and Save the token (for later use save it in ```.env``` file )

