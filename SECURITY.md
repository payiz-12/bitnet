# Security policy

## Reporting a vulnerability

Please do not report security problems in public issues. Use GitHub's private
reporting instead: open the **Security** tab of this repository and choose
**Report a vulnerability**, or go to
https://github.com/payiz-12/bitnet/security/advisories/new.

Describe the problem, the versions it affects and, if you can, how to reproduce
it. You will get an answer within a few days. Once a fix is released, the
advisory is published with credit to the reporter, unless you prefer otherwise.

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
problems in the rules themselves need agreement across the whole network and
are better discussed with the community once they are safe to disclose.
