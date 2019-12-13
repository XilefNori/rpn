<!-- # RPN - Bash tool for managing a set of repositories as a bunch -->

## What it does

You have lots of repositories which you want to manipulate as a bunch and see their status at glance

With `rpn` you can:
+ Execute `any command` for all _(or some of)*_ repos in your current list
+ See all repos state list at glance (`branch`, `ahead`/`behind`/`changes`, `origin`)
+ Find files in all _(or some of)*_ repos and _(if needed)_ `grep` them
+ Create any number of lists of repositories you want to manipulate as a bunch

\* Filtering can be done by: `name`, `branch`, `package`, `having changes` (_or not_)

`rpn` also has predefined commands to interact with `git`, `composer` and `docker` containers.

For `git` you can:
- Switch to branch of interest (fuzzy naming supported) with (if needed): fetching, reseting to origin
- View log list
- Show stastus
- Fetch updates
- View last set tags
- Etc...

For `composer` you can:
- Look which repo has which verson of composer package of interest
- Install packages

For `docker` containers you can:
- Run/stop/build/restart containers (you have to rewrite at-repo commands)
- Run php commands in containers like `composer install` or `bin/console doctrine:migrations:migrate`

## How it works

Every `rpn` command:
- Steps into each _(can be filtered)_ repo item listed in `list-directory`
- Run some shell code inside it

All you need is to organize your `bunch-of-repos` as a number of soft links to repo directories _(or real directories)_ in special `list-directory` and point it to `rpn`.

If you have more than one `bunch-of-repos` you can make special `groups-directory`  where every subfolder is a `list-directory` described above.

## Installation

1. Clone code to your HOME directory
```bash
git clone git@github.com:XilefNori/rpn.git ~/.rpn
```

2. Add `start-up code` to your .bashrc
```bash
cat << EOL | sed -r 's:^\s+::' >> ~/.bashrc
    source ~/.rpn/src/init.in
EOL
```

3. Set main setting: path to your current `list-directory`
```bash
cat << EOL | sed -r 's:^\s+::' >> ~/.bashrc
    N_REPO_DIR_ROOT=~/projects/_groups/main
EOL
```

That's it. `rpn` is ready to use.

### Additional settings

If you have more than one `bunch-of-repos`, you may want to set `groups-directory`:
```bash
cat << EOL | sed -r 's:^\s+::' >> ~/.bashrc
    N_REPO_DIR_GROUPS=~/projects/_groups
EOL
```

If you want you can set some custom settings:
```bash
cat << EOL | sed -r 's:^\s+::' >> ~/.bashrc
    N_REPO_QUITE=0
    N_REPO_NS_IMPORT=git,dc
    N_REPO_LINE_BRANCH_LEN=30
EOL
```

*NOTICE*:
**Set up your settings _after_ executing `start-up code`**

## Commands

Rpn commands are subcommands of `rpn` util. They are made as bash functions.

To get simple list of all available commands run:
```bash
rpn
```

### Repo state list

Run:
```bash
rpn l # Prints out repo-state-list
```

You will get repo-state-list (sample):
```
REPO: [ xxxxx     ]  1 hour [release/v2.4.0       ..M] (git@domain.com:xx/xxxxx.git)
REPO: [ xxxx      ]  6 day  [release/v2.3.1       ▲▼.] (git@domain.com:xx/xxxx.git)
REPO: [ xxxxxxx   ]  3 day  [release/v2.3.1-dev   ...] (git@domain.com:xx/xxxxxxx.git)
REPO: [ xxxxxxx   ]  7 day  [release/v2.3.1-fix   ...] (git@domain.com:xx/xxxxxxx.git)
REPO: [ xxxxxxx   ]  3 day  [release/v2.3.1       ...] (git@domain.com:xx/xxxxxxx.git)
REPO: [ xxxxxxxxx ]  6 day  [release/v2.3.1       ..U] (git@domain.com:xx/xxxxxxxxxx.git)
REPO: [ xxxx      ]  3 day  [release/v2.3.0       ...] (git@domain.com:xx/xxxx.git)
REPO: [ xxxxxxx   ]  6 day  [release/v2.3.1       12.] (git@domain.com:xx/xxxxxxx.git)
REPO: [ xxxxxxx   ]  2 day  [release/v2.3.1       ..M] (git@domain.com:xx/xxxxxxx.git)
REPO: [ xxxxxx    ]  9 day  [release/v2.3.1       3..] (git@domain.com:xx/xxxxxx.git)
REPO: [ xxxxxxxxx ]  4 week [master               ...] (git@domain.com:xx/xxxxxxxxx.git)
```

Description:
```
REPO: [ xxxxxxxx  ]  4 week [release/v2.3.1       6▼M] (git@domain.com:xx/xxxxxxxxx.git)
      ↑              ↑       ↑                    ↑↑↑         ↑
      repo           commit  branch               ||changes   origin
                                                  |behind
                                                  ahead

repo    - project directory name
commit  - last commit time ago
branch  - current branch
ahead   - number commits of commits ahead of remote branch (more than 9: ▲)
behind  - number commits of commits behind from remote branch (more than 9: ▼)
changes - M - having modified files, U - having uncommitted files
origin  - origin address
```

