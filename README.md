# dbxbin
## Dropbox binary syncing tool

**MinGW is not supported**

Do you store config files, RC files, or custom scripts in Dropbox? And you want these to seamlessly sync across operating systems? Well, then you've already got what you need. Bye!

Haha, JK! Do you ever think to yourself, "Gee, I could just symlink all of these files to my dropbox and everything would work magically. But then if one computer is compromized, suddenly I'm syncing compromized files all over the place!"? Dbxbin is the answer!

The goals of dbxbin are:
  * Self updating - the script can update itself
  * Stupid-simple setup - once configured, installing dbxbin on a new system is a single command
  * Relatively secure - entries are not symlinked to Dropbox (unless configured as such) and dbxbin presents the user with a diff so unwanted changes can be detected
  * Minimal (necessary) configuration - in most cases, all you need is two files: dbxbin and your config
  * Straight forward configuration - for each file you want to sync, you only need a source and a destination
  * Highly configurable
    * The script sources the config file, thus the config is itself a bash script.
    * The script exposes variables and functions that can be used to make OS-dependent entries and other such things (see the example.config).
    * Entries have an optional CMD setting that allows for customization, such as symlinking or installing to a protected directory (e.g. `/usr/local/bin`).
    * The script (by default) exposes various variables that point to locations such as the user's home, root's home, the local usr bin folder, etc.
    * Each system has it's own env file that can be used for system-specific setup. It is not synced.

## Installation

### Easy

Designate (and/or create) a folder in dropbox for syncing private settings. Create a folder inside that folder for scripts you may want to share with a friend (historical basis for the path expectations). Copy/download dbxbin into that folder. In bash (cygwin ok, mingw not), run `~/Dropbox/MySettings/bin/dbxbin setup`. Now, in the settings folder (e.g. `~/Dropbox/MySettings`), create `dbxbin.config` and add the following:

```
function add_local_bin {
	su_cp "$@"
	su_chmod "$@"
}

SRC[config]=/dbxbin.config

SRC[dbxbin]=/bin/dbxbin
DEST[dbxbin]=$LOCAL_BIN/dbxbin
CMD[dbxbin]=add_local_bin
```

You probably also want to add your own entries, such as:

```
SRC[zshrc]=/zshrc
DEST[zshrc]=$USER_RC/.zshrc
```

More examples in `example.config`. BOOM! DONE! Unless you want to update your env file (`~/.dbxbin/env`). But the default one is good in most cases. Now you just have to run `dbxbin update` whenever you want to sync.

### Custom

Copy dbxbin somewhere (in Dropbox, is the intention). Also somewhere, probably in your dropbox, make `dbxbin.init` (default name) or whatever you want to call it. You probably also want to create `dbxbin.config` and maybe `dbxbin.env`. Now run `~/Dropbox/MySettings/bin/dbxbin setup ~/Dropbox/path/to/dbxbin.init`. If you don't specify the location of dbxbin.init, dbxbin looks for `../dbxbin.init`, relative to itself.

