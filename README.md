<h1 align="center">
    <img src="https://i.imgur.com/wXp9xwZ.png"
        height="50" width="50">
    ShulkerRoadCrypt™ <h3>End-To-End File & Message Encryption For Discord, forked from DiscordCrypt</h3>
</h1>

![Encrypted Message](https://i.imgur.com/ItVOlWd.png)

Discord message encryption plugin that enables client side end-to-end  encryption for your messages and files with automatic key exchange, works without BetterDiscord.


# Installation

For `Chrome` (and similar)
[download the extension](https://chrome.google.com/webstore/detail/shulkerroadcrypt/bgheoljmdcaomnakjefkdcpjeocgbgmo) from Chrome Webstore <br>
Or [download the dev extension](https://github.com/HyCore/ShulkerRoadCrypt/archive/refs/heads/main.zip) and unpack it. Click on **Menu > More tools > Extensions**, enable **Developer mode**, click on **Load unpacked** and select the folder where you've unpacked the downloaded file.<br>
If you have `Discord installed`, [download the installer](https://raw.githubusercontent.com/HyCore/ShulkerRoadCrypt/main/ShulkerRoadCrypt.ps1). Click the link and press **Ctrl+S** to save it as file. Launch Powershell as administrator, enter this command: `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` and enter the path to the file to execute it (i.e. `cd C:\Users\You\Downloads\ShulkerRoadCrypt.ps1`).<br>
For `mobile` try [Yandex Browser](https://play.google.com/store/apps/details?id=com.yandex.browser). The browser is Chromium based and supports extensions.<br>
For `Android` install [Discord APK](https://auriane.eu/pages/img/SRDiscord.apk) and be sure to watch the tutorial. <br>
Alternatively install it as [userscript](https://github.com/HyCore/ShulkerRoadCrypt/raw/main/ShulkerRoadCrypt.user.js) (with [Tampermonkey](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo)) or include the js file somehow else.

# Features

- supports Windows, MacOS and Linux. 
- all messages in DMs, group DMs and channels can be configured to be end-to-end encrypted
- uses a dual encryption mode with multiple ciphers (defaults to AES-256 and Camellia-256)
- emojis (including animated) are supported even without Nitro
- encrypted and password protected database to store all keys
- ability to backup, restore and manage all key in the database 
- strong and automatic key exchanges for DMs allowing you to easily secure sessions with your friends
- supports encrypted file uploads up to 50 MB which are automatically deleted after 7 days
- automatic updates that ensure having the latest features while staying secure
- blocks various known tracking elements used by Discord via SecureDiscord
- designed to provide the maximum security and privacy
- extensively tested for reliability and stability

# How to use

`Open the menu` by right clicking on the **lock icon**.

`Select` the key you want to use in the drop down menu on top, next to the **lock icon**.

`Toggle encryption` by clicking the **lock icon** or use the prefix `:ENC:` or `ENC` at the start of your message to force  encryption. To send non encrypted messages in a channel where encryption is turned on, use the prefix `NOENC`.

`Download an encrypted image` by middle clicking on the image. 

`Switch devices` by exporting and importing your database. If you would like to use the plugin on multiple devices choose a main one, export the database (right click on **lock icon > Export Database**), right click on the **lock icon** (in on the secondary device) **> New Database > Secondary**). Repeat this after adding new keys in your primary device. 

`ECDH P-521` is used for the key exchange and `AES256-CBC` for the messages offering the equivalent security of 256-bits

[Here](https://gitlab.com/leogx9r/DiscordCrypt) you can find the original DiscordCrypt (discontinued). 

If you have any questions come to my [Discord server](https://discord.gg/kits)

# Database

The **database password** is optional but the database isn't! The database stores your keys and channel settings.

It is recommended to have only one database, multiple databases will mess things up. In order to manage this, you should `keep backups` by exporting your database regularly (right click on **lock icon > Export Database**).

If you would like to use the plugin on multiple devices choose a main one, export the database from that and click on **secondary** when importing. Repeat this after adding new keys.

If you import the database as secondary you can still export it and import it as primary database again. The difference is that a device with a secondary database will ignore key exchanges done on this device, which means you have to update your secondary devices with the new database.

##### Cleaning the database 

<sub>You can paste these into the `Ctrl+Shift+I` console (Sadly with last chrome update it became maybe deprecated)</sub>
<details><summary>SdcClearChannels(filterFunc)</summary>

```js
deleteBefore = (now = new Date()).setMonth(now.getMonth() - 6);
SdcClearChannels((channel) => (
    //Number(channel.lastseen) ms precision unix timestamp
    //String(channel.descriptor) descriptor from the channel manager
    //Boolean(channel.encrypted) is the encryption toggled on

    channel.lastseen < deleteBefore && //not seen in 6 months
    /^DM with \d{17,20}$/.test(channel.descriptor) //name resolution failed

    //'true' return value deletes the record
));
```
</details>
<details><summary>SdcClearKeys(filterFunc)</summary>

```js
deleteBefore = (now = new Date()).setMonth(now.getMonth() - 6);
SdcClearKeys((key) => (
    //Number(key.lastseen) ms precision unix timestamp
    //String(key.descriptor) descriptor from the key manager
    //Boolean(key.hidden) is the key hidden
    //String(key.type) one of ['GROUP', 'CONVERSATION'/*DM*/, 'PERSONAL']
    //Number(key.registered) when the key was added to the database

    key.lastseen < deleteBefore && //not seen in 6 months
    /^(?:DM key with \d{17,20}|\d{17,20}'s personal key)$/.test(key.descriptor)
    //name resolution failed if the id is used as name

    //'true' return value deletes the record
));
```
</details>

# Keys
        
**Keys** are used to encrypt messages. Additional to that there is **database key** for encrypting the plugin's local database.
You can `select` which key to use for the channel at the top, besides the lock icon.

## Key Types
- **Personal key:** every user has a personal key (that is different for other users). It is used to encrypt messages in channels where are no other options available. It is shared with everyone you key-exchange with, so the security of it varies depending on the number of people you share it with.
- **Group key:** You can create a group key if you are in a group channel. Right click on the **lock icon > Create Group Key**. It is advised to change the name of the key right away to spot if someone accidentally generated a different key for your channel. You can do this in the **Key Manager**. After generating the group key, you can use it in other channels as well. If you set a group key in the **Key Manager** as **hidden** it will no longer show up in the drop down and it won't be automatically shared with your friends. If you really want to know if the other is using the same key as you, compare the first few characters (up to 16) of the encrypted message.
- **DM key:** These keys are generated by a Diffie-Hellman key exchange between two users, it provides a secure connection over an unsafe medium.<br>

# Key Exchanges

If the plugin comes across a message with a key that you don't have it will attempt to get that key by a key exchange with the sender of the message. You need to be able to DM the other user for the key exchange.

If a group key isn't set as **hidden** it will  be shared automatically with a **discord friend** if they ask for it. (key exchange)

In order to share a key you need to establish a secure connection with an initial key exchange, it is usually automatic or can be started manually by right clicking on the **lock icon > Start Key Exchange** in the DM channel.

You can manually share keys by using the **Share Keys** menu option. During a key share, the sharing party will suggest you channels to use so it's advised to **not** set anything for the channel (or toggle encryption) before you have received the key for it.

# Key Rotation and Trusted Keys

Key rotation periodically generates a new key and switches to it in all channels that use the rotated key. Use `SdcSetKeyRotation(7)` in the console command to start a key rotation on the key of the current channel. 

In the example above , the key changes every 7 days. By default this is aligned to arbitrary points so if you use the same command on your other primary devices (within a day, it will do the same). If the key is set to **hidden**, the new key will be hidden too.

**Trusted Keys** can be enabled in the **Key Visualizer** (in DM), this feature can be used to do **automatic key exchange** for hidden keys and/or rotated keys.

Keys shared with `trusted keys` can swap keys for channels automatically, which is **useful for rotated keys**.

# Extras

You can click on an already enlarged image to further enlarge it, you can also drag the enlarged image if it doesn't fit the screen. Use the **arrow keys** to navigate like a gallery when zoomed in. You can use emojis (static and animated) in encrypted messages.

Encrypted messages above 1600 characters will be compressed, but this feature might have incompatibilities with future versions. This also means that the not encrypted messages can be 2000 character long if they are mostly made of normal characters.
For large audio and video files, you can use [Mega](https://mega.io/), it automatically embeds.

The encryption is completely secure unless someone at Discord replaces your messages right when you secure your DMs.
    To make sure you have the right keys, you can use the **Key Visualizer** and compare your result with the other party.

<details><summary><b>Getting notifications on custom phrases</b></summary>

This feature should be a good compensation for no searches and role mentions. (You use the console to tell you what can ping you or not).
Use the `SdcSetPingOn(regexStr)` console command to set the match string or regular expressions (regex) for extra pings.
    
For example `SdcSetPingOn('An0')` will only ping if the message contains that **exact word**, `SdcSetPingOn(/\bAn[0o]\b/i)` will match **every form of it**

**Regex explanation:** 
- `/An0/` only matches this exact expression
- `/An[0o]/` will match `An0` and `Ano`
- `/An[0o]/i` will be case insensitive, so it matches `ano` and `ANO` and everything in between`
- `\b` means word border, which means "start or end of word", so `/\bano\b/` won't find a match in the word 'another' (only works if you use letters, numbers or '_')
`/(?:An0|SimpleDiscordCrypt|SDC)/` will match **any of the three** smaller regexes between the `(?:)`
Please note that characters like `.\\+?*()[]$^` etc. have a specific function in regex and should be escaped with `\`.

<b>Sample regex for role mentions:</b>
`/<@&(?:473998238156849152|474006463749160992)>/`
You can get the role ids with `\@Role` in the chat

</details>

To uninstall, just delete it from %localappdata% and make new shortcuts.

# Credit

Thanks to EnigmA_008#1505 for her help, she's an angel ! <br>
ShulkerRoadCrypt by !!ExoRam!!#9430 forked from DiscordCrypt & SimpleDiscordCrypt
