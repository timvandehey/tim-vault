Pushover
Pushover is a reliable service designed specifically for push notifications. While not self-hosted, it's a very popular choice for its simplicity and reliability.

How it works: You get a Pushover account and install their app on your Android device. You then use the n8n Pushover node to send notifications via their API.

Pros:

Dedicated n8n node: The integration is built-in and easy to configure.

Reliable and fast: Designed for quick delivery.

Device-specific targeting: You can send notifications to specific devices or groups.

n8n Implementation:

Add a Pushover node to your workflow.

Create a Pushover credential and enter your API token and user key.

In the node settings, specify the message, and optionally a title, priority, and sound.