### Help system

To get list of available commands with short description run:
```bash
rpn help
```

To get help-info for specific command run:
```bash
rpn help <cmd> # prints help for <cmd> command
rpn <cmd> help # prints help for <cmd> command
```

If commmand does not have predefined help-info you will see its bash-code.

To see bash-code of the command run:
```bash
rpn .show <cmd> # prints <cmd> bash-code
rpn <cmd> .show # prints <cmd> bash-code
```

To get this manual page run:
```bash
rpn man    # Output format defined by 'view-man-as-man' option
rpn man -m # Man-page  format
rpn man -t # Text-page format (run if you don't like man-page)
```

### Main command

Main `rpn` command is `do` command which executes any bash code inside each repo
in current `list-directory`

Run:
```bash
# Prints out repo directory and git remote name repository
rpn do 'pwd; git remote'
```

`do` command has these options:

```
-- Filters --

-b <branch>   - execute on repos with branch '<branch>.*'
-r <repo>     - execute on with name <repo>, can be set multiple times (-r rp1 -r rp2 -r rp3)
-R <repo>     - execute on all filters repos but process repo <repo> first, can be set multiple times
-p <package>  - execute on repos which have composer package <package>
-g            - execute on repos which have local changes
-G            - execute on repos which do not have local changes

For -r|-R options you can set '.' value which mean 'current repo' (in $PWD)

-- Code --

-f <file>   - execute file <file>
-c <code>   - execute code <code>
-s          - execute as standalone script file
-S          - only print out code will be executed (useful for debugging)

-- Verbosity --

-Q  - 'quite-mode' is turned on if N_REPO_QUITE
-q  - Turn on 'quite-mode' (output is suppressed, and is showed only if return code is none zero)
-v  - Turn on 'verbose-mode'
-vv - Turn on 'verbose-mode' + 'debug mode' (every command in executed code is printed)

-- Output --

-l - print only repo state list
-h - print header (repo state list)
-H - print no header
-L - pipe output to less
```

Most of other commands can accept most of these options.

Many of other commands are just simple wrapper for `do` command and look like this:

```bash
n-repo-git.fetch()  {
    n-repo-do "$@" -Qs << 'EOF'
        printf -- "[ %-${max_length}s ] fetching ...\n" "$repo"
        git fetch
EOF

    n-repo-l
}
```

By default commands are executed in `subshell` .
This can be overriden by `-s` option which leads to executing code as a standalone bash script file.

### Filtering repositories

You can filter repos in which command will be run by:
- enumerating repo names                 (`-r repo1 -r repo2 -r repo3`)
- setting branch mask                    (`-b branch`)
- having/not-having changes in repo      (`-g/-G`)
- having some composer package installed (`-p package`)

### List of commands

```
-- Main --
l           - print repo state list
do          - execute any code inside each repo directory
find        - find files in current bunch-of-repos

-- Help --
man         - show rpn manual (README file)
help        - print command list with short description
help <cmd>  - print <cmd> command help-section
<cmd> help  - print <cmd> command help-section or bash-function body (if no help-section)
version     - print version

-- System --
cd <repo>       - cd to repo directory
.show <cmd>     - print <cmd> bash-function body
<cmd> .show     - print <cmd> bash-function body
.update         - update rpn code from github
.cd-group <grp> - cd to <grp> directory
.dirs           - print directories in current 'list-directory'
.hint.<xxxx>    - create and print 'hint-list' <xxxx>

-- Config --
cfg         - print environment configuration
root        - print or set current 'list-directory'

-- Composer packages --
package <name>  - print repo name and package version for repos which has package <name>
package.install - install composer packages

-- Git --
git.switch      - switch to branch (release/v4 -> release/v4.3.1)
git.last-tags   - show last created tags
git.origin      - reset current local <branch> to origin/<branch>
git.fetch       - fetch
git.log         - show log
git.di          - show diff
git.info        - show [header, git status, git log -1]
git.master      - reset to master
git.st          - show git status

-- Docker --
dc.build         - build docker containers
dc.restart       - restart docker containers
dc.stop          - stop docker containers
dc.up            - up docker containers
dc.env           - show diff between .env and .env.dist

-- Symfony inside docker (commands executed inside docker containers for each repo) --
code.clear-cache - clear cache for symfony project
code.install     - full install composer packages (with 'scripts' and 'platform-requirements')
code.migration   - run symfony migration
code.prepare     - code.(clear-cache + install + migration)
```

### Command completion

`rpn` is trying to support `tab-completion` anywhere it possibly can

