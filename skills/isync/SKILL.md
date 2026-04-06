---
name: isync
description: "Manage email via mbsync/isync — sync IMAP mailboxes to local Maildir, read messages, search mail, check for new/unread emails. Use when user asks to check email, read a message, search inbox, find emails from someone, monitor for new mail, or anything involving email reading/syncing. NOT for sending email (use himalaya or another SMTP tool for that)."
---

# isync (mbsync) — Local Email Sync & Reader

Sync IMAP mailboxes to local Maildir format using `mbsync`, then read/search/filter messages directly from the filesystem. Fast, offline-capable, no IMAP timeout issues.

## Architecture

```
IMAP Server ──mbsync──▶ ~/Mail/{account}/INBOX/{new,cur,tmp}/
                              ▲
                         Read files directly
```

- **mbsync** handles IMAP sync (pulls new messages, flags, deletions)
- Messages are plain text files in Maildir format
- `new/` = unread messages, `cur/` = read messages, `tmp/` = in-progress

## Prerequisites

- `isync` installed (`brew install isync`)
- `~/.mbsyncrc` configured with accounts
- Maildir directories created (e.g., `~/Mail/{account}/INBOX`)

## Core Commands

### Sync all accounts
```bash
mbsync -a
```

### Sync a specific channel/account
```bash
mbsync work
mbsync personal
```

### Check for new (unread) mail
```bash
ls ~/Mail/{account}/INBOX/new/
```

### Read a message
Messages are plain RFC 822 text files. Read with `cat` or parse headers:
```bash
# Get headers
grep -i "^From:\|^Subject:\|^Date:\|^To:" ~/Mail/{account}/INBOX/cur/{filename}

# Get full message
cat ~/Mail/{account}/INBOX/cur/{filename}
```

### Search across messages
```bash
# Find emails from a specific sender
grep -rl "^From:.*sender@example.com" ~/Mail/*/INBOX/cur/

# Find by subject
grep -rl "^Subject:.*keyword" ~/Mail/*/INBOX/cur/

# Full-text search
grep -rl "search term" ~/Mail/*/INBOX/cur/
```

### Count messages
```bash
# Unread count
ls ~/Mail/{account}/INBOX/new/ 2>/dev/null | wc -l

# Total in inbox
ls ~/Mail/{account}/INBOX/cur/ ~/Mail/{account}/INBOX/new/ 2>/dev/null | wc -l
```

### Find most recent messages
Maildir filenames include UIDs. Sort by UID (higher = newer):
```bash
ls ~/Mail/{account}/INBOX/cur/ | sed 's/.*U=\([0-9]*\).*/\1 &/' | sort -n | tail -5 | cut -d' ' -f2
```

## Maildir Filename Format

Files follow the pattern: `{timestamp}.{pid}_{seq}.{hostname},U={uid}:2,{flags}`

Flags in the filename (after `:2,`):
- `S` = Seen (read)
- `R` = Replied
- `F` = Flagged
- `T` = Trashed
- `D` = Draft

## Configuration Reference (~/.mbsyncrc)

```ini
IMAPAccount {name}
Host imap.example.com
Port 993
User user@example.com
PassCmd "security find-generic-password -s '{keychain-item}' -w"
TLSType IMAPS
CertificateFile /etc/ssl/cert.pem

IMAPStore {name}-remote
Account {name}

MaildirStore {name}-local
Subfolders Verbatim
Path ~/Mail/{name}/
Inbox ~/Mail/{name}/INBOX

Channel {name}
Far :{name}-remote:
Near :{name}-local:
Patterns INBOX
Create Near
Expunge None
SyncState *
```

### Password handling
Use `PassCmd` to avoid plaintext passwords. Common patterns:
```ini
# macOS Keychain
PassCmd "security find-generic-password -s 'keychain-item' -w"

# For passwords with special characters, use a wrapper
PassCmd "python3 -c \"import keyring; print(keyring.get_password('service', 'user'))\""
```

## Workflow: Check for New Mail

1. Sync: `mbsync -a`
2. Check `new/` dirs for each account
3. If files exist in `new/`, read headers (From, Subject, Date)
4. Classify/report as needed

## Workflow: Search for Specific Email

1. Sync first if freshness matters: `mbsync -a`
2. Use `grep -rl` to find matching files
3. Read full message with `cat`

## Limitations

- **Read-only by design** — mbsync syncs, doesn't send. Use himalaya, msmtp, or another SMTP client for sending.
- **INBOX only by default** — add more folders to `Patterns` in `.mbsyncrc` if needed (e.g., `Patterns INBOX Sent Drafts`)
- **First sync downloads everything** — can be slow for large mailboxes. Use `MaxMessages` in config to limit.
