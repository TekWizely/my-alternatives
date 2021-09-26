# My-Alternatives<br/>(hacking update-alternatives to make user-scoped changes)
[![MIT license](https://img.shields.io/badge/License-MIT-green.svg)](https://github.com/tekwizely/pre-commit-golang/blob/master/LICENSE)

My-Alternatives is an [honest run](#_miscellany_) at allowing you to configure [Debian alternatives](https://wiki.debian.org/DebianAlternatives) that only affect your local account / shell sessions.

**NOTES:**

My-alternatives does not require _root / sudo_ privileges to use, as it creates and maintains user-owned _alt root_ directories where your alternative links are stored.

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
  - `Ubuntu 20.04.3 LTS`
  - `Debian update-alternatives version 1.19.7.`

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

-------------------------------------
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

_example: create &amp; init new shell-scoped (random) directory_
```shell
# confirm not already defined
$ echo ${MY_ALTS_ROOT?not defined}

bash: MY_ALTS_ROOT: not defined

# init
$ eval "$( my-alternatives init )"

# confirm defined
$ echo $MY_ALTS_ROOT

/tmp/my-alts.abcd

# check contents
$ ls -l $MY_ALTS_ROOT

drwxr-xr-x  user  group  bin
drwxr-xr-x  user  group  man

# check path
$ echo $PATH

/tmp/my-alts.abcd/bin:...

# check manpath
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

_example: create &amp; init with `~/.my-alts`_
```shell
# confirm does not exist
$ ls -l ~/.my-alts

ls: /home/user/.my-alts: No such file or directory

# init
$ eval "$( my-alternatives init ~/.my-alts )"

# confirm defined
$ echo $MY_ALTS_ROOT

/home/user/.my-alts

# confirm exists / check contents
$ ls -l $MY_ALTS_ROOT

drwxr-xr-x  user  group  bin
drwxr-xr-x  user  group  man

# check path
$ echo $PATH

/home/user/.my-alts/bin:...

# check manpath
$ echo $MANPATH

/home/user/.my-alts/man:...
```

### Re-Using a Specific Directory

The command for re-using a previously-created alt setup is:
```shell
$ eval "$( my-alternatives shellenv <alt_root> )"
```

**NOTE:** You can place this _eval_ in your `.bash_profile` to configure a default _alt root_ on each login.

_example: configure previously-created `~/.my-alts`_
```shell
# confirm exists
$ ls -l ~/.my-alts

drwxr-xr-x  user  group  bin
drwxr-xr-x  user  group  man

# confirm not already defined
$ echo ${MY_ALTS_ROOT?not defined}

bash: MY_ALTS_ROOT: not defined

# configure
$ eval "$( my-alternatives shellenv ~/.my-alts )"

# confirm defined
$ echo $MY_ALTS_ROOT

/home/user/.my-alts

# check path
$ echo $PATH

/home/user/.my-alts/bin:...

# check manpath
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

_example: configure local alternative for `pager`_
```shell
# confirm current system value for pager
$ which pager
/usr/bin/pager

$ ls -l /usr/bin/pager
lrwxrwxrwx root root  /usr/bin/pager -> /etc/alternatives/pager

$ ls -l /etc/alternatives/pager
lrwxrwxrwx  root  root  /etc/alternatives/pager -> /usr/bin/less

# init my-alternatives
$ eval "$( my-alternatives init ~/.my-alts )"

# confirm alt root dirs initialized + empty
$ find "${MY_ALTS_ROOT}"

/home/user/.my-alts
/home/user/.my-alts/bin
/home/user/.my-alts/man

# configure pager
$ my-alternatives config pager

There are 2 choices for the alternative pager (providing ${MY_ALTS_ROOT}/bin/pager).

  Selection    Path            Priority   Status
------------------------------------------------------------
  1            /bin/more        50        
  2            /usr/bin/less    77        system value

Type selection number: 

# select non-system value /bin/more
... Type selection number: 1

configured local alternative for pager: /bin/more

# confirm updated value
$ which pager

/home/user/.my-alts/bin/pager

# review changes to alt root
$ find "${MY_ALTS_ROOT}"

/home/user/.my-alts
/home/user/.my-alts/bin
/home/user/.my-alts/bin/pager
/home/user/.my-alts/man
/home/user/.my-alts/man/man1
/home/user/.my-alts/man/man1/pager.1.gz

# take a closer look
$ ls -l "${MY_ALTS_ROOT}"/bin/pager "${MY_ALTS_ROOT}"/man/man1/pager.1.gz
lrwxrwxrwx  user  group  /home/user/.my-alts/bin/pager -> /bin/more
lrwxrwxrwx  user  group  /home/user/.my-alts/man/man1/pager.1.gz -> /usr/share/man/man1/more.1.gz
```

#### Reverting to System Alternative

To delete your local alternative, select the `system value` alternative from the list:

_example: revert to system alternative for `pager`_
```shell
# revert to system value
$ my-alternatives config pager

There are 2 choices for the alternative pager (providing ${MY_ALTS_ROOT}/bin/pager).

  Selection    Path            Priority   Status
------------------------------------------------------------
* 1            /bin/more        50        
  2            /usr/bin/less    77        system value

Press <enter> to keep the current choice[*], or type selection number: 

# select system value /usr/bin/less
 ... or type selection number: 2

reverted to system alternative for pager

# confirm system value restored
$ which pager

/usr/bin/pager

# review changes to alt root
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

#### Brew Core
TBD

#### Brew Tap

In addition to working on brew core support, I have also created a tap to ensure the latest version is always available:

* https://github.com/TekWizely/homebrew-tap

_install my-alternatives directly from tap_
```
$ brew install tekwizely/tap/my-alternatives
```

_install tap to track updates_
```
$ brew tap tekwizely/tap

$ brew install my-alternatives
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

---

###_miscellany_

<dl>
  <dt>Honest Run</dt>
  <dd>Making an honest attempt to build a useful tool out of a fun hack</dd>
</dl>
