# SAP NOTE DOWNLOAD

A Codex skill that downloads one or more SAP Notes from SAP for Me and recursively follows prerequisites matching an exact software component, From version, and To version.

## Requirements

- Codex with the in-app Browser available
- An SAP account authorized to access [SAP for Me](https://me.sap.com/)

The skill asks the user to sign in through SAP's website. Do not provide SAP passwords or other sign-in secrets in chat.

## Install

Copy the `sapnotedownload` folder into your Codex skills directory. On Windows, the destination is normally:

```text
%USERPROFILE%\.codex\skills\sapnotedownload
```

Restart Codex if the skill does not appear immediately.

## Use

Invoke:

```text
$sapnotedownload
```

The skill requests:

- one or more SAP Note numbers;
- Software Component;
- From version;
- To version;
- destination folder.

It reads the complete prerequisites table, follows every qualifying dependency without duplicate downloads or cycles, verifies each downloaded archive, and reports the dependency results.

This repository contains workflow instructions only. It does not contain SAP credentials or downloaded SAP Note content.
