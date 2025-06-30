# Webhooks

Oracle Digital Assistant (ODA) uses an **asynchronous messaging model** for sending and receiving messages. This model is essential when you’re integrating with external messaging clients or services. While ODA includes native support for several built-in channels (like Facebook, Slack, etc.), you can use **webhooks** to integrate with any third-party messaging client that isn't supported out of the box.

Webhooks allow your messaging client to send and receive messages to and from your ODA instance over HTTP. This flexibility makes them ideal for custom integrations.

## Quickstart

If you're just getting started, check out the [Webhook Starter Example](https://github.com/oracle/bots-node-sdk/blob/master/examples/webhook/starter). It includes ready-to-run code using Node.js and Express, and shows how to connect your client to Oracle Digital Assistant through webhooks.

> 💡 Tip: You can use tools like [Beeceptor](https://beeceptor.com/) or [Pipedream RequestBin](https://pipedream.com/requestbin) to simulate your webhook endpoints during development and debugging. These tools are helpful for inspecting incoming requests without spinning up a full server.

---

## Webhook Client

Oracle provides a `WebhookClient` through the `@oracle/bots-node-sdk` package. This middleware helps you connect your bot to any client over HTTP using webhooks.

Here’s a basic Node.js Express example that shows how to:

- Receive messages from the bot
- Send messages to the bot
- Handle errors

```javascript
const express = require('express');
const OracleBot = require('@oracle/bots-node-sdk');

const app = express();
OracleBot.init(app);

// Load webhook middleware
const { WebhookClient, WebhookEvent } = OracleBot.Middleware;

// Configure webhook channel details (from ODA UI)
const channel = {
  url: process.env.BOT_WEBHOOK_URL,
  secret: process.env.BOT_WEBHOOK_SECRET,
};

// Initialize WebhookClient
const webhook = new WebhookClient({ channel });

// Error handling
webhook.on(WebhookEvent.ERROR, console.error);

// 1. Receive messages from bot
app.post('/bot/message', webhook.receiver());
webhook.on(WebhookEvent.MESSAGE_RECEIVED, (message) => {
  // Process the message and respond to your messaging client
  console.log('Bot sent message:', message);
});

// 2. Send messages to bot
app.post('/user/message', (req, res) => {
  const userMessage = {
    userId: 'user123',
    messagePayload: {
      type: 'text',
      text: 'Hello from user!',
    },
  };

  webhook.send(userMessage)
    .then(() => res.send('Message sent to bot'))
    .catch((e) => res.status(400).send('Error sending message'));
});
```

### Additional Notes

- `webhook.send()` also accepts a second optional `channel` argument for per-request channel selection.
- For verbose logging, you can pass a logger during SDK init:

```javascript
OracleBot.init(app, {
  logger: console,
});
```

---

## Webhook Utilities

If you need more control over the integration — for example, to send messages without using middleware — the SDK provides low-level utilities.

The `webhookUtil` module exposes functions to manually send messages to the bot:

```javascript
const { webhookUtil } = require('@oracle/bots-node-sdk/util');

const webhookUrl = 'https://your.bot.url';
const secret = 'your-webhook-secret';
const userId = 'user-abc';
const message = {
  type: 'text',
  text: 'Hi from webhookUtil!',
};

const extras = {
  profile: { firstName: 'Test' },
  context: { variables: { myVar: '123' } },
};

webhookUtil.messageToBotWithProperties(
  webhookUrl,
  secret,
  userId,
  message,
  extras,
  (err, result) => {
    if (err) {
      console.error('Error:', err);
    } else {
      console.log('Bot responded:', result);
    }
  }
);
```

This method is especially useful when integrating from a service that doesn’t support Express middleware, or when you want to programmatically trigger bot interactions.

---

## Debugging Tips

| Tool                  | Purpose                                         | Link                                                                 |
|-----------------------|-------------------------------------------------|----------------------------------------------------------------------|
| Beeceptor             | Inspect and test incoming/outgoing HTTP calls  | [https://beeceptor.com](https://beeceptor.com)                       |
| Pipedream RequestBin  | Temporary endpoints for webhook development     | [https://pipedream.com/requestbin](https://pipedream.com/requestbin) |

These tools let you verify that your ODA webhook integration is sending and receiving messages correctly — without deploying a backend server.

---

## References

- [Using Webhooks with Oracle Digital Assistant](https://www.oracle.com/pls/topic/lookup?ctx=en/cloud/paas/digital-assistant&id=DACUA-GUID-96CCA06D-0432-4F20-8CDD-E60161F46680)
- [Webhook Starter Code on GitHub](https://github.com/oracle/bots-node-sdk/blob/master/examples/webhook/starter)
