# My-Alternatives<br/>(hacking update-alternatives to make local changes)
[![MIT license](https://img.shields.io/badge/License-MIT-green.svg)](https://github.com/tekwizely/pre-commit-golang/blob/master/LICENSE)

My-Alternatives is a light-weight wrapper over _update-alternatives_, using it to make changes to user-level configurations.

With my-alternatives, configuring custom alternatives is as easy as:

_home configuration example_
```shell
    # initialize my-alternatives
    # note: place this in your .profile
    $ eval "$( my-alternatives init )"

    # customize an alternative
    $ my-alternatives select <name>
```

Your selections will be saved into your `HOME` configuration, and made active in any login shell that performs the initialization routine.

*NOTE:* You can save the initialization routine in your `.profile`

### Per-Shell Configuration

If you want to make a temporary change that only modifies your current shell session, you can initialize a `TEMP` configuration:

_temp configuration example_
```shell
    # initialize a temporary configuration
    # note: your HOME configuration must already be initialized
    $ my-alternatives init-tmp

    # customize an alternative for just the current shell
    $ my-alternatives select <name>
```

### Auto-Import

The first time you `select` an alternative, my-alternatives automatically imports the alternative group into your configuration.

When the `HOME` configuration is the active configuration, my-alternatives imports the group from the system-level configuration.

When a `TEMP` configuration is active, my-alternatives first tries to import from your `HOME` configuration, falling back onto the system-level configuration if the group is not present.


### Manual Import

If you want to import an alternative group into your active configuration without selecting an alternative, you can use the `import` command:

_import example_
```shell
$ my-alternatives import <name>
```

### My-Alternatives Commands

Below is the list of custom commands that my-alternatives implements:

| Command            | Description 
|--------------------|------------
| `init`, `shellenv` | Prepare the current shell session for user-level alternatives.
| `init-tmp`, `tmp`  | Configure the current shell session for temporary (short-lived) changes.
| `select`, `config` | Select the active alternative for group <name>.  This is equivalent to `update-alternatives --config` with the adition of the auto-import logic.
| `import`           | Import an alternative group <name> into the current configuration.
| `add`              | Add an alternative to the group <name> within the current configuration.  This is equivalent to `update-alternatives --install` but has _slightly_ different syntax.  see `my-alternatives help add` for details.
| `rm-tmp`           | Remove the temporary configuration from the current shell session, making the `HOME` configuration active.
| `version`          | Display my-alternatives version number.

*NOTE:* See `my-alternatives help <command>` to learn about a specific command, including additional options.

### Update-Alternatives Commands

Below is the list of commands that are implemented as pass-through to the related update-alternatives command:

| My-Alternatives Command | Update-Alternatives Command
|-------------------------|----------------------------
| `select`, `config`      | `--config`
| `select-all`, `--config-all` | `--all`
| `rm-alt`                | `--remove`
| `rm-group`, `rm-grp`    | `--remove-all`
| `query`                 | `--query`
| `display`               | `--display`
| `list`                  | `--list`
| `set`                   | `--set`
| `auto`                  | `--auto`
| `get-selections`        | `--get-selections`
| `set-selections`        | `--set-selections`
| `ua-help`               | `--help`
| `ua-version`            | `--version` 

My-Alternatives will set the `--admindir` and `--altdir` options to point to your active configuration.

All additional command-line options are passed-through, unmodified.

*NOTE:* See `update-alternatives --help` or `man update-alternatives` to learn more about the various commands and their options.

### Invoking Update-Alternatives Directly

If you need to pass a specific command to update-alternatives, you can do so using the `ua` command:

_invoke update-alternatives directly_
```shell
$ my-alternatives ua --display pager
```

My-Alternatives will set the `--admindir` and `--altdir` options to point to your active configuration.

All additional command-line options are passed-through, unmodified.

*NOTE:* See `update-alternatives --help` or `man update-alternatives` to learn more about available commands and their options.

-------------
## Installing

### Releases

See the [Releases](https://github.com/TekWizely/my-alternatives/releases) page for downloadable archives of versioned releases.

### Git

```
git clone git://github.com/TekWizely/my-alternatives.git
```

---------------
## Contributing

To contribute to My-Alternatives, follow these steps:

1. Fork this repository.
2. Create a branch: `git checkout -b <branch_name>`.
3. Make your changes and commit them: `git commit -m '<commit_message>'`
4. Push to the original branch: `git push origin <project_name>/<location>`
5. Create the pull request.

Alternatively see the GitHub documentation on [creating a pull request](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request).

----------
## Contact

If you want to contact me you can reach me at TekWizely@gmail.com.

----------
## License

The `tekwizely/my-alternatives` project is released under the [MIT](https://opensource.org/licenses/MIT) License.  See `LICENSE` file.

----------------
##### Misc Notes

As of `v0.6.0`, my-alternatives is a complete re-write.  As much as possible, commands are implemented as pass-through to _update-alternatives_, pointing to your active configuration.

My-Alternatives does not require _root / sudo_ privileges to use, as it creates and maintains user-owned configuration directories.

**CAVEATS:**
- Written in _Bash_
- Requires the following system tools:
  - `update-alternatives`
  - `mktemp`
  - `umask`
  - `dirname`
  - `basename`
  - `manpath` _(optional)_
- Utilizes tmp files / directories
  - Sets `umask 077` for safety
- Tested on:
  - Ubuntu
    - `Ubuntu 20.04.3 LTS`
    - `Debian update-alternatives version 1.19.7.`
    - `Bash 5.0.17(1)-release`
  - openSUSE
    - `openSUSE Leap 15.3`
    - `SUSE update-alternatives version 1.19.0.4.`
    - `Bash 4.4.23(1)-release`
