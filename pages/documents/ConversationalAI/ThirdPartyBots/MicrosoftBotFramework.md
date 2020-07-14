---
pagename: Microsoft Bot Framework
redirect_from:
  - bot-connectors-microsoft-bot-framework.html
sitesection: Documents
categoryname: "Conversational AI"
documentname: Third-Party Bots
permalink: third-party-bots-microsoft-bot-framework.html
indicator:
---

### Overview

The following documentation outlines the configuration for the connector and how to implement functions specifically for **Microsoft Bot Framework**.

{: .important}
We support the Direct Line API version 3.0.

### Bot Configuration

{: .important}
See the [Getting Started guide](bot-connectors-getting-started.html) first to complete pre-requisite steps.

Throughout this document, you will be presented with the following screen to fill in the vendor data if you've selected **Microsoft Bot Framework**.

<img class="fancyimage" style="width:600px" src="img/msbotframework/vendor.png">

Figure 1.1 Showing the configuration that needs to be filled out

The following Microsoft information should be provided to LivePerson:

<table>
  <thead>
  <tr>
    <th>Item</th>
    <th>Description</th>
    <th>Example</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td>Direct Line Secret</td>
    <td>The first key provided in the direct line channel configuration </td>
    <td>x3XkXXoadX0.xknotlXk8VbU6vXXXX4kwdsGW-N3m-4XjTHPTde4XXX</td>
  </tr>
  <tr>
    <td>Direct Line Endpoint</td>
    <td>The endpoint the connector should use to reach the bot. Apart from the default you can also choose an endpoint that is close to the region that your account is configured for</td>
    <td>https://directline.botframework.com/v3/directline</td>
  </tr>
  <tr>
    <td>Multiple Activities (Optional)</td>
    <td>The connector normally only waits on the first response activity send by the bot and pass it to the customer. If you intent to send multiple activities with the same `ReplyToId` you need to activate this feature. The value describes the maximum time in milliseconds that is allowed to pass between two activities. After this time has elapsed, the connector will regard the request as handled</td>
    <td>1000</td>
  </tr>
  </tbody>
</table>

The Direct Line Secret can be found in the Azure Portal if you select the corresponding Web App Bot and edit the Configuration of the Direct Line Channel.

<img class="fancyimage" style="width:750px" src="img/msbotframework/secret.png">
Figure 1.2 The Direct Line Secret

For validation of the credentials provided, you can perform a connection test to see if the messages can be sent to the channel with the provided secret by clicking on the button "Test Connection".
For this test it is not necessary for the bot to respond with a message.

### Limitations

Currently advanced features of the direct line protocol like actions and attachments are not supported.
The Bot Connector utilizes the **channelData** property for anything besides plain text.

It is expected that a bot responds to every message sent by the consumer.
If no response is detected in a certain time frame, the Bot Connector assumes something is wrong and tries to transfer the conversation to an agent.

