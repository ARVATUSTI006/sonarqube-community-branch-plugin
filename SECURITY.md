# Security Policy

This repository is Passion Factory's fork of
[mc1arke/sonarqube-community-branch-plugin](https://github.com/mc1arke/sonarqube-community-branch-plugin).
See [UPSTREAM.md](./UPSTREAM.md) for how the fork relates to upstream.

## Supported Versions

Security fixes are provided for the latest release on the `main` branch only.

| Version              | Supported          |
| -------------------- | ------------------ |
| latest on `main`     | :white_check_mark: |
| anything older       | :x:                |

## Reporting a Vulnerability

**Please do not report security vulnerabilities through public GitHub issues,
discussions, or pull requests.**

Report it privately through
[GitHub private vulnerability reporting](https://github.com/passionfactory-oss/sonarqube-community-branch-plugin/security/advisories/new):
open the **Security** tab and select **Report a vulnerability**.

If you can't use GitHub, email **security@passionfactory.ai** instead.

Please include:

- A description of the vulnerability and its impact
- Steps to reproduce, or a proof of concept
- Affected versions, and any known mitigations

## Vulnerabilities in upstream code

Most of the plugin code comes from the upstream project. If the vulnerability is
in code that upstream also contains, please report it to the
[upstream project](https://github.com/mc1arke/sonarqube-community-branch-plugin)
as well, so that every user of the plugin gets the fix.

We ask that you give us a reasonable opportunity to address the issue before any
public disclosure.
