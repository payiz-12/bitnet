# Security policy

This is a community project, and its work happens in public.

## Reporting a problem

Report security problems like any other bug, in a public issue. That is the
right place for most of them: weaknesses that are hard to exploit, problems
that are already public (for example a known go-ethereum advisory), unsafe
defaults and configuration mistakes.

If you find a serious problem that is easy to exploit, such as a way to crash
or stall nodes remotely, to make them accept an invalid block, or to leak their
keys, you may instead report it privately, so that a fixed release is out
before the details are: open the **Security** tab of this repository and
choose **Report a vulnerability**, or go to
https://github.com/payiz-12/bitnet/security/advisories/new. Which way to
report is your choice.

Private reports are published in full, with credit to the reporter unless you
prefer otherwise, as soon as a fixed release is out.

Problems in code this build shares with the official Bitnet client, or with
go-ethereum, are welcome here as well; say if you have also reported them
elsewhere.

## Supported versions

Fixes go into the newest release. While a release is marked as a beta, the
previous release does not receive fixes.

## Scope

Anything that lets a remote party crash or slow down a node, make it accept an
invalid block or transaction, follow the wrong chain, leak its keys, or misuse
its RPC server. The consensus rules are the same as in the official client;
changing them needs agreement across the whole network.
