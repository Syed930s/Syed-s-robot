# Syed-s-robot

# Syed's Robot

A small terminal tool for sending Discord webhook messages (with file attachments) from `.syd` files. Define your webhooks once in a C-style config file, then pick a bot and send. Every message posts as **Syed's Robot**, whatever the webhook is named in Discord.

## Requirements

- Python 3.8+
- `requests` (`pip install requests`)

## Run

```
python3 syd_robot.py              # then: use file.syd
python3 syd_robot.py file.syd     # load a file on startup
```

If the script's filename contains an apostrophe, wrap it in double quotes: `python3 "Syed's-robot.py"`.

## Commands

At the `Syed's Robot>` prompt:

| Command | What it does |
|---|---|
| `use <file.syd>` | Load bots from a `.syd` file (replaces the loaded bots) |
| `bots` | List loaded bots |
| `<name>` or `<number>` | Select a bot |
| `list` | Show scheduled messages |
| `cancel <id>` | Cancel a scheduled message |
| `help` | Show help |
| `exit` | Quit |

Inside a bot:

| Command | What it does |
|---|---|
| `:at <when>` | Schedule the next message |
| `:list`, `:cancel <id>` | Manage scheduled messages |
| `:back` | Leave the bot |

Times for `:at`: `30s`, `10m`, `2h`, `1h30m`, `18:30` (next occurrence), or `2026-10-05 09:00`.

Scheduled messages live in memory only, so they are lost when you exit.

## The .syd format

```c
#include define;
#include webhook;

// single-line comment
/* multi-line
   comment */

#define {
    #for general;                       // label name
        #stdin;                         // you type the message
        #stdin attach_file with name "files";   // then: files /path/a.png /path/b.txt
        #webhook {
            "https://discord.com/api/webhooks/ID/TOKEN";
        }
}

#define {
    #for daily_report;
        #stdout "Daily report attached";   // quoted = hard-coded text
        #stdout /home/me/report.pdf;       // unquoted = file path to attach
        #webhook {
            "https://discord.com/api/webhooks/ID/TOKEN";
        }
}

return 0;   // optional: stop reading the file here
```

### Directives

- `#include define;` / `#include webhook;`: required libraries. They are built in.
- `#define { ... }`: one bot per block. A file can hold as many as you like, across different channels or servers.
- `#for <name>;`: the bot's name.
- `#stdin;`: ask for the message at runtime.
- `#stdin attach_file with name "x";`: ask for attachments. Type paths at the `x>` prompt, with or without `x` in front. Several files are allowed, and a single path may contain spaces. Press Enter to skip.
- `#stdout "text";`: hard-coded message text.
- `#stdout /path/file;`: hard-coded attachment (unquoted means a path).
- `#webhook { "url"; }`: one or more webhook URLs. With several, the message goes to all of them.
- `return 0;` inside a `#define`: after one message, go back to the main prompt instead of staying in the bot.
- `return 0;` at the top level: stop reading the rest of the file.

A bot with only `#stdout` lines sends once as soon as you select it. A bot with `#stdin` keeps asking for messages until you type `:back`.

## Security

A webhook URL is a secret. Anyone who has it can post to that channel under any name, ping @everyone, and delete the webhook. A banned user with the URL can still post. A webhook cannot read messages, post elsewhere or change server settings.

- Don't commit real `.syd` files to git. Add `*.syd` to `.gitignore` and share only a placeholder `example.syd`.
- If a URL leaks, delete the webhook in Discord (Channel Settings → Integrations → Webhooks) and create a new one.

## License

BSD 3-Clause ("New" or "Revised"). See [LICENSE](LICENSE).

## Notes

- Text over 2000 characters is split into several messages.
- Attachments are sent 10 per message.
- Rate limits (HTTP 429) are retried once.
