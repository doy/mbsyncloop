# mbsyncloop

`mbsyncloop` combines [`mbsync`](https://isync.sourceforge.io/) and
[`goimapnotify`](https://gitlab.com/shackra/goimapnotify) to create a
persistent service which waits for new mail and immediately fetches it when
available, similar to [`offlineimap`](https://www.offlineimap.org/). At its
simplest, it just runs in a loop which fetches all mail every 15 minutes, but
it can be configured to watch some or all mailboxes using IMAP IDLE to
immediately trigger mail fetching as it comes in.

## Configuration

Configuration is read from `~/.config/mbsyncloop/config.json` if available. All
configuration is optional, although you will need a working `mbsyncrc` file
with certain things configured (described below). Valid keys for the mbsyncloop
config are:

```json
{
    "mbsync_config": "~/.mbsyncrc",
    "boxes": ["INBOX", "personal"],
    "box_patterns": ["^(?!old\\.)"],
    "on_new_mail": "notmuch new | grep -v '^No new mail\\.$'",
    "poll_interval": "900"
}
```

* `mbsync_config`: File to read `mbsync` configuration from. Defaults to
  `~/.mbsyncrc`.
* `boxes`: List of IMAP mailboxes to be immediately notified of changes to. If
  unset, defaults to watching all mailboxes on the server (although see
  `box_patterns`).
* `box_patterns`: If `boxes` is unset, `mbsyncloop` will request a list of all
  mailboxes on the server and use that list. If you set `box_patterns`, it will
  filter that list to only include mailboxes that match one of the (PCRE)
  regular expressions given by `box_patterns`. If `box_patterns` is unset, all
  mailboxes will be watched.
* `on_new_mail`: If set, this command will be run after `mbsync` completes
  successfully. By default, no command will be run.
* `poll_interval`: Number of seconds to wait between full syncs. Defaults to
  900.

In addition to `config.json`, `mbsyncloop` reads configuration from your
`mbsyncrc`. Your `mbsyncrc` should contain a `MaildirStore` and an `IMAPStore`,
and the `IMAPStore` should have `Host`, `User`, and `PassCmd` configured.
Additionally, `Port` and `SSLType` will be used if present. `mbsyncloop` will
extract those options and use them to configure `goimapnotify` to talk to the
same server.
