# API introduction

## Intro

SDK hides most of the complexity from you. It fetches messages, parses them, constructs responses, handles errors and reconnections and so on. The defaults should be suitable in most cases. There's also a lower level API.

In a typical bot you just receive a command and react to it somehow. All else is done by the library.

If multiple bots are in a group, users can add bot usernames to commands in order to avoid confusion, for example: `/command@nameofmy_bot`. This is also handled by the library, you receive the command with bot username deleted from it.

So, `/command@nameofmy_bot argument` becomes `/command argument`. Commands addressed to other bots are ignored.

Router class can be used for splitting a command into parts and calling appropriate handler functions. More on it in [Parsing the commands](parsing_commands) section.

> Bot fetches it's username from Telegram by issuing a `/getMe` request on start.

## Sync and async methods

Most API methods have two versions: synchronous and asynchronous. They are distinguished by their suffix, for example: `respondSync` and `respondAsync`.

**Synchronous** methods block until the operation is completed.

**Asynchronous** methods accept an optional completion handler which will be called when the operation is completed.

For example, to send two messages to user guaranteeing the order in which they arrive you can do either:

```swift
bot.respondSync("Message 1") // blocks until the message is sent
bot.respondSync("Message 2")
```

or

```swift
bot.respondAsync("Message 1") {
    bot.respondAsync("Message 2") {
    }
}
// execution continues immediately
```

It's better to use the second approach because the server can process and respond to other messages and won't block waiting for confirmation.

Completion handler is called on the main thread by default.

For simplicity, it's possible to synchronously process messages, but respond asynchronously to avoid blocking the processing of the next message. So, a typical bot's main loop looks like this:

```swift
while let message = bot.nextMessageSync() {
    if let command = bot.lastCommand {
        bot.respondAsync("Hi \(bot.lastMessage.from.firstName)! You said: \(command).\n") 
    }
}
```

[Next: Parsing the commands](parsing-commands.md)