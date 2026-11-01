# Licensing and redistribution

This repository is a lab guide. It does **not** contain vendor images, license
files, or activation codes.

## What is in the repository

- Documentation, diagrams, and example configurations.
- No `.qcow2`, `.image`, `.bin`, `.iso`, `.unl`, or `iourc` files.
- The `.gitignore` blocks these file types and common secret patterns.

## Cisco IOS and IOL

Cisco IOS and Cisco IOL images are proprietary Cisco software. They are not
included here and must not be redistributed.

- Use only images you are entitled to use under your Cisco licenses or lab
  subscriptions (for example, Cisco CML/VIRL entitlements).
- Do not upload Cisco images to a public repository, file share, or container
  image.
- Do not share the `iourc` license file. It is hostname-bound and is a license
  artifact.
- Cisco IOS images may be subject to export-control laws. You are responsible
  for compliance.

## FortiGate VM

The FortiGate VM image is proprietary Fortinet software. It is not included
here.

- Use only a FortiGate VM image you are licensed to use.
- FortiGate VM evaluation licenses have throughput and feature limitations.
  Some features require a paid license.
- Do not redistribute the image or license files.
- Fortinet, FortiGate, FortiAnalyzer, and FortiManager are trademarks of
  Fortinet, Inc.

## Slax

Slax is a free/open-source Linux distribution. The Slax project's own license
terms apply to the distribution. The EVE-NG packaging is a separate artifact;
check its source and license before redistributing it.

## EVE-NG

EVE-NG Community Edition is distributed under its own license. This repository
does not include EVE-NG itself. Follow the EVE-NG license terms for your
installation.

## Documentation license

The documentation and example configurations in this repository are provided
under the MIT License. See [../LICENSE](../LICENSE).

The MIT License applies only to the material authored for this guide. It does
not grant any rights to Cisco, Fortinet, EVE-NG, Slax, or other third-party
software.

## Security notes

- The labs use telnet consoles inside EVE-NG. Do not expose EVE-NG or these
  consoles to an untrusted network.
- Change the default EVE-NG admin password.
- Do not commit passwords, API keys, license files, or private keys.
- If you fork this repository, keep vendor images and `iourc` out of the fork.

## No warranty

The guide is provided "as is", without warranty. Lab configurations can break
when images, EVE-NG versions, or host networking change. Validate in your own
environment.
