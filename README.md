# Switchboard

Detox without losing your social network.

Switchboard is a small bridge that forwards messages from LINE, WhatsApp, and Discord to your flip phone as plain SMS, and lets you reply from the flip phone using a short routing code. No smartphone apps required on your end.

Right now it runs on macOS using Apple Messages as the SMS bridge. Linux and Windows support is planned through an SMS API such as Twilio or Vonage (see the [roadmap](#roadmap)).

> **Status:** early prototype. `v1sketch.py` is a conceptual routing engine, not a finished, installable tool.

## How it works

```
 LINE / WhatsApp / Discord
            │
            ▼
   okline · whatsmeow · discli      (platform clients)
            │
            ▼
      Switchboard (Python)
            │   SMS transport (Apple Messages today)
            ▼
        Your flip phone  ──── reply ────┐
                                        ▼
                          Inbound SMS monitor (chat.db today)
                                        │
                                        ▼
                          Switchboard routes reply back
                          to the correct platform + chat
```

1. A platform client (okline, whatsmeow, or discli) receives an internet message.
2. Switchboard wraps it in an **envelope** and texts it to your number.
3. You reply from the flip phone with a **routing code** followed by your message.
4. Switchboard detects your reply and sends it to the right platform and chat.

## Message format

### Incoming (Switchboard → flip phone)

```
{<platform><chat_id>} <sender> (<chat name>): <message>
```

Example:

```
{l1} Alex (Weekend Hikers): hi
```

### Replying (flip phone → Switchboard)

```
<platform><chat_id> <your reply>
```

Example:

```
l1 yeah
```

### Platform prefixes

| Prefix | Platform | Backend   |
|--------|----------|-----------|
| `l`    | LINE     | okline    |
| `d`    | Discord  | discli    |
| `w`    | WhatsApp | whatsmeow |

The number after the prefix is the chat ID, so `l1` is LINE chat 1, `d2` is Discord chat 2, and so on.

### Loop prevention

Anything that starts with `{` is treated as one of Switchboard's own outgoing envelopes and dropped, so the bridge never relays its own messages back to the internet. Don't start your replies with `{`.

## Components

| Function | Role |
|----------|------|
| `route_outbound_to_flip_phone(platform_prefix, chat_id, sender_name, message_text)` | Builds the envelope and sends it to your phone |
| `process_inbound_from_flip_phone(raw_sms_text)` | Parses a reply from the flip phone and dispatches it to the right platform |

## Requirements

Current (macOS):

- A Mac signed into iMessage, with permission to read `~/Library/Messages/chat.db` (Full Disk Access for your terminal or Python)
- Python 3
- [okline](#) for LINE
- [whatsmeow](https://github.com/tulir/whatsmeow) for WhatsApp, running as a background process
- [discli](#) for Discord
- A flip phone that can send and receive SMS

> Replace the `#` links above with the repos you're actually using.

## Roadmap

- [ ] Implement `send_sms_via_imessage_bridge` (AppleScript or shell)
- [ ] Implement the `chat.db` watcher that feeds `process_inbound_from_flip_phone`
- [ ] Wire up the okline, discli, and whatsmeow clients
- [ ] Persistent chat-ID mapping (so `l1` always means the same chat)
- [ ] Split long messages to fit SMS length limits
- [ ] Config file for credentials and allowed chats
- [ ] **Linux and Windows support** via an SMS API (most likely Twilio or Vonage) in place of the Apple Messages bridge
- [ ] Pluggable SMS transport layer, so macOS (Apple Messages) and API-based backends share the same routing code

## Known limitations

- Replies must follow the `<prefix><id> <text>` format exactly; malformed texts are ignored.
- Plain SMS only: no images, stickers, or reactions.
- macOS only for now, and the machine needs to stay on and online. Linux and Windows are planned.
- Once API-based SMS lands, carrier and provider fees (e.g. per-message pricing) will apply.

## Contributing

Ideas and PRs are welcome. Since this is an early sketch, open an issue first to talk through bigger changes.

## License

Add a license of your choice (MIT is a common default).
