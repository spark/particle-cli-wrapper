# Particle CLI Wrapper & Installer

This tools is a wrapper around the [Particle CLI](https://github.com/particle-iot/particle-cli) that manages the version of Node.js used and auto-updates the Javascript modules.

# [Install through the Particle website](https://www.particle.io/cli)

_If you already have the CLI installed, uninstall it with `npm uninstall -g particle-cli`_

## Overview

**The Particle CLI Wrapper is a delicacy consisting of a light Go shell filled with a sweet creamy Node.js core.**

The Go shell manages its own Node.js installation. It is able to update itself, Node and Node modules. It forwards commands to the main Node.js CLI module. Since it is compiled natively for each platform it is easy to install.

The Node.js core is today's production Particle CLI. It performs all interactions with the Particle cloud.

## Architecture

The version of Node.js to be installed to run the CLI is selected in [`set-node-version`](/set-node-version). When this script is run, the installation files for Node.js and npm for Mac OSX, Windows and Linux are downloaded from <nodejs.org> then uploaded to <binaries.particle.io>

The CLI wrapper is compiled for Mac OSX, Windows and Linux with `rake build` and uploaded to <binaries.particle.io> with `rake release`

A manifest file is also uploaded at <https://binaries.particle.io/cli/master/manifest.json> with the latest version of the CLI wrapper for each platform and the Node.js version that should be installed.

When the CLI wrapper is run, it will download the manifest file. It will check if there's a new version of itself and download it. If the managed version of Node.js is missing or there is a new one, it will download and install Node.js. If the main Node.js module `particle-cli` is missing or there's a new version on npm, it will download and install it.

The managed version of Node.js, the modules and the CLI wrapper executable are stored in `~/.particle` or `C:\Users\name\AppData\Local\particle`

## Installer

See the [installer directory](/installer) for the source code of the Mac OSX, Windows and Linux installers that downloads the latest CLI wrapper and runs it once to make Node.js install.

See the [Mac and Linux installer README](/installer/unix/README.md) and [Windows installer README](/installer/windows/README.md) for more details.

## Development

- Install godep `go get github.com/tools/godep`
- Clone repo into your Go workspace. For example, `~/go/src/github.com/particle/particle-cli-wrapper`
- cd to that folder
- Install dependencies `godep get`
- Build executable `go build`
- Run `./particle-cli-wrapper`

## Releasing

See [RELEASE.md](RELEASE.md)

## Validations

Before releasing a new version to particle-cli-wrapper, perform these validation steps:

- Update the `beta` branch to the git commit to test
- Run the steps in `RELEASE.md` to publish a beta release

On Windows, Linux and macOS, run the following tests:
- Download the new particle-cli-wrapper
- Take a backup of the Node runtime `node-v*` and `node_modules` in `~/.particle` or `C:\Users\<username>\AppData\Local\particle`
- On a machine that ran the previous version of particle-cli-wrapper more than 4 hours ago (this is determined by the modification time of the `autoupdate` file), run the new particle-cli-wrapper. Expect the update to run in the background (confirm with `ps` or Task Manager). After the update is finished, `particle serial list` should work.
- Restore the copy of `node-v*` and `node_modules`. Run the old particle-cli-wrapper. Run the new particle-cli-wrapper. Expect a fast output since no update should happen immediately. `particle serial list` should work.
- Delete `node-v*` and `node_modules` to simulate a fresh install. Run the new particle-cli-wrapper. Expect a long wait. After the install, `particle serial list` should work.
- Restore the copy of `node-v*` and `node_modules`. Run the old particle-cli-wrapper. Run `particle update-cli` with the new particle-cli-wrapper. After the update, `particle serial list` should work.

_Note: the behavior of `particle serial list` is to show the Particle devices connected to the USB port. Even when no device is connected, the message should be `No devices available via serial`._

## Updating version of Node

- Update the version of Node and npm in [`set-node-version`](set-node-version)
- Run `PARTICLE_CLI_RELEASE_ACCESS=<aws-token> PARTICLE_CLI_RELEASE_SECRET=<aws-secret> ./set-node-version` (this will upload Node tarballs to binaries.particle.io for later retrieval by the CLI wrapper)
- Test upgrade to new version with extra verbose logging `go build && GODE_DEBUG=verbose PARTICLE_DEBUG=1 ./particle-cli-wrapper update-cli`
- Follow instructions in [RELEASE.md](RELEASE.md) to cut a beta release

## License

Copyright 2016 © Particle Industries, Inc. Licensed under the Apache 2 license.

[Based on the Heroku CLI](https://github.com/heroku/heroku-cli)

