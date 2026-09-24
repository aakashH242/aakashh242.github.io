---
title: ChatGPT app missed this
description: An update checker to keep the ChatGPT linux app up-to-date
---

I am a regular user of Codex via the ChatGPT desktop app for Linux. After installing it, I checked its menus but could not find a "Check for updates" button like the
Windows desktop app has. I was like - meh, I guess they auto-update. 

Once GPT-6 Astra was released, I was surprised that I could not find it in my Codex app. A quick look at the forums told me to tell Codex to update the app.
When I asked Codex to do so, I was shocked so see I was quite a few versions behind! I was frustrated that I was never notified of the updates or could not 
manually check them via the app. Also, turns out I did not have an auto-updater installed and KDE's updated didn't catch ChatGPT.

With a bit of help from Codex, I ended up writing a [small utility](https://github.com/aakashh242/chatgpt-update-notifier) that will notify you whenever there is a ChatGPT app update available.
It is Linux only and tested on every Linux flavor. 

Installation it with the below commands:

```bash
curl -fsSLo chatgpt-update-notifier-install.sh https://raw.githubusercontent.com/aakashH242/chatgpt-update-notifier/refs/heads/main/install.sh &&
bash chatgpt-update-notifier-install.sh
```
