---
title: A simple anonymizer bot with uBot
toc: false
tags: git github telgram bot api golang documentation example go
category: development
---

This is what it takes to implement an anonymizer bot with **uBot**:

```golang
package main

import (
	"context"
	"flag"
	"sync"

	"github.com/sdurz/axon"
	"github.com/sdurz/ubot"
)

var (
	apiKey string
)

func init() {
	flag.StringVar(&apiKey, "apikey", "", "api key")
	flag.Parse()
}

func main() {
	var (
		wg  sync.WaitGroup
		bot *ubot.Bot
	)
	bot = ubot.NewBot(&ubot.Configuration{APIToken: apiKey})
	bot.AddMessageHandler(ubot.MessageIsPrivate, func(ctx context.Context, b *ubot.Bot, message axon.O) (done bool, err error) {
		messageId, _ := message.GetInteger("message_id")
		chatId, _ := message.GetInteger("chat.id")
		bot.CopyMessage(axon.O{
			"chat_id":      chatId,
			"from_chat_id": chatId,
			"message_id":   messageId,
		})
		return
	})
	bot.Forever(context.Background(), &wg, ubot.GetUpdatesSource)
}
```

Just start it with your api key as a parameter:
```bash
$ ./anonymizer -apyKey <yourapikey>
```

As usual, you can get the code on [GitHub](https://github.com/sdurz/anonymizerbot), please remember that **uBot** is still very mych work in progress and your contibutions
are always welcomed.