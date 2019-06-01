# MessageBLocker: A middleware for EFB 

## Notice

**Middleware ID**: `catbaron.message_blocker`

**MessageBLocker** is a middleware for EFB, to manage filters and block some specific messages. 

Be aware that this is a very early develop version. Please let me know if you found any problem.

You need to use **MessageBlocker** on top of [EFB](https://ehforwarderbot.readthedocs.io). Please check the document and install EFB first.

## Dependense

* Python >=3.6
* EFB >=2.0.0
* peewee
* PyYaml

## Install

* Install
```
git clone https://github.com/catbaron0/efb-msg_blocker-middleware
cd efb-msg_blocker-middleware
Python setup.py install # You may need su permission here
```
* Register to EFB
Following [this document](https://ehforwarderbot.readthedocs.io/en/latest/getting-started.html) to edit the config file. The config file by default is `~/.ehforwarderbot/profiles/default`. It should look like:
```
master_channel: foo.demo_master
slave_channels:
- foo.demo_slave
- bar.dummy
middlewares:
- foo.other_middlewares
- catbaron.message_blocker
```

You only need to add the last line to your config file.

* Restart EFB.

## How it works
MesageBlocker support three types of filter:

1. `user`:  The unique ID of a user in a chat. It's useful when you want to block some users in a group.
2. `type`: Block some specific types of messages. Support all the `MsgType` defined by `EFB`, including `image`, `audio`, `file`, `link`, `location`, `status`, `sticker`, `text`, `video`, and `unsupported`.
3. `text`: Block messages matched by a Regex string. Only effective for messages of `text` type.

A `filter` is a dictionary with at least one of three keys: `user`, `type` and `text`. Messages matched by all the provided filters will be blocked.

For instance, here is a filter:
```
{
    "type": ["image", "video"], 
    "user": "e5141de3"
}
```
This filter matches `image` or `video` messages that are sent by the user whose id is `e5141de3`.

## Usage
Three commands are supported by this middleware.

* `\list`: List all the filters you have added to one chat. The EFB will reply to you with a list of `json` string. e.g.:

    ```
    {'id': 1, 'author_channel_id': 'blueset.wechat', 'author_channel_name': 'WeChat Slave', 'author_chat_name': '', 'author_chat_alias': 'None', 'chat_chat_uid': 'c00003', 'chat_chat_name': '', 'chat_chat_alias': 'None', 'filter_text': '{"type": ["image"], "user": "12345"}'}
    ```

    `id` is the unique ID for this filter, and `filter_text` is the content of this filter. The others are information about chat where the filter is active.
    
    You can reply `\list` to a message to check all the filters you added to the author of message.


* `\del {id}`: Delete a filter with filter `id`. (e.g. `\del 1`). You can delete all the filters you added to a user by replying `\del` to his/her message.
* `\add {arg}`: Add filters. There are some ways to add filters.
    
    * `arg` is one of any  supported `type`, so that all the messages in the specific type will be blocked. For example `\add image` adds a filter to block all the image messages.
    * `arg` is `filter_text`, like `\add {"type": ["image"]}`
    * You cloud reply `\add {arg}` to a message  in order to block messages from one user. Foe example, after you reply `\add {"type": ["image"]}` to a message sent by user whose `id` is `12345`, it's equivalent to `\add '{"type": ["image"], "user": "12345"}`. It's convinent especially when you don't know the id of a user.

