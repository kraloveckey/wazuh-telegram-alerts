# Integrating Telegram with Wazuh

This integration allow to send alert in Telegram from Wazuh Server.

## Creating the bot and getting information

To send Wazuh alerts to a Telegram chat, we need to create a bot first. To do this we have to send a couple of messages to `@BotFather`. After starting the bot with the `/start` command, we have to send the `/newbot` command to start creating the bot, and we will choose the name of the bot, ex. Wazuh.

Once the bot is ready, we can write the script that will send the Wazuh alerts. First, we need the `Chat ID`, this is the identifier of the conversation we are having with the bot. 

Before you find this ID, send your bot a small "hello" message so that a chat can be created between you or add the bot to Telegram group and add `@my_id_bot` into chat to get CHAT_ID .

```shell
https://api.telegram.org/bot<YOUR-BOT-TOKEN>/getUpdates
```

You will get a similar result to this one:

```shell
{"ok":true,"result":[{"update_id":421416869,"message":{"message_id":2,"from":{"id":12345678,"is_bot":false,"ﬁrst_name":"xxxxxx","last_name":"xxxxxx","username":"xxxxxx" ,"language_code":"en"},
```

With the token given by `BotFather` and `Chat ID` we just got, we have all the necessary information for the script.

## Writing the script

To solve this problem it is important to understand two things:

- Local configuration: `/var/ossec/etc/ossec.conf` - where the Wazuh configuration located.
- Integration location: `/var/ossec/integrations/` - where your integrations are located.

To keep things simple, after you create an integration, you should let the local configuration know that this custom integration exists by referencing it.

The first thing to do will be installing the `requests` package using `pip3` to send requests to the Telegram servers:

```shell
$ pip3 install requests
```

The python script will look like [custom-telegram](./custom-telegram). Copy it and paste to `/var/ossec/integrations/custom-telegram`.

```shell
$ nano /var/ossec/integrations/custom-telegram
```

Let's take a look at it step by step.

- First, the `#!/usr/bin/env python3` is used to indicate that the script should be executed using Python version 3.x.
- `UTF-8` encoding is set to handle characters in the code.
- The necessary modules are imported: `sys`, `json`, and, if available, `requests`.
- The `CHAT_ID` variable is defined, which represents the chat identifier from Telegram.
- The `create_message` function is created, which generates a message based on the input data.
- Read configuration parameters from command line arguments: path to the file with alert data and address for sending notifications.
- Reading an alert from a file and converting it from JSON to a Python object.
- Generating a message using the `create_message` function.
- Sending a request with the generated message to a specified address using the `requests` module.
- Output debugging information to a file.

> The integration name must start with `custom-`, otherwise Wazuh won't understand what you want it to do.

## Configuring Wazuh to send alerts to Telegram

After writing previous Python script, we have to copy it to the machine where the Wazuh manager is installed, in this folder: `/var/ossec/integrations/`. Now, let’s give the script the corresponding permissions and user:

```shell
$ chmod 750 /var/ossec/integrations/custom-telegram
$ chown root:wazuh /var/ossec/integrations/custom-telegram
```

After doing this, the last step will be to add the integration configuration in the `ossec.conf` file (with `<YOUR-BOT-TOKEN>` being your token given by BotFather) and the Telegram integration will be ready:

```shell
$ nano /var/ossec/etc/ossec.conf

...
  <!--Telegram-->
  <integration>
      <name>custom-telegram</name>
      <level>5</level>
      <hook_url>https://api.telegram.org/bot<YOUR-BOT-TOKEN>/sendMessage</hook_url>
      <alert_format>json</alert_format>
  </integration>
...
```

After this, restart the Wazuh manager and the integration will be working.

```shell
$ systemctl restart wazuh-manager.service
```

You’ll receive messages like this.

```shell
Agent: AGENT_NAME (001) - Level 10: High amount of POST requests in a small period of time (likely bot).

DESCRIPTION - - [01/Feb/2024:19:00:00 +0100] "POST /check.php?11111.111111 HTTP/2.0" 200 24 "DESCRIPTION" "Mozilla/5.0 (X11; Linux x8664) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
Groups: web, appsec, attack
Rule: 31533
```

---

<a href="https://www.paypal.com/donate/?hosted_button_id=GWWLEXEF3XL92">
  <img src="https://raw.githubusercontent.com/kraloveckey/kraloveckey/refs/heads/main/.assets/paypal-donate-button.png" alt="Donate with PayPal" width="225" height="100"/>
</a>