Init does the following:

  1. Creates `~/.dbxbin`.
  2. Sources dbxbin.init (`$INIT_FILE`), if found (if dbxbin.init is not specified and ../dbxbin.init does not exist, defaults are used for everything)
  3. Copies `$DBX_ENV_SRC` to `~/.dbxbin/env` (hard coded, can't be changed) or generates a default env file (same path).
  4. Copies `$DBX_CFG_SRC` (defaults to `../dbxbin.config`) to `$DBX_CFG_DEST` (defaults to `~/.dbxbin/config`)

Setup calls init and then update. Update copies everything to its destination (or whatever CMD[entry] says to do).

The default env file is written into the script, so no other files are necessary. But `default.env` is a representation of the default env (but not exact since the default env varies by OS) and should be up to date. If you're worried about that last part, look in the script.

## Usage

```
% dbxbin help
Usage: /usr/local/bin/dbxbin VERB [ARGUMENT] ...
Manages scripts stored in and synchronized by Dropbox. The exact behavior is
dependent on the verb.

  help [VERB]
     Displays this message or shows help for a specific verb.

  usage [VERB]
     Displays a short usage message for dbxbin or for the specified verb.

  init [INIT_FILE]
    Initializes the local dbxbin configuration.

    Sources INIT_FILE and sets up the dbxbin home directory. Creates ~/.dbxbin,
    copies DBX_ENV_SRC to ~/.dbxbin/env, and copies DBX_CFG_SRC to DBX_CFG_DEST.

    INIT_FILE defaults to '~/Dropbox/MySettings/bin/dbxbin.init'. If INIT_FILE
    does not exist, defaults are used for everything.

    DBX_ENV_SRC defaults to '~/Dropbox/MySettings/bin/dbxbin.env'. If
    DBX_ENV_SRC does not exist, a default file is used.

    DBX_CFG_SRC defaults to '~/Dropbox/MySettings/bin/dbxbin.config'.

    DBX_CFG_DEST defaults to ~/.dbxbin/config.

  reinit [INIT_FILE]
    Reinitializes the local dbxbin configuration.

  check [OPTIONS] [ENTRY] ...
    Returns a list of any out of date entries.

    Sources config and env from ~/.dbxbin and returns a list of out of date
    entries. If no entries are specified, all entries are checked.

    -z --skip
      Check all entries, skipping the specified entries.

  sync ENTRY ...
    Syncrhonizes the specified entries.

    For each entry, displays the file in Dropbox to the user with 'less' and the
    asks the user if the entry should be synchronized. If the user says yes, the
    destination file is deleted and replaced by the Dropbox file. The check is to
    prevent security vulnerabilities from propagating through Dropbox.

  setup [INIT_FILE]
    Initializes and updates

  update [OPTIONS] [ENTRY] ...
    Checks for out of date entries and syncrhonizes them.

    Same options as check.

  info ACTION [ARGUMENT] ...
    Provides information on the local dbxbin configuration.

    src|source [ENTRY]
      Prints the source path for the entry.

    dst|dest|destination [ENTRY]
      Prints the destination path for the entry.

    cmd|command [ENTRY]
      Prints the command for the entry.
```

## FAQ

  * What entries are required?
    * Absolutely none! Technically, you don't need a config file, but that would kind of defeat the point. It is **highly recommended** that you have an entry for your config file, because otherwise you aren't syncing it or you're pointing dbxbin to Dropbox. And that last is bad because dbxbin sources the config file, so you're making yourself vulnerable.
    * If you are constantly tweaking dbxbin itself, you may want to add an entry for it.
    * `Ethan`: I have entries for both (config and dbxbin), and as such I haven't tested not having them. So it may get weird. But I'll fix it if you tell me.
  * If I'm using the default config file and not pointing dbxbin directly to dropbox (as in `DBX_CFG_DEST=/some/where/in/dropbox`), won't I have to run dbxbin multiple times to get everything updated?
    * If you run `dbxbin update`, the first (actually second) thing dbxbin does is check if the config has been updated. If it has, it is re-sourced. Only then are the other entries checked. (The actual first thing is checking if dbxbin needs updating).
  * Why isn't the env file synced?
    * You can totally sync it. However, the env file is intended to be host-specific, whereas the config file is intended to be host-independant, so it's not recommended.
    * `Ethan`: I'm not syncing it, so it hasn't been tested, so weird stuff may happen.
  * What if I don't want a specific entry on a specific machine?
    * Add any entries you want to ignore to the IGNORE variable in that machine's env file (if using the default, remember to uncomment it and remove the dummy entry1 and entry2)
  * What if [some more complicated entry criteria]?
    * The config file is a bash script. If the entry isn't defined when the config file is sourced, then dbxbin won't see it.
  * What is all this about bash and sourcing?
    * `source` is a pretty straight forward bash builtin. Instead of executing the script in a new shell, bash executes the script in the current shell. [Here](http://superuser.com/a/176788) is a pretty thorough explanation.
    * If you're not familiar with bash, this tool is not friendly enough for you.
    * If you really want to know, I can find some good (IMHO) tutorials.

## Known bugs & limitations

  * **NO MINGW SUPPORT**
    * The main developer ran into problems making everything work in MinGW and promptly stopped caring, as he had Cygwin and it's better in every way (IMHO).
    * The issue he remembers ATM is sudo support. Cygwin has `cygstart --action=runas`.
  * Not tested under OSX
    * It will probably work though.
  * Only one person has actually used this to date
    * Let me know!
  * Slow in Cygwin
    * Windows doesn't have real fork() support, so [Cygwin's implmentation](https://www.cygwin.com/faq.html#faq.api.fork) is slow, and bash uses fork() for *everything*, and dbxbin is written in standard bash style, so it's slow.
  * No real directory syncing support
    * If you change the command for an entry, you might get directory syncing to work, but the comparison function won't. If there's demand, this may be added.
  * The whole 'self updating' thing is problematic
    * When you change a file that bash (or zsh) is currently executing, bash gets weird. It interprets the file line by line without reading the whole thing into memory, so when you modify the file, suddenly the executing script is inconsistent.
    * Fortunately, this has never actually caused a problem (that's been reported). You may have to re-run the command.
    * However the current version of dbxbin exits immediately after updating itself with the message `dbxbin updated, please re-run`, so that should mitigate the issue.

## Planned features

  * Steam support
    * Partial implementation
    * Adds a steamapp_path function that takes an app ID and outputs the location of it's install folder, or nothing if the app or if steam is not installed.
    * Allows for things like, "if Terraria is installed, symlink it to Dropbox"
