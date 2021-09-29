# My-Alternatives<br/>(hacking update-alternatives to make local changes)
[![MIT license](https://img.shields.io/badge/License-MIT-green.svg)](https://github.com/tekwizely/pre-commit-golang/blob/master/LICENSE)

My-Alternatives takes an [honest run](#miscellany) at making a useful tool to allow you to configure _alternatives_ (see: [update-alternatives](https://www.google.com/search?q=update-alternatives)) that only affect your local account / shell sessions.

**NOTES:**

My-Alternatives does not require _root / sudo_ privileges to use, as it creates and maintains user-owned _alt root_ directories that store your alternative links.

Skip down to _[configuraing an alternative](#configuring-an-alternative)_ to see an example how this works.

**CAVEATS:**
- Written in _Bash_
- Requires the following system tools:
  - `update-alternatives` 
  - `readlink`
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

#### TOC

- [Using](#using)
- [Installing](#installing)
- [Contributing](#contributing)
- [Contact](#contact)
- [License](#license)

--------
## Using

- [Initializing an Alt Root Directory](#initializing-an-alt-root-directory)
  - [Shell-Scoped Setup](#shell-scoped-setup)
  - [Specific Directory Setup](#specific-directory-setup)
  - [Re-Using a Specific Directory](#re-using-a-specific-directory)
- [Configuraing an Alternative](#configuring-an-alternative)
- [Reverting to System Alternative](#reverting-to-system-alternative)

---------------------------------------
### Initializing an Alt Root Directory

Before you can start using my-alternatives, you need to create &amp; initialize an `alt root` directory, where your alternatives will be stored.

You can configure a random, shell-scoped _alt root_ that only affects the current shell instance, or you can configure and re-use longer-lived roots.

You can also mix the two, configuring a user-level default _alt root_ with the ability to create and use shell-scoped roots as desired.

**NOTES:**
* The configured _alt root_ value will be placed in variable `MY_ALTS_ROOT`
* Additionally, both your shell's `PATH` and `MANPATH` values will be modified to include `${MY_ALTS_ROOT}/bin` and `${MY_ALTS_ROOT}/man`, respectively

#### Shell-Scoped Setup

The command for creating and initializing a random shell-scoped alt setup is:
```shell
$ eval "$( my-alternatives init )"
````

**Example: create &amp; init new shell-scoped (random) directory:**

_confirm not already defined_
```shell
$ echo ${MY_ALTS_ROOT?not defined}

bash: MY_ALTS_ROOT: not defined
```

_init my-alternatives_
```shell
$ eval "$( my-alternatives init )"
```

_confirm defined_
```shell
$ echo $MY_ALTS_ROOT

/tmp/my-alts.abcd
```

_check contents_
```shell
$ ls -l $MY_ALTS_ROOT

drwxr-xr-x  user  group  bin
drwxr-xr-x  user  group  man
```

_check path_
```shell
$ echo $PATH

/tmp/my-alts.abcd/bin:...
```

_check manpath_
```shell
$ echo $MANPATH

/tmp/my-alts.abcd/man:...
```

##### Specific Directory Setup

Although designed to support ad-hoc per-shell usage, there can be value in having a long-lived configuration.

A clear example would be a user-level setup shared across all user login shells.

The command for creating and initializing a specific alt setup is:
```shell
$ eval "$( my-alternatives init <alt_root> )"
```

**Example: create &amp; init with `~/.my-alts`:**

_confirm does not exist_
```shell
$ ls -l ~/.my-alts

ls: /home/user/.my-alts: No such file or directory
```

_init my-alternatives_
```shell
$ eval "$( my-alternatives init ~/.my-alts )"
```

_confirm defined_
```shell
$ echo $MY_ALTS_ROOT

/home/user/.my-alts
```

_confirm exists / check contents_
```shell
$ ls -l $MY_ALTS_ROOT

drwxr-xr-x  user  group  bin
drwxr-xr-x  user  group  man
```

_check path_
```shell
$ echo $PATH

/home/user/.my-alts/bin:...
```

_check manpath_
```shell
$ echo $MANPATH

/home/user/.my-alts/man:...
```

### Re-Using a Specific Directory

The command for re-using a previously-created alt setup is:
```shell
$ eval "$( my-alternatives shellenv <alt_root> )"
```

**NOTE:** You can place this _eval_ in your `.bash_profile` to configure a default _alt root_ on each login.

**Example: configure previously-created `~/.my-alts`:**

_confirm exists_
```shell
$ ls -l ~/.my-alts

drwxr-xr-x  user  group  bin
drwxr-xr-x  user  group  man
```

_confirm not already defined_
```shell
$ echo ${MY_ALTS_ROOT?not defined}

bash: MY_ALTS_ROOT: not defined
```

_configure my-alternatives_
```shell
$ eval "$( my-alternatives shellenv ~/.my-alts )"
```

_confirm defined_
```shell
$ echo $MY_ALTS_ROOT

/home/user/.my-alts
```

_check path_
```shell
$ echo $PATH

/home/user/.my-alts/bin:...
```

_check manpath_
```shell
$ echo $MANPATH

/home/user/.my-alts/man:...
```

### Configuring an Alternative

The command to configure an alternative is:
```shell
$ my-alternatives config <name>
```

This is meant to be equivelant to the `update-alernatives --config <name>` command.

Any _name_ available via `update-alternatives` should be usable.

**Example: configure local alternative for `pager`:**

_confirm current system value for pager_
```shell
$ which pager

/usr/bin/pager
```

_take a closer look_
```shell
$ ls -l /usr/bin/pager

lrwxrwxrwx root root  /usr/bin/pager -> /etc/alternatives/pager

$ ls -l /etc/alternatives/pager

lrwxrwxrwx  root  root  /etc/alternatives/pager -> /usr/bin/less
```

_init my-alternatives_
```shell
$ eval "$( my-alternatives init ~/.my-alts )"
```

_confirm alt root dirs initialized + empty_
```shell
$ find "${MY_ALTS_ROOT}"

/home/user/.my-alts
/home/user/.my-alts/bin
/home/user/.my-alts/man
```

_configure pager_
```shell
$ my-alternatives config pager

There are 2 choices for the alternative pager (providing ${MY_ALTS_ROOT}/bin/pager).

  Selection    Path            Priority   Status
------------------------------------------------------------
  1            /bin/more        50        
  2            /usr/bin/less    77        system value

Type selection number: 
```

_select non-system value /bin/more_
```shell
... Type selection number: 1

configured local alternative for pager: /bin/more
```

_confirm updated value_
```shell
$ which pager

/home/user/.my-alts/bin/pager
```

_review changes to alt root_
```shell
$ find "${MY_ALTS_ROOT}"

/home/user/.my-alts
/home/user/.my-alts/bin
/home/user/.my-alts/bin/pager
/home/user/.my-alts/man
/home/user/.my-alts/man/man1
/home/user/.my-alts/man/man1/pager.1.gz
```

_take a closer look_
```shell
$ ls -l "${MY_ALTS_ROOT}"/bin/pager "${MY_ALTS_ROOT}"/man/man1/pager.1.gz

lrwxrwxrwx  user  group  /home/user/.my-alts/bin/pager -> /bin/more
lrwxrwxrwx  user  group  /home/user/.my-alts/man/man1/pager.1.gz -> /usr/share/man/man1/more.1.gz
```

#### Reverting to System Alternative

To delete your local alternative, select the `system value` from the alternatives list:

**Example: revert to system alternative for `pager`:**

_revert to system value_
```shell
$ my-alternatives config pager

There are 2 choices for the alternative pager (providing ${MY_ALTS_ROOT}/bin/pager).

  Selection    Path            Priority   Status
------------------------------------------------------------
* 1            /bin/more        50        
  2            /usr/bin/less    77        system value

Press <enter> to keep the current choice[*], or type selection number: 
```

_select system value '/usr/bin/less'_
```shell
 ... or type selection number: 2

reverted to system alternative for pager
```

_confirm system value restored_
```shell
$ which pager

/usr/bin/pager
```

_review changes to alt root_
```shell
$ find "${MY_ALTS_ROOT}"

/home/user/.my-alts
/home/user/.my-alts/bin
/home/user/.my-alts/man
/home/user/.my-alts/man/man1
```

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

-----------------
#### _miscellany_

<dl>
  <dt>Honest Run</dt>
  <dd>Making an honest attempt to build a useful tool out of a fun hack</dd>
</dl>
