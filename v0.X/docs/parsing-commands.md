# Parsing the commands

## Ignoring commands addressed to other bots in group chats

For fetching the messages, use `bot.nextMessageSync()` helper method. It will return the fetched message and will 
 also set `bot.lastMessage` to the last fetched message and `bot.lastCommand` to the last user command, if any.

If `bot.lastCommand is nil`, then the message is a file attachment or another media type.

> When instance of TelegramBot class is constructed, it issues `/getMe` request and fetches the bot's name. This name is then used to distinguish this bot's commands from other bots in group chats.

Messages addressed to other bots will be silently ignored unless you pass `onlyMyMessages: false` flag to `nextMessageSync()`.

## Constructing the router

Router maps commands to their handler functions.

```swift
let router = Router(bot)
router["command1"] = handler1
router["command2"] = handler2
...
router.process(command)
```

If user passes more arguments than command handler expects, a warning will be issued. You can customize this behavior by passing your own partial match handler to router:

```swift
    func partialMatchHandler(args: Arguments) -> Bool {
        bot.respondAsync("Part of your input was ignored: \(args.scanRestOfString())")
        return true
    }
    router.partialMatch = partialMatchHangler
```

So, for example, if we have a command `swap` which expects two arguments, but user types: `/swap aaa bbb ccc`, he will see: 
```
bbb aaa
Part of your input was ignored: ccc
```

If the command is not found, a warning will also be issued. This also can be customized:

```swift
func unknownCommandHandler(args: Arguments) -> Bool {
    guard bot.lastMessage.chat.type == .privateChat else { return false }
    guard session.started else {
        bot.respondAsync("Bot is inactive, please type /start")
        return true
    }		
    bot.respondAsync("Unknown command. /help")
    return true
}
router.unknownCommand = unknownCommandHandler
```

Of course, you can also pass closures or class methods to router.

## Adding paths to router

Any bot typically processes at least these 4 commands:

```swift
router["start"] = controller.start
router["stop"] = controller.stop
router["help"] = controller.help
router["settings"] = controller.settings
```

Command name is processed differently in private and group chats.

* In private chats slash is optional. `start` matches `/start` as well as `start`.
* It group chats it only matches `/start`.

If you don't want optional slash in private chat, pass a `slash` option:

```swift
router["mycommand", slash: .Required] = controller.myHandler
```

## Commands with arguments

Words can be captured and then processed by using a `scanWord` method.

```swift
router["process_word"] = controller.processWord
...
class Controller {
    func processWord(args: Arguments) {
        guard let word = args.scanWord() else {
            bot.respondAsync("Expected argument")
            return
        }
        bot.respondAsync("You said: \(word)")
    }
}
```

To split string into many words, use `args.scanWords()`.

```swift
let words = args.scanWords()
for word in words { ... }
```

Check `Arguments.swift` source code for more parsing options. Internally it uses NSScanner, so it's methods can be used as well. Make sure to fetch all the arguments, otherwise partialMatchHandler will come into play.

There's also `args.scanRestOfString()` method which captures rest of string as a single string.

## Handler functions

Any function, class method or a closure can be used as a handler.

The following signatures are supported:

```swift
() -> ()
```
Simple handler. Treats the command as processed.

```swift
() -> Bool
```
Cancellable handler. Return `true` if the command was processed, `false` to continue matching other rules.

```swift
(args: Arguments) -> ()
(args: Arguments) -> Bool
```
Handler taking the captured arguments. Extract arguments by calling corresponding methods of `args`.

```swift
() throws -> ()
() throws -> Bool
(args: Arguments) throws -> ()
(args: Arguments) throws -> Bool
```
Throwing versions of the above. These just pass the exception your code raises to the top. Surround `router.process(command)` with `do {} catch {}` if you use them.

More details are available in Router's documentation (to be done, please check the SDK source until then).

[Back to Wiki Home](index.md)