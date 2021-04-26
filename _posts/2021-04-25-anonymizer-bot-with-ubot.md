---
title: A simple anonymizer bot with uBot
toc: false
tags: git github telgram bot api golang documentation example go
category: development
classes: wide
---

This is what it takes to implement an anonymizer bot with **uBot**. 

The bot will echo back any message on a private chat, ignoring only _/start_ command, please note that this behaviour is achived just by composing 
existing matchers.

The bot will also that intercept and handle properly any interruption signal by calling a cancel function on the context.

## Code
```golang
// A simple anonymizer bot that echoes back any message it gets in a private chat, so that you can forward it without exposing the original sender.
package main

import (
	"context"
	"flag"
	"log"
	"os"
	"os/signal"
	"sync"

	"github.com/sdurz/axon"
	"github.com/sdurz/ubot"
)

var (
	apiKey  string
	signals chan os.Signal
)

func init() {
	flag.StringVar(&apiKey, "apikey", "", "api key")
	flag.Parse()

	signals = make(chan os.Signal, 1)
	signal.Notify(signals, os.Interrupt)
}

func main() {
	var (
		wg  sync.WaitGroup
		bot *ubot.Bot
	)
	bot = ubot.NewBot(&ubot.Configuration{APIToken: apiKey})
	bot.AddMessageHandler(
		ubot.And(ubot.MessageIsPrivate, ubot.Not(ubot.MessageHasCommand("/start"))),
		func(ctx context.Context, b *ubot.Bot, message axon.O) (done bool, err error) {
			messageId, _ := message.GetInteger("message_id")
			chatId, _ := message.GetInteger("chat.id")
			bot.CopyMessage(axon.O{
				"chat_id":      chatId,
				"from_chat_id": chatId,
				"message_id":   messageId,
			})
			return
		})

	ctx, cancel := context.WithCancel(context.Background())
	go bot.Forever(ctx, &wg, ubot.GetUpdatesSource)
	wg.Add(1)
	<-signals

	cancel()
	wg.Wait()
	log.Println("done with main")
}
```

## Usage
Just start it with your api key as a parameter:
```bash
$ ./anonymizer -apykey <yourapikey>
```

As usual, you can get the code on [GitHub](https://github.com/sdurz/anonymizerbot), please remember that **uBot** is still pre alpha and your contibutions are always welcome.