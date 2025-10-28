+++
title = "Sane Claude Code installation"
description = "Getting started with configuring Claude Code on Linux"
tags = ["llm"]
+++

I'm just starting to learn to use Claude Code and, like most software,
it requires some fiddling with settings to have a relatively sane experience.
This is a short checklist of useful things.


| Action                                          | Reason                                                                  |
| ----------------------------------------------- | ----------------------------------------------------------------------- |
| Install through package manager                 | Avoid `npm` or installation bash scripts                                |
| `CLAUDE_CONFIG_DIR="${XDG_CONFIG_HOME}"/claude` | Respect the XDG spec                                                    |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1`    | Disable auto-updates, error reporting, telemetry and the `\bug` command |

Relative to Claude Code version 2.0.28.
