# mc-debug
[Deutsch](README.de.md) | **English**

Read-only troubleshooting helper for [mailcow-dockerized](https://github.com/mailcow/mailcow-dockerized).
It wraps the container, database and `doveadm` calls you need when "mail is missing" or an IMAP sync looks stuck,
so you don't have to remember the long one-liners. It changes nothing on your system.

## Requirements

- Run on the mailcow host as root or a user with Docker access
- Docker Compose v2
- imapsync report parsing tested with imapsync 2.178 and 2.314 (bundled with mailcow); mailcow versions tested: 2026-10 and 2026-11

## Install

    install -m 0755 mc-debug /usr/local/bin/mc-debug

The mailcow directory defaults to `/opt/mailcow-dockerized`. Override with `MAILCOW_DIR=/path mc-debug ...`.

## Usage

    mc-debug sync                       status of all IMAP sync jobs (look at min_ago and running)
    mc-debug syncreport <job-id>        short report of the last run of one job
    mc-debug runner                     running imapsync processes and the parallel limit
    mc-debug box [user@domain]          message count per user, or per folder for one user
    mc-debug find <user@domain> <text>  find mails by subject text in all folders
    mc-debug last <user@domain> [period] [count]  latest received mails of a user, newest first (default 2d, 20)
    mc-debug trace <text> [period]      search postfix/rspamd/dovecot logs (30m, 3h, 2d; default 3h)
    mc-debug errors [period] [all]      postfix/dovecot problems grouped by message, scanner noise hidden (default 24h)
    mc-debug stats [period]             short mail statistics: counts, rspamd actions, top domains (default 24h)
    mc-debug spam [period]              rspamd actions, postfix rejects, top flagged senders, quarantine count (default 24h)
    mc-debug findspam <text> [period]   flagged/rejected mails matching sender, recipient or IP, plus junk folder search
    mc-debug mailtrace <message-id> [period]  where did a mail go? rspamd decisions, postfix delivery, dovecot filing (default 72h)
    mc-debug show <user> <message-id|subject> [full|links]  read one mail: key headers, text with full links, attachment names (read-only)

## Example: sync jobs stop running

Symptom: mails fetched via IMAP sync arrive late or not at all, but only sometimes.

1. `mc-debug sync` shows one job with `running = 1` and a large `min_ago`, while all other jobs are overdue too.
2. `mc-debug runner` shows a single long-running imapsync process.
3. `mc-debug syncreport <id>` shows what that job is doing, e.g. thousands of messages being removed one by one
   on the remote side.

In our case the runner processed jobs strictly one after another (`max_parallel = 1`), so one slow job delayed every other job. On the mailcow version we saw this on (2026-11) the runner can run several jobs in parallel; the limit is `max_parallel` in the web UI.
The cause was a Gmail account that kept messages in "All Mail" after deletion, so the same mails were
compared, copied back and deleted again on every run.

## Example: I can't find a mail

1. `mc-debug last user@example.com 3d` lists the latest received mails of a user with folder, sender, subject and Message-ID. Junk and Trash are included, a mail often sits there.
2. `mc-debug show user@example.com <message-id>` reads one of them: key headers (spam verdict, DKIM/SPF results), the text and attachment names. If a link is missing in the text, `mc-debug show user@example.com <message-id> links` lists the links of the HTML part.
3. If the mail is not in the list: `mc-debug findspam <sender>` shows whether it was flagged or rejected, `mc-debug trace <sender> 24h` searches the raw logs, and with a Message-ID `mc-debug mailtrace <message-id>` follows the mail through Rspamd, Postfix and Dovecot.

## Notes

- `show` prints the content of a user's mail. Use it only where you are authorised to access that mailbox.
- Table and column names (`imapsync`, `returned_text`, ...) come from mailcow's database and may change between versions.
- `find` matches the decoded subject via `doveadm`; `trace` only finds what the services actually log
  (queue IDs, addresses, message IDs), usually not subjects.

## AI assistance

This tool was created with the help of AI assistants and is used by the author on a live mailcow installation.

## License

MIT