If you want the bot to reply with more than one message, you need to use the [multiMessage](#sending-multiple-responses) feature or configure the `Multiple Activities` option.

### Sending Rich Content (Structured Content)

Structured content/Rich Content is supported by the core LivePerson platform. Documentation for the feature can be found [here](getting-started-with-rich-messaging-introduction.html).

To send structured content from a bot implemented with the Microsoft Bot Framework, send the rich content in the `channelData` of the message activity.

This should contain a valid structured content body, along with an optional property containing metadata required for the structured content. Always validate your structured content using [this tool](https://livepersoninc.github.io/json-pollock/editor/) before using it in a bot.

{: .important}
If Images are sent in Rich content, then their URLs must be added to a whitelist via internal LivePerson configuration (Houston: `messaging.rich.content.valid.urls`). Please note that you must add all possible domains to this list manually as wildcards are not supported. Moreover, All domains must be HTTPS secure.

```json
{
  "type": "message",
  "text": "",
  "channelData": {
    "metadata": [
      {
        "type": "ExternalId",
        "id": "ABCD1234"
      }
    ],
    "structuredContent": {
      "type": "vertical",
      "elements": [
        {
          "type": "button",
          "click": {
            "actions": [
              {
                "text": "Recommend me a movie",
                "type": "publishText"
              }
            ]
          },
          "title": "Recommend a movie"
        }
      ]
    }
  }
}
```

Figure 4.1 Activity with Structured Content

### Sending Quick Replies (Structured Content)

{: .important}
**Please note** Quick Replies are only supported in Messaging Conversations.

Quick Replies is a special type of Structured Content. Is is a message sent along with predefined answers. The documentation can be found [here](quick-replies-introduction-to-quick-replies.html).
The message property in the Structured Content is mandatory.
For detailed information on Quick Replies check out the documentation for the specific channel ([Mobile SDK and Web](mobile-sdk-and-web-templates-quick-replies-template.html), [Facebook Messenger](facebook-messenger-templates-quick-replies-template.html), [Google RCS Business Messaging](google-rcs-business-messaging-templates-quick-replies-template.html))

```json
{
  "type": "message",
  "text": "",
  "channelData": {
    "metadata": [
      {
        "type": "ExternalId",
        "id": "ABCD1234"
      }
    ],
    "structuredContent": {
      "quickReplies": {
        "type": "quickReplies",
        "itemsPerRow": 8,
        "replies": [
          {
            "type": "button",
            "tooltip": "yes i do",
            "title": "yes",
            "click": {
              "actions": [
                {
                  "type": "publishText",
                  "text": "yep"
                }
              ],
              "metadata": [
                {
                  "type": "ExternalId",
                  "id": "Yes-1234"
                }
              ]
            }
          },
          {
            "type": "button",
            "tooltip": "No!",
            "title": "No!",
            "click": {
              "actions": [
                {
                  "type": "publishText",
                  "text": "No!"
                }
              ],
              "metadata": [
                {
                  "type": "ExternalId",
                  "id": "No-4321"
                }
              ]
            }
          }
        ]
      },
      "message": "Do you like Bots?"
    }
  }
}
```

Figure 5.1 Activity with Quick Replies

### Change Time To Response of Conversation

By providing a specific **action** in the **channelData**, the bot can change the TTR of a conversation.

LivePerson Messaging uses 4 different types of priorities:

- "URGENT"
- “NORMAL”
- “PRIORITIZED”
- “CUSTOM”

Only “CUSTOM” can set a value. The unit of the value is in seconds. The values of the other types are defined in the Agent Workspace.

A text message can also be provided simultaneously in the activity json.

```json
{
  "type": "message",
  "text": "",
  "channelData": {
    "action": {
      "name": "CHANGE_TTR",
      "parameters": {
        "ttrType": "CUSTOM",
        "value": "300"
      }
    }
  }
}
```

Figure 6.1 Activity with TTR Change

### Transfer / Escalations

Transfers and escalations are straightforward in both chat and messaging.

At the beginning of a chat session or when a messaging bot logs in, the whole list of enabled skills on the account is retrieved, keyed by name and stored.

When a transfer is requested by the bot, the skill name is matched to one already on the account and the id is retrieved and escalated to.

For **Microsoft Bot Framework**, the bot should provide the specific action in the **channelData** of the message activity.

An additional text message can also be provided.

```json
{
  "type": "message",
  "text": "",
  "channelData": {
    "action": {
      "name": "TRANSFER",
      "parameters": {
        "skill": "CUSTOM"
      }
    }
  }
}
```

Figure 7.1 Activity excerpt for a transfer Request

Also, you can use the official Microsoft BotBuilder SDK method called [EventFactory.createHandoffInitiation](https://docs.microsoft.com/en-us/javascript/api/botbuilder/eventfactory?view=botbuilder-ts-latest) and you can also add an additional text message.

```javascript
{
  text: "",
  ...EventFactory.createHandoffInitiation(context, handoffContext, transcript)
}
```

Figure 7.2 Alternative activity excerpt for a transfer Request

### Sending Pause/Delay Message

It is possible to send an event of type "delay" before regular content events and actions. This specifies the time the bot will wait before displaying the next message. There are two properties, `delay` and `typing`.

<ul>
  <li> <b>delay</b>: This is the number of seconds the bot will wait. These are expected to be only whole numbers for example for one second delay you will write 1 as a value</li>
  <li><b>typing</b>: This property will enable/disable the typing indicator while delay is happening. It is optional; if not provided then the value will be considered as true.</li>
</ul>

#### Sending delay between multiple messages

Setting a delay in between multiple messages (for more information on multiple message [check here](third-party-bots-microsoft-bot-framework.html#sending-multiple-responses)) is possible and an example of such a case (Message - Delay - Structured Content - Delay - Message) can be seen below:

```javascript
{
  "channelData": {
    "multiMessage": [
      {
        "type": "text",
        "value": "this is a text"
      },
      {
        "type": "delay",
        "value": 5
      },
      {
        "type": "structured-content",
        "value": {
          "metadata": [],
          "structuredContent": {
            "type": "vertical",
            "elements": [
              {
                "type": "button",
                "click": {
                  "actions": [
                    {
                      "text": "Recommend me a movie, please",
                      "type": "publishText"
                    }
                  ]
                },
                "title": "Recommend a movie"
              }
            ]
          }
        }
      },
      {
        "type": "delay",
        "value": {
          "delay": 3
          "typing": false
        }
      },
      {
        "type": "text",
        "value": "this is last message after a delay"
      },
    ]
  }
}
```

Please note the different ways of writing delay message in above example. In the first delay message Agent typing indicator will always be shown. In the second delay message user has ability to set the typing indicator as well.

#### Sending a single delay message (Legacy)

A single delay message can be send by adding `delay` and `typing` properties to `channelData`. An example of single text message that comes after delay can be seen below:

```json
{
  "type": "message",
  "text": "Hi i am sending a text coming after the delay!!",
  "channelData": {
    "delay": 8,
    "typing": false
  }
}
```

**Note:** using the delay as a single/sole response from the bot to the consumer, is effectively a ‘no response’ action. Using this allows the bot to receive a consumer message without responding to the consumer.

### Sending Private Text Message

It is possible to send a private text message from the Live Engage (LE-UI) via agent workspace. This feature can now be used via the Third-Party bots as well. This will allow Brands to define private message text within the conversational flow of the bot. These messages are published into the conversation for other Agent/Manger participants. This enables Brands to customize messages giving more insight, summarizing actions taken by the bot, or also advising on next actions the handover agent should take.

{: .important}
Please note If you have not migrated to new Agent Workspace you will not be able to see the `Private` message indicator in the conversation window. Nevertheless, private text messages will not be shown to the consumer and only remain visible to Agents and Managers.

Please note private text message will never be shown to the consumer and will be visible only inside the conversation window of agent workspace. There are two properties, `text` and `messageAudience` which need to be added in with the response body of the function.

| key             | value                                 | notes                     |
| --------------- | ------------------------------------- | ------------------------- |
| text            | any string value                      | mandatory                 |
| messageAudience | value should be "AGENTS_AND_MANAGERS" | case sensitive, mandatory |

<br />

#### Sending Private Text Message between multiple messages

Setting a private text message in between multiple messages (for more information on multiple messages [check here](third-party-bots-microsoft-bot-framework.html#sending-multiple-responses)) is possible and an example of such a case (Simple Text Message - Private Text Message) can be seen below:

```javascript
{
  "channelData": {
    "multiMessage": [
      {
        "type": "text",
        "value": "Hi How are you doing?"
      },
      {
        "type": "private-message",
         "value": {
              "text": "This is a private text",
              "messageAudience": "AGENTS_AND_MANAGERS",
          }
      }
    ]
  }
}
```

#### Sending Single Private Text Message with Action

A single private text message with action can be send by adding `text` and `messageAudience` properties with relevant action (e.g. [Transfer/Escalate](third-party-bots-microsoft-bot-framework.html#transfer--escalations)) properties. An example of such case is below:

```json
{
  "type": "message",
  "text": "This is a private text",
  "channelData": {
    "messageAudience": "AGENTS_AND_MANAGERS",
    "action": {
      "name": "TRANSFER",
      "type": "client",
      "parameters": {
        "skill": "human_skill"
      },
      "result_variable": "none"
    }
  }
}
```

### Close Chat/Conversation

To close a chat or messaging conversation, we provide the action object as we did for a transfer. The activity should contain the following action.

An additional text message can also be provided.

```json
{
  "type": "message",
  "text": "",
  "channelData": {
    "action": {
      "name": "CLOSE_CONVERSATION"
    }
  }
}
```

Figure 9.1 Activity excerpt for a close conversation request

Also, you can use the official Microsoft BotBuilder Activity Type [End of Conversation](https://docs.microsoft.com/en-us/azure/bot-service/dotnet/bot-builder-dotnet-activities?view=azure-bot-service-3.0#endofconversation) with an additional text message:

```json
{
  "type": "endOfConversation",
  "text": ""
}
```

Figure 9.2 Alternative activity excerpt for a close conversation request

### Engagement attributes as context

Third-Party bots allows the collection of engagement attributes (more information can be found [here](engagement-attributes-types-of-engagement-attributes.html)) if `Engagement Attributes` option is checked in the `Conversation Type` step as shown in Figure 10.1.

<img class="fancyimage" style="width:750px" src="img/engagement_attr_select.png">
Figure 10.1 Conversation Type step in creation/modification of bot configuration.

These attributes are **only** collected at the start of a conversation. Third-Party bots leverage the LivePerson Visit Information API to collect the engagement attributes, Further information Visit Information API can be found [here](visit-information-api-visit-information.html). Moreover, Engagement attributes are not updated throughout the life cycle of a conversation and only passed along with each message request. In Microsoft Bot these engagement attributes are added to the property `lpSdes` which is part of another custom property of `context`. This context information within a conversation is preserved/passed in `channelData` property (further information about `channelData` can be found [here](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0#activity-object)). An example of the request body can be seen below:

```javascript
{
  "text": "Some Conversation Text",
  "channelData": {
    "context": {
      "lpEvent": {}, // Holds LP Events
      "lpSdes": {}
    }
  }
}
```

### Welcome Event

The behavior of the welcome event is different depending on whether the bot is for chat or messaging. This divergence comes down to the way that each individual LivePerson product works..

A Messaging conversation qualifies as "initiated" from a Conversational Cloud perspective only after the consumer sends their first message. The consumer is prompted for their initial message in the channel they have chosen to initiate the conversation. As a result, the consumer’s first message is something that can be parsed by the bot and an intent determined.

A Chat conversation is considered started when the chat is routed to an agent. Best practice is for the agent to provide the first response. In this scenario, there is no text from the consumer to parse, thus the default ‘WELCOME’ event is utilized as a start point for the bot to prompt the user to provide input and progress the conversation.

Ensure you have an ‘entry point’ in your bot that responds to the default ‘WELCOME’ action send by a new chat customer.

```json
{
  "type": "message",
  "text": "",
  "channelData": {
    "action": {
      "name": "WELCOME"
    }
  }
}
```

Figure 11.1 Customer activity excerpt on a new chat

### Sending Multiple Responses

As stated under Limitations we only process the first activity that is send in response to a customer message.
If your bot should reply with more than one message, you need to send a multiMessage property inside the channelData.

You can define any number of messages in this array. This messages can be plain text, define a delay before the
next message is send or contain structured content in the same format that could also be be send directly as a channel data object.

```javascript
{
  "type": "message",
  "text": "",
  "channelData": {
    "multiMessage": [
      {
        "type": "text",
        "value": "this is a text"
      },
      {
        "type": "delay",
        "value": 1 // value is considered as seconds
      },
      {
        "type": "structured-content",
        "value": {
          "metadata": [],
          "structuredContent": {}
        }
      }
    ]
  }
}
```

### Sending Encoded Metadata

Conversational Cloud Messaging platform provides a new metadata input type (“encodedMetadata”) for passing a base64 encoded metadata on a conversation. The new metadata input type is in addition to the existing [conversation metadata](messaging-agent-sdk-conversation-metadata-guide.html) input field. Third-party Bot also supports this property and this section will cover the information needed for you to send encoded metadata within your conversations. Before sending encoded metadata you must ensure the following conditions in order to successfully send the data.

<ul>
  <li><b>Common.EncodedMetadata</b> AC feature is ON</li>
  <li>Content is base64 encoded</li>
  <li> Metadata size is limited to 5k</li>
</ul>

{: .important}
Failing to comply with the above validation points will cause the message to be dropped. This feature is only available for the messaging conversations not for chat conversations

Encoded Metadata can be sent with simple Text, Rich Content (structured content) and Multiple responses.

#### Sending Text Message with Encoded Metadata

For sending `encodedMetadata` with a text message you need to provide this property in `channelData` object. Be careful with the camel-case characters you must provide it exactly the same. An example of the simple text message response is below:

```json
{
  "type": "message",
  "text": "Hi i am sending a text with encoded metadata!!",
  "channelData": {
    "encodedMetadata": "ewoic29tZUluZm8iOiAiSSB3YXMgZW5jb2RlZCIKfQ=="
  }
}
```

#### Sending Rich Content (structured content) with Encoded Metadata

For sending [structured content](getting-started-with-rich-messaging-introduction.html). You need to add additional property of `encodedMetadata` with your rich content object that you have defined within `channelData` property. An example of the simple Rich Content `JSON` can be seen below:

```json
{
  "type": "message",
  "text": "Hi i am sending a structured content with encoded metadata!!",
  "channelData": {
    "metadata": [
      {
        "id": "1234",
        "type": "ExternalId"
      }
    ],
    "encodedMetadata": "ewoic29tZUluZm8iOiAiSSB3YXMgZW5jb2RlZCIKfQ==",
    "structuredContent": {
      "type": "vertical",
      "elements": [
        {
          "type": "button",
          "click": {
            "actions": [
              {
                "text": "Recommend a movie",
                "type": "publishText"
              }
            ]
          },
          "title": "Recommend a movie"
        }
      ]
    }
  }
}
```

#### Sending Multiple Responses with Encoded Metadata

For sending Encoded Metadata with multiple responses one must provide an additional property of `encodedMetadata` with the already existing `type` and `value` properties under `multiMessage` array object. Sending encoded metadata is supported for the `text` and `structure-content` types only. An example of sending encoded metadata with both types can be found below.

{: .important}
**Please note** if you will send `encodedMetadata` within the `value` property of type `structure-content`, then it will not be passed. This kind of format is only acceptable if you are sending a single Rich Content as a response. Furthermore, for each message, you can see different `encodedMetadata` are defined so both of the messages will be sent with different encoded metadata.

```javascript
{
  "type": "message",
  "text": "",
  "channelData": {
    "multiMessage": [
      {
        "type": "text",
        "value": "this is a text",
        "encodedMetadata": "ewoic29tZUluZm8iOiAiSSB3YXMgZW5jb2RlZCIKfQ=="
      },
      {
        "type": "structured-content",
        "encodedMetadata": "ZGlmZmVyZW50IGVuY29kZWQgbWV0YWRhdGE=",
        "value": {
          "metadata": [
            // ... Some structured content metadata
          ],
          "structuredContent": {
            // ... Some structured content object
          }
        }
      }
    ]
  }
}
```
