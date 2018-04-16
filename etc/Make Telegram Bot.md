## Make Telegram Bot

#### 텔레그램 API 발급

@Botfather를 통해 만든다.

1. @Botfather를 초대한 채팅방에 입장하여 /help
2. /newbot
3. bot name 입력
4. user name 입력 (unique)

위와 같은 방식으로 Bot API를 발급받을 수 있으며, 

Token을 얻을 수 있다.



#### 텔레그램 Chat id 조회

1. 채널을 생성
2. 발급받은 bot을 관리자로 초대
3. 해당 채널에 아무 대화나 입력
4. https://api.telegram.org/bot${bot_token}/getUpdates 을 호출

4번의 url을 호출하면 chat id를 얻을 수 있다.



#### 메세지 전송

```shell
curl --silent POST "${BOT_API}" -d text="${MESSAGE}" -d chat_id=${CHAT_ID}
```

* https://api.telegram.org/bot${bot_token}/sendMessage을 호출

* parameter 정보

  * 필수 파라미터는 chat_id와 text

  | **Parameters**           | **Type**                                                     | **Required** | **Description**                                              |
  | ------------------------ | ------------------------------------------------------------ | ------------ | ------------------------------------------------------------ |
  | chat_id                  | Integer or String                                            | Yes          | Unique identifier for the target chat or username of the target channel (in the format `@channelusername`) |
  | text                     | String                                                       | Yes          | Text of the message to be sent                               |
  | parse_mode               | String                                                       | Optional     | Send [*Markdown*](https://core.telegram.org/bots/api#markdown-style) or [*HTML*](https://core.telegram.org/bots/api#html-style), if you want Telegram apps to show [bold, italic, fixed-width text or inline URLs](https://core.telegram.org/bots/api#formatting-options) in your bot's message. |
  | disable_web_page_preview | Boolean                                                      | Optional     | Disables link previews for links in this message             |
  | disable_notification     | Boolean                                                      | Optional     | Sends the message [silently](https://telegram.org/blog/channels-2-0#silent-messages). Users will receive a notification with no sound. |
  | reply_to_message_id      | Integer                                                      | Optional     | If the message is a reply, ID of the original message        |
  | reply_markup             | [InlineKeyboardMarkup](https://core.telegram.org/bots/api#inlinekeyboardmarkup) or [ReplyKeyboardMarkup](https://core.telegram.org/bots/api#replykeyboardmarkup) or [ReplyKeyboardRemove](https://core.telegram.org/bots/api#replykeyboardremove) or [ForceReply](https://core.telegram.org/bots/api#forcereply) | Optional     | Additional interface options. A JSON-serialized object for an [inline keyboard](https://core.telegram.org/bots#inline-keyboards-and-on-the-fly-updating), [custom reply keyboard](https://core.telegram.org/bots#keyboards), instructions to remove reply keyboard or to force a reply from the user. |



