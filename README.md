# mc-debug
[Deutsch](README.de.md) | **English**

Read-only troubleshooting helper for [mailcow-dockerized](https://github.com/mailcow/mailcow-dockerized).
It wraps the container, database and `doveadm` calls you need when "mail is missing" or an IMAP sync looks stuck,
so you don't have to remember the long one-liners. It changes nothing on your system.

## Requirements

- Run on the mailcow host as root or a user with Docker access
- Docker Compose v2
- imapsync report parsing tested with imapsync 2.178 (bundled with mailcow); mailcow version tested: Version 2026-10

## Install

    install -m 0755 mc-debug /usr/local/bin/mc-debug

The mailcow directory defaults to `/opt/mailcow-dockerized`. Override with `MAILCOW_DIR=/path mc-debug ...`.

## Usage

    mc-debug sync                       status of all IMAP sync jobs (look at min_ago and running)
    mc-debug syncreport <job-id>        short report of the last run of one job
    mc-debug runner                     is imapsync running right now? is the lock present?
    mc-debug box <user@domain>          message count per folder
    mc-debug find <user@domain> <text>  find mails by subject text in all folders
    mc-debug trace <text> [period]      search postfix/rspamd/dovecot logs (30m, 3h, 2d; default 3h)

## Example: sync jobs stop running

Symptom: mails fetched via IMAP sync arrive late or not at all, but only sometimes.

1. `mc-debug sync` shows one job with `running = 1` and a large `min_ago`, while all other jobs are overdue too.
2. `mc-debug runner` shows a single long-running imapsync process holding the lock.
3. `mc-debug syncreport <id>` shows what that job is doing, e.g. thousands of messages being removed one by one
   on the remote side.

In our case the runner processed jobs strictly one after another, so one slow job delayed every other job.
The cause was a Gmail account that kept messages in "All Mail" after deletion, so the same mails were
compared, copied back and deleted again on every run.

## Notes

- Table and column names (`imapsync`, `returned_text`, ...) come from mailcow's database and may change between versions.
- `find` matches the decoded subject via `doveadm`; `trace` only finds what the services actually log
  (queue IDs, addresses, message IDs), usually not subjects.

## AI assistance

This tool was created with the help of AI assistants and is used by the author on a live mailcow installation.

## License

MIT
