# Set system-wide environment variables for GUI applications

macOS has, for many years, not supported a simple way of setting environment variables that are visible to GUI applications. Environment variables that are configured in e.g. `~/.zshrc` are only visible to the terminal shell, not in GUI applications (except when launched via `open -a` from a terminal). Some applications offer workarounds, but this invariably means duplicating environment variable configuration in multiple locations.

This project contains a launch agent which attempts to remediate the situation, by reading the shell’s (specifically: `/bin/zsh`’s, which is macOS’ default login shell) configuration. The agent runs at startup and copies that environment into the user’s GUI domain. This is the environment that’s passed to a GUI application launched by launchd.


## Installation

To install, clone this repository and create the following symlinks from inside the repo directory:

```sh
mkdir -p ~/Library/bin ~/Library/LaunchAgents
ln -s $PWD/_gui_setenv ~/Library/bin
ln -s $PWD/me.klmr.env.plist ~/Library/LaunchAgents
```

Then, install the launch agent:

```sh
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/me.klmr.env.plist
```


## Usage

The environment is set automatically after the script installed, and after restart. However, when changing the shell configuration (adding/changing environment variables), these changes won’t be visible until the agent is rerun using


```sh
launchctl kickstart gui/$UID/me.klmr.env
```

Note that the environment of running applications will *never* be updated. That’s the same as for command-line applications, and is expected.


## Caveats

* **The launch order of login items is undefined.** This means that, after a restart, macOS may (*will*) launch some applications before this script gets a chance to initialise the environment. These applications may need to be relaunched to see the correct environment.

* Rerunning the launch agent via `kickstart` will add new environment variables and update existing ones, but it will never *delete* environment variables that have been removed. Remove these manually by running `launchctl unsetenv ‹VAR_NAME›`.
