# Try the mailbox in Bash

Send a test message in one terminal, then claim and answer it from another.
This runs on your computer using synthetic data. You don't need an AI account,
an API key or a public tunnel. It doesn't test a live agent connection.

You'll need Bash, Git and Python 3.10 or newer with `venv`.
The example runs Dotline directly from a pinned checkout in an isolated Python
environment. Git needs internet access to fetch the source; no package install is needed.

## Terminal A: start the mailbox

```bash
umask 077
demo="$(mktemp -d "${TMPDIR:-/tmp}/dotline-demo.XXXXXXXX")"
cd "$demo"
git clone https://github.com/okgrit/dotline.git source
git -C source checkout --detach ec42408b8e476c2a27a69d662f4a288db8b4ce8a

python3 -m venv --without-pip .venv
export DOTLINE_HOME="$PWD/mailbox"
export DOTLINE_URL="http://127.0.0.1:18790"
export PYTHONPATH="$PWD/source/src"
unset DOTLINE_TOKEN_FILE

printf 'Demo directory: %s\n' "$PWD"
.venv/bin/python -m dotline init
.venv/bin/python -m dotline serve --host 127.0.0.1 --port 18790
```

`init` prints the token file's path, not its contents. Leave the token in that
private file. Keep this terminal open while the server runs. If port 18790 is
already in use, choose a different unused port in both `DOTLINE_URL` and
`--port`, then use that same URL in Terminal B.

## Terminal B: send, claim and answer

Open another Bash terminal and use the directory Terminal A printed.
The command substitution captures the actual message ID.

```bash
cd '/path/printed/by/terminal-a'
export DOTLINE_HOME="$PWD/mailbox"
export DOTLINE_URL="http://127.0.0.1:18790"
export PYTHONPATH="$PWD/source/src"
unset DOTLINE_TOKEN_FILE

.venv/bin/python -m dotline health
id="$(.venv/bin/python -m dotline send 'Synthetic local demo request' --topic demo)"
printf 'Sent message %s\n' "$id"
.venv/bin/python -m dotline pending --json
.venv/bin/python -m dotline claim "$id" --by manual-demo
.venv/bin/python -m dotline reply "$id" 'Synthetic local demo reply' --by manual-demo
.venv/bin/python -m dotline wait "$id" --minutes 0.1
.venv/bin/python -m dotline replies
```

Check that `pending` shows the sent ID, `wait` prints `Synthetic local demo reply`,
and the reply's JSON `to` field matches the sent ID. A successful `health` alone
doesn't test mailbox access.

## Stop and clean up

Press Ctrl+C in Terminal A. Close the two demo terminals to discard their
environment changes. The demo directory contains a plaintext bearer and
synthetic messages. Keep it private if you want to revisit it, or delete just
that verified `dotline-demo.*` directory after the server has stopped. Leave
your normal Dotline data and configuration alone.

Rehearsed on Linux with Python 3.12.14 and Bash using the pinned revision above.
The local server answered health, and the sent message was listed, claimed and
answered. The reply text and the original message ID matched. macOS was not tested.

See the [PowerShell walkthrough](LOCAL-DEMO.md) for Windows, or the
[security notes](../README.md#trust-and-security) before connecting a real agent.
