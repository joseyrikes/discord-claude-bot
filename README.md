# Discord Claude Bot

Run [Claude Code](https://docs.anthropic.com/en/docs/claude-code) on your local projects from Discord. Each Discord channel maps to a folder on your machine, so you can work with different repositories just by switching channels.

```
#my-project   →  /path/to/repos/my-project
#another-repo →  /path/to/repos/another-repo
```

## Features

- **Channel-to-project mapping**: the channel name selects the working folder.
- **Persistent sessions**: each channel keeps its own session and resumes automatically.
- **Live streaming**: watch tool calls and responses as they happen.
- **Activity log**: shows up to 20 lines of recent activity, including tool calls and their parameters.
- **Single-user access**: only responds to the Discord user you configure.
- **`/clear` command**: reset a channel's session at any time.

## Prerequisites

- [Bun](https://bun.sh/)
- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code), installed and authenticated
- A Discord account with permission to add bots to a server

## Quickstart

```bash
git clone <repository-url>
cd claude-code-discord
bun install
```

Create a `.env` file in the project root:

```env
DISCORD_TOKEN=your_discord_bot_token_here
ALLOWED_USER_ID=your_discord_user_id_here
BASE_FOLDER=/path/to/your/repos
```

Start the bot:

```bash
bun start
```

You should see:

```
Bot is ready! Logged in as Claude Code Bot#1234
Successfully registered application commands.
```

New to Discord bots? Follow the full setup below.

## Configuration

| Variable          | Required | Description                                                                 |
| ----------------- | -------- | --------------------------------------------------------------------------- |
| `DISCORD_TOKEN`   | Yes      | Bot token from the Discord Developer Portal. Keep it secret.                |
| `ALLOWED_USER_ID` | Yes      | Your Discord user ID. The bot ignores everyone else.                        |
| `BASE_FOLDER`     | Yes      | Parent folder of your repos. Each channel maps to a subfolder of this path. |

For example, with `BASE_FOLDER=/Users/you/repos`, messages in `#my-project` run Claude Code in `/Users/you/repos/my-project`.

## Full Setup Guide

### 1. Create a Discord application and bot

1. Open the [Discord Developer Portal](https://discord.com/developers/applications) and click **New Application**.
2. Name it (e.g. "Claude Code Bot") and click **Create**.
3. Go to the **Bot** section in the sidebar. (Newer applications already include a bot user; if you see an **Add Bot** button, click it.)
4. Click **Reset Token** / **Copy** to get your bot token and store it somewhere safe.
5. Under **Privileged Gateway Intents**, enable **Message Content Intent**, then save.

### 2. Invite the bot to your server

1. Go to **OAuth2 → URL Generator**.
2. Select the scopes `bot` and `applications.commands`.
3. Select the bot permissions: Send Messages, Use Slash Commands, Read Message History, Embed Links.
4. Open the generated URL, choose your server, and authorize.

### 3. Get your Discord user ID

1. In Discord, go to **Settings → Advanced** and enable **Developer Mode**.
2. Right-click your username in any channel and choose **Copy User ID**.

### 4. Prepare your repositories

Put your projects under `BASE_FOLDER`, with folder names matching your channel names:

```
/path/to/your/repos/
├── my-project/        # → #my-project
├── another-repo/      # → #another-repo
├── test-app/          # → #test-app
└── experimental/      # → #experimental
```

> **Important**: channel names must match folder names exactly. Discord converts spaces to hyphens and lowercases channel names, so name your folders accordingly.

### 5. Create the channels

In your Discord server, create one channel per repository: `#my-project`, `#another-repo`, `#test-app`, `#experimental`.

### 6. Run the bot

```bash
bun start
# or
bun run src/index.ts
```

> **Warning**: do not use hot reload (`bun --hot`). It can break process management and spawn multiple Claude processes.

## Usage

Type any message in a channel that matches a repository folder. The bot runs Claude Code with your message as the prompt and streams the results back.

| Input         | What it does                                                    |
| ------------- | --------------------------------------------------------------- |
| Any message   | Runs Claude Code with the message as the prompt                 |
| `/clear`      | Resets the current channel's session; the next message starts fresh |

### Example

```
You: hello
Bot: 🔧 LS (path: .)
     🔧 Read (file_path: ./package.json)
     Hello! I can see this is a Node.js project. What would you like to work on?
     ✅ Completed (3 turns)
```

## How It Works

1. You send a message in a channel such as `#my-project`.
2. The bot checks that the sender matches `ALLOWED_USER_ID`.
3. It resolves the channel name to `BASE_FOLDER/my-project` and starts (or resumes) that channel's Claude Code session there.
4. Tool calls and responses are streamed back into the channel as they happen.

## Security

This bot gives Claude Code access to files and commands on the machine it runs on, driven from a chat message. Treat it accordingly:

- Set `ALLOWED_USER_ID` to your own ID only. Never leave it blank or share it with others.
- Keep `.env` out of version control and never share your bot token. If it leaks, reset it in the Developer Portal.
- Point `BASE_FOLDER` at a directory containing only the projects you want Claude to touch.
- Run the bot in a private server where you control who can post.

## Troubleshooting

| Problem                          | Things to check                                                                                          |
| -------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Bot is online but doesn't reply  | Is **Message Content Intent** enabled? Does `ALLOWED_USER_ID` match your account? Does the bot have access to the channel? |
| Slash commands don't appear      | Was the bot invited with the `applications.commands` scope? Commands can take a moment to register.       |
| "Folder not found" or wrong repo | Does the channel name exactly match a folder in `BASE_FOLDER`? Remember Discord uses lowercase and hyphens. |
| Multiple Claude processes        | Make sure you aren't running with `bun --hot` or running two bot instances.                               |
| Claude Code fails to start       | Confirm the `claude` CLI is installed, on your `PATH`, and logged in.                                     |

For more detail on development and debugging, see [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Released under the [MIT License](LICENSE).
