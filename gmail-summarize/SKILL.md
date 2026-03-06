---
name: gmail-summarize
description: Fetch recent unread Gmail (yesterday + today) and send a digest to the user.
metadata: {
  "nanobot": {
    "emoji": "📬",
    "requires": {
      "config": "~/.nanobot/config.json",
      "config_fields": ["channels.email.imapHost", "channels.email.imapPort", "channels.email.imapUsername", "channels.email.imapPassword"],
      "external_connections": ["IMAP server specified in channels.email.imapHost"]
    }
  }
}
---

# Gmail Recent Digest

## Requirements
- **Python 3** must be available in the container environment (`python` command).
- **`~/.nanobot/config.json`** must exist and contain a valid `channels.email` block with the following fields:
  - `imapHost`, `imapPort`, `imapUsername`, `imapPassword`
  - If any field is missing or incorrect, the fetch script will fail with an authentication error.
- The email channel does **not** need to be `"enabled": true` — credentials are read directly by the script regardless of channel state.

## Security notes
- The fetch script reads the **entire** `~/.nanobot/config.json` file (which may contain other sensitive data such as API keys), but uses **only** the `channels.email` fields. No other fields are accessed or transmitted.
- The only external connection made is to the IMAP server declared in `channels.email.imapHost`. No other endpoints are contacted.
- Credentials are used solely to authenticate the IMAP session and are not logged or stored elsewhere by this skill.

## When to use
- User asks "check my Gmail", "summarize my emails", "邮件摘要"
- Cron trigger message contains "gmail_digest"

## Workflow
1. Run the fetch script via exec tool:
   `python {workspace}/skills/gmail-summarize/scripts/fetch_unseen.py`
   (replace {workspace} with the actual workspace path, typically the nanobot source root)
2. Parse the JSON array. Each item has: sender, subject, date, body
3. For each email compose one line:
   `[date] sender | subject — one-sentence body summary`
4. Send the full digest via MessageTool in this format:

   📬 **邮件摘要** (N封，覆盖 MM/DD–MM/DD)

   • [日期] 发件人 | 主题 — 一句话摘要
   • [日期] 发件人 | 主题 — 一句话摘要
   ...

5. If result is empty, send: 📭 近两日暂无未读邮件

## Output Rules
- Send the digest message ONLY. Do NOT add any extra comments, greetings, explanations, or follow-up questions before or after the digest.
- Do NOT say things like "主人，以下是您的邮件摘要" or "如需了解详情请告知" etc.
- The digest message itself is the complete and final response.
- The one-sentence body summary MUST be translated into Chinese. Sender names and subjects should keep their original text as-is.