```
# -- Common completions --

rpn <tab>        # Will propose user command
rpn .<tab>       # Will propose system command (started with a dot '.')

rpn help <tab>   # Will propose command with help-section or standalone help-section

rpn cmd -<tab>   # Will propose cmd-option

rpn cmd -b <tab> # Will propose list of all current branches
rpn cmd -r <tab> # Will propose list of repos
rpn cmd -R <tab> # Will propose list of repos
rpn cmd -p <tab> # Will propose list of packages

# -- Command specific completions --

rpn cd         <tab> # Will propose list of all repos
rpn find       <tab> # Will propose list of files (list is configured by find-hint option)
rpn package    <tab> # Will propose list of all possible packages
rpn git.switch <tab> # Will propose list of all possible branches
```

#### Command completion caching

Because generation of `hint-lists` can be time consuming, heavy `hint-lists` are cached in files in `$N_REPO_DIR_ROOT/.cache` subdirectory.

`hint-lists` generation commands are system and prefixed with `.hint` prefix.

For example for package list it is: `rpn .hint.package`

Cache files are kept due to `cache-lifetime` setup in cache-* parameters in seconds

For example `cache-package`=600 which means than `cache-file` with package-list will be regenerated after 600 seconds since it was (re)created last time.

To force recreation of `cache-file` you should run corresponding `hint-command`

You can see current cache settings by running:
```bash
rpn cfg cache
```

### Command namespace import

If you do not like to enter command prefixes such as `git.` or `dc.` and want to type
`rpn fetch` instead of `rpn git.fetch` you can import `command-namespace` to global namespace.

This is done by setting configuration parameter `ns-import`.

Namespaces are delimited by comma:
```
N_REPO_NS_IMPORT=git,dc
```

Once you did it you can run: `rpn switch` instead of `rpn git.switch`.

## Configuration

Configuration of `rpn` is made by environment variables with prefix `N_REPO`

To see all current configuration, run:
```bash
rpn cfg    # Output format defined by 'view-config-as-env'
rpn cfg -p # Configuration parameters format
rpn cfg -e # Environment variables format
```

Parameters are just more short format for environment variables:
```
dir-root <=> N_REPO_DIR_ROOT
```

To see only group of interest configuration, run:
```bash
rpn cfg cache # Show cache configuration
rpn cfg dir   # Show dir configuration
```

To set configuration parameter:
```bash
N_REPO_CACHE_BRANCHES=120  # Set environment variable
rpn cfg cache-branches 120 # Set parameter with 'cfg' command
```

*NOTICE*:
**Set up your configuration _after_ executing `start-up code`**

### Configuration parameters description

```
-- Directories --
dir-groups           path to `groups-directory`
dir-root             path to current `list-directory`
dir-install          path to rpn dist installed

-- Commands parameters --
ns-import            string of command namespaces imported to global namespace delimited by comma, sample: 'git,dc'
quite                default output mode is `quite-mode` (output is suppressed, and is showed only if return code is none zero)

-- Completion caches life-times in seconds --
cache-branches       all known branches     [in all repos]
cache-branches-cur   current branches       [in all repos]
cache-find-hint      file names             [in all repos]
cache-package        composer package names [in all repos]

-- Repo state list parameters --
line-ahead-sign           sing for ahead commits (when exceeds 9)
line-behind-sign          sing for behind commits (when exceeds 9)
line-branch-len           max length of branch name
line-repo-len             max length of repo name
line-long                 long format of `repo-state-list` (includes: origin or last commit message)

line-color-name           color of repo name
line-color-ahead          color of `ahead` symbol
line-color-behind         color of `behind` symbol
line-color-modified       color of `modified` symbol
line-color-untracked      color of `untracked` symbol
line-color-mod-untracked  color of `modified and untracked` symbol

-- Find command parameters --
find-depth                default -maxdepth of searching
find-hint                 list of directories in repo where to find files for `tab-completion`
find-short                print short file name (from repo subdirectory)

-- View config --
view-config-as-env        show configuration as ENVIRONMENT_VARIABLES (or config-parameters)
view-man-as-man           show manual as man page (filter README.md with `pandoc` and `nroff`)
```

### Default Configuration

```
dir-groups                ~/projects/_groups
dir-root                  ~/projects/_groups/main
dir-install               <path to rpn dist installed>

ns-import
quite                     1

cache-branches            60
cache-branches-cur        60
cache-find-hint           600
cache-package             600

line-color-ahead          $'\E[0;32m'
line-color-behind         $'\E[0;31m'
line-color-modified       $'\E[0;94m'
line-color-mod-untracked  $'\E[0;33m'
line-color-name           $'\E[0;94m'
line-color-untracked      $'\E[0;33m'

line-ahead-sign           ▲
line-behind-sign          ▼
line-branch-len           25
line-repo-len             15
line-long                 1

find-depth                4
find-hint                 ([0]="config" [1]=".docker")
find-short                1

view-config-as-env        0
view-man-as-man           0
```

## Naming history

There was a tool for repo manipulating called `rp` which is abbreviation of `repo`.
Then I made `n-repo` for manipulating a `bunch-of-repos` which leads us to `rpn`.

