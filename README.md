<br>

# [cli-best-practices](https://hackmd.io/@arturtamborski/cli-best-practices)


Collection of best practices for CLI tools 👩‍💻

:black_small_square: 
[view on github.com](https://github.com/arturtamborski/cli-best-practices)
:black_small_square: 
[view on hackmd.io](https://hackmd.io/@arturtamborski/cli-best-practices)
:black_small_square: 


<br>

# 1. BASICS

This section points out a minimal set of necessary requirements to meet in order to make the CLI friendly and usable. Think `cargo` instead of `gcc`, or `httpie` instead of `curl`.


<br>

## 1.1. Make sure that your tool works

That may sound obvious but it's very important that your tool works. It prints something to stdout, it does not trip on it's own flags and that these flags actually are documented and have an effect on the application.
This point is intentionally obvious, because it's 'obvious', so you might not focus on it a lot and just assume you meet it, but to your user - it's the first and most important rule one sees when interacting with the tool.

Imagine how terrible it would be if you'd run a fresh'n'cool CLI tool only to see it breaking imidiately with some stack trace or even worse - obscure, undocumented error. 

:sparkles: friendly tip :sparkles:
Consider adding at least some simple test scenario to your CI release pipeline, just to be sure that the CLI at least works. Might not be fully functional, but it has to at least work.

### Example

Bad

```shell
$ my-cool-tool
Stack trace: [...]
```

Good

```shell
$ my-cool-tool
Error: no input specified! Please use --help to find out more.
```

You should aim much higher than that, but this is sensible starting point for your tool.


<br>

## 1.2. Document all public flags

Every flag that's available to the user must be documented. It is discouraged to have hidden flags but if that's the case - make sure that hidden flags are not used as an example in documentation or do not show up in answers.

### Example

https://stackoverflow.com/a/24470998  
https://stackoverflow.com/a/31092052  
https://cgold.readthedocs.io/en/latest/glossary/-H.html

```shell
cmake -H # set home directory
cmake -B # set build directory
```

Both flags are undocumented, yet incredibly useful, so they are used and do appear
in various online answers and code examples.


<br>

## 1.3. Follow well established conventions

Don't reinvent the flags with your fancy syntax like `-Cool_Flag:someValue`. It's confusing for everyone, including you.

(Suggested by zapier.com)

### Example

Bad

```shell
$ 7z a ../lambda.zip -xr'!.venv' .

# to ignore directory you have to specify -x (exclude) and -r (recursive)
# and '!<directory>' -- to exclude it again?
# man 7z: Do not use "-r" because this flag does not do what you think.
```

Good

```shell
$ zip -r ../lambda.zip -x '.git' .
$ zip -r ../lambda.zip --exclude='.git' .  # alternative version

# much cleaner :)
```


<br>

## 1.4. Use exit codes

Use, document, and inform the user about them.


### Example

```shell
$ ls /usrer/
ls: cannot access '/usrer/': No such file or directory
$ echo $?
2
$ man ls
   [...]
   Exit status:
       0      if OK,
       1      if minor problems (e.g., cannot access subdirectory),
       2      if serious trouble (e.g., cannot access command-line argument).
```


<br>

## 1.5. Always provide `-h/--help` flag

Your tool has to provide some built-in way of informing the user what it does and how to use it. That's the minimum requirement but it's absolutely necessary. Man pages aren't always installed.

### Example

Bad

```shell
$ 411toppm -h
option `--height' requires an argument
$ 411toppm --help
411toppm: Use 'man 411toppm' for help.
```

Good

```shell
$ xz -h  # or xz --help, it's the same output
Usage: xz [OPTION]... [FILE]...
Compress or decompress FILEs in the .xz format.

  -z, --compress      force compression
  -d, --decompress    force decompression
[...]
```


<br>

## 1.6. Learn from others, copy the best

[`gh-cli`](https://cli.github.com/manual/index), 
[`heroku cli`](https://devcenter.heroku.com/articles/heroku-cli-commands), 
[`httpie`](https://httpie.io/docs/cli/examples),
[`cargo`](https://doc.rust-lang.org/cargo/commands/cargo.html)

<br>

## 1.7. Keep the tool name short, unique and easy to remember

If that's not possible consider supporting alternative binary name (`angular-cli` → `ng`).


This point is highly specific to your case, it might be that you're building a collection of related CLI tools, where it makes sense to name the binary in a namespaced manner, like `gnome-session` / `gnome-session-new-session` / `gnome-session-properties`...


<br>

## 1.8. Tool has to be predictable and idempotent

In other words, it should behave the same way no matter the time of execution or other external factors.

For expanded explanation, see:
- https://daniel.haxx.se/blog/2021/12/06/no-easter-eggs-in-curl/ 
- https://unix.stackexchange.com/questions/405783/why-does-man-print-gimme-gimme-gimme-at-0030
- https://unix.stackexchange.com/a/406169
- https://labs.detectify.com/2012/10/29/do-you-dare-to-show-your-php-easter-egg/
- https://arslan.io/2019/07/03/how-to-write-idempotent-bash-scripts/


<br>

## 1.9. Tool has to be task oriented

The idea here is to use the tool to fulfill some need (like a need to copy a file, copy a repository, or format a filesystem). Your CLI tool might be specific to the service you're offering (like a `gh-cli`) in which case there are many tasks you can achieve, but because `gh-cli` is task oriented, it's easy to execute desired action.

### Example

```shell
$ gh --help
Work seamlessly with GitHub from the command line.

USAGE
  gh <command> <subcommand> [flags]

CORE COMMANDS
  browse:     Open the repository in the browser
  codespace:  Connect to and manage your codespaces
  gist:       Manage gists
  issue:      Manage issues
  pr:         Manage pull requests
  release:    Manage GitHub releases
  repo:       Create, clone, fork, and view repositories

ACTIONS COMMANDS
  actions:    Learn about working with GitHub actions
  run:        View details about workflow runs
  workflow:   View details about GitHub Actions workflows

ADDITIONAL COMMANDS
  alias:      Create command shortcuts
  api:        Make an authenticated GitHub API request
  auth:       Login, logout, and refresh your authentication
  completion: Generate shell completion scripts
  config:     Manage configuration for gh
  extension:  Manage gh extensions
  gpg-key:    Manage GPG keys
  help:       Help about any command
  secret:     Manage GitHub secrets
  ssh-key:    Manage SSH keys

FLAGS
  --help      Show help for command
  --version   Show gh version

EXAMPLES
  $ gh issue create
  $ gh repo clone cli/cli
  $ gh pr checkout 321
```


<br>

## 1.10. Tool has to be friendly people and scripts

This point is often missing, especially in older CLI tools which were designed for proficient CLI users but with little thought for use in automated scripts.
That's why tools like `cut`, `awk` and others are so often seen in bash scripts - it's the only way to convert human readable output to machine readable format used by other tools.

### Example

Bad

```shell
$ docker ps -a | grep alpine | cut -c 1-12
# get container IDs which are based on alpine image
```

This command is very fragile to output-specific behaviour - it can easily fail if the output changes and it also isn't exactly correct, because `grep alpine` will only find the obvious cases, but it will not show containers which are based on apline by inheritance.

Good

```shell
$ docker ps -a --filter=ancestor=alpine --format "{{ .ID }}"
# get container IDs which are based on alpine image
```

Docker CLI was specifically designed to be parseable by scripts without the impact on user experience (when output changed for some reason). It will work reliably every time and is more correct, because it will check the actual dependency on alpine image instead of just searching it in the output.



<br>

## 1.11. Tool has to be high quality

It's as important as your API. Actually, it *is* your API, but for humans.


<br>

## 1.12. Commands that read like sentences are easier to remember


<br><br><br>

# 2. FLAGS


<br>

## 2.1. As always, Consistency is the Key

By far the most commonly used (and thus the *only good way)* of writting flags is with the dash-case. Do not use anything else. Yeah, java uses `-Xmx`, but it's their problem, not your option. Do not use anything else other than `--option-name`. Please.


<br>

## 2.2. This applies to environment variables as well

`GOOD_ENV_VAR_NAME` vs `BAD_envVarName`. Yes, they are case sensitive, no, you should not leverage that. Please stick with upper casing or `SCREAMING_SNAKE_CASE` if that convinces you more.


<br>

## 2.3. Every flag that can have a default value, should have default value

Defaults have to be sensible and meaningful. If you are not sure, better to leave it off and come back after reasuring yourself on the most commonly used value.


<br>

## 2.4. Provide long options first, then shorten the most commonly used ones


<br>

## 2.5. Don't use `-longopt`, instead use `--longopt`

Copypaste of someones opinion:

I hate `-long` style options too. It's not just aesthetic/confusing, it's objectively worse/more limited:

- It means `-thing` != `-t -h -i -n -g`, so the latter has to be that long;
- You also can't have `readable-long-things-as-an-option`;
- You can't (or not without crazy logic and a confusing UX) have variables passed with no space, like `-ffoobar`
- 


<br>

## 2.6. Named flags should be position independent

```jsx
ls --list --all .
# is the same as...
ls --all --list .
```


<br>

## 2.7. Long named flags > short named flags > positional arguments

```jsx
find -R . -name '*.txt' -type f
vs
find . -R -n '*.txt' -f
vs
find . r '*.txt' f
```


<br>

## 2.8. List of common conventions often used in CLI tools

- `-` for marking stdin / stdout
- `--` for marking end of arguments
- `--longopt=somevalue` for passing values with long flags
- `-s somevalue` for passing values with short flags
- `--pointer` / `--no-pointer` prefix flags with `no` for switch-like functionality
- `-f` / `--file` for selecting file
- `-o` / `--output` for specyfing output file
- `-i` / `--input` for specyfing input file
- `-h` / `--help` for displaying help
- `-n` for dry run
- `-v` / `--verbose` for increasing verbosity
- `-q` / `--quiet` for decreasing verbosity
- `-V` / `--version` for showing version
- [http://docopt.org/](http://docopt.org/)

Commonly established conventions take priority.

```jsx
-v -> verbose / version
-r -> reverse / recursive
-a -> all / append
-l -> list / lines
-n -> n things
```

`--recursive -> -r` / `--reverse -> -r`

`--all -> -a`


<br>

## 2.9. Allow passing sensitive values trough multiple channels

Flags are not always the best option, for example `--password=qwerty123` really won't work. There has to be a way to pass sensitive data without it being shown to the user.
A good rule of thumb is to expect your tool to be used in public CI pipeline or shown in youtube tutorial.
If you have to obfuscate the secrets in your docs to show the option then you're probably doing it incorrectly, because every user will have to keep that in mind as well.

`env MY_TOOL_SECRET | my-tool --password -` is a good way of sending the input secretly. 


<br>

## 2.10. If your tool is big, split it into subcommands

```jsx
aws [global flags] acm [acm specific args] [acm specific flags]
aws [global flags] s3 [s3 specific args] [s3 specific flags]

aws --region=us-east-1 s3 ls --bucket=test
```


<br><br><br>

# 3. USER EXPERIENCE


<br>

## 3.1. Inform the user early


<br>

## 3.2. Don't go for a long period without output to the user

If you do print status messages like that, make sure to send them to STDERR if your utility outputs any actual data (like a report or file listing). Same for progress meters.


<br>

## 3.3. If a command has a side effect provide a dry-run/whatif/no_post option


<br>

## 3.4. For long running operations, allow the user to recover at a failure point if possible


<br>

## 3.5. Support scaffolding, if applicable


<br>

## 3.6. Support autocompletion


<br>

## 3.7. Make it easy to install and easy to update


<br>

## 3.8. Suggest commands on typing errors (`git satus` → `Did you mean git status`)


<br>

## 3.9. Always do the least surprising thing

`rm -rf /*` - rm has to be aware what might happen an try to block the user. It's too destructive to accept it without confirmation.

`pkill -v <some process id>` - `v` stands for "inverse input", so this actually kills every process except the one passed in input. Why?


<br>

## 3.10. Repair what you can — but when you must fail, fail noisily and as soon as possible


<br>

## 3.11. Design for the future, because it will be here sooner than you think


<br>

## 3.12. Aliases provide balance between brevity and discoverability

Consider splitting your cli into `Resources: ` as in what you can achieve and `Aliases` as in list of shortcuts to do stuff.


<br>

## 3.13. Piping is good for automation but people don't want to pipe

If its a common operation then provide a dedicated command for it. Either you will or every user will wrap some commands with subshells and pipes without setting ‘set -o pipefail”


<br>

## 3.14. Default to human-first output, but support multiple (json ftw)

Optimize output for humans, `10 days ago` > `2019-07-15T14:32:22Z`

You can use ISO for JSON :)


<br>

## 3.15. Avoid positional arguments where the order matters

which one is correct and why?

```jsx
emote add repo funk https://x.com/funk.json
vs
emote add repo https://x.com/funk.json funk
```

answer: neither! it's not obvious, you have to check docs to make sure = bad UX.

```jsx
emote add repo funk --url https://x.com/funk.json
```

command subcommand NAME —more-flags 


<br>

## 3.16. Positional arguments are cool when the order doesn't matter

```jsx
emote repo delete funk more-funk
vs
emote repo delete more-funk funk
```


<br>

## 3.17. Expect that user will try to stop the tool at the worst moment (eg. during lengthy process like cloning a repo)

Be sure to not fail unexpectely and to not leave trash after unfinished action. You can overwrite action on CTRL+C to clean up right before the program finishes.


<br><br><br>

# 4. ACCESSIBILITY


<br>

## 4.1. Keep the output short

Try to not go over 80 characters per line of output

Bad

```bash
$ ls --help
[...]
  --all -- this option allows you to print every file in given directory specified in the input so that you can see every file in local directory even if its hidden.
```

Good

```bash
$ ls --help
[...]
  --all -- print every file including hidden ones
```


<br>

## 4.2. `$NO_COLOR` support

The CLI tool should support commonly used modifiers for it's output.
See [https://no-color.org/](https://no-color.org/) for more information.


<br>

## 4.3. `--verbose` support

Preferably with some way of increasing/decreasing it.


<br>

## 4.4. Localization support

Common pitfal: [http://www.scalingbits.com/aws/CLI](http://www.scalingbits.com/aws/CLI)


<br>

## 4.5. Translations

I don't actually know much about that and how it should work... 


<br>

## 4.6. `$DEBUG` support

Tool should provide sensible way to debug it.


<br>

## 4.7. Don't use too many colors and don't expect users have more than 16 colors by default

CLI tools don't control their background so the ouput must be background-color agnostic. There shouldn't be any asumption as to the color scheme used by the user, instead all colors should be treated as hints only. Eample: use bold to visibly separate short important parts (like headers),

If in doubt, change the color to default and say what you have to say, most often than not this will end up looking better than some combination of inverse bold text with custom background color.

Unless your application controls the whole terminal (like vim, mutt, etc) it's just not worth playing too much with colors because you're raising your chances of readable output which should be your priority.

More info on colors: [https://accessibility.psu.edu/legibility/contrast/](https://accessibility.psu.edu/legibility/contrast/)


<br>

## 4.8. Decrease use of emojis to minimum and stick to the old and well known emojis

Think clearly on what you're trying to communicate, too many emojis may obfuscate this where a few words would do. Use emojis to transfer emotions and common behaviours (hand waving to say hello, confetti to express joy or sucess) instead of using them as pictograms with some underlying meaning. This meaning might not be valid in other cultures (praise hands is a good example) or it might not mean what you thought it means. If you have to stop and think what your output with emoji means then you should change that to clarify your intentions.

Another issue is with support - many fonts  fonts don't support recent emojis or are visualised differently across operating system (consider every font used in every CI pipeline + every monospaced font on commonly used OS + some of these are not update (Ubuntu 12? Ubuntu 10? Old mac? They all will have problems with emojis) Use [emojipedia.com](http://emojipedia.com) to verify that emojis are actually representing the same thing and same meaning.


<br>

## 4.9. Use gender neutral language

[https://man7.org/linux/man-pages/man7/man-pages.7.html](https://man7.org/linux/man-pages/man7/man-pages.7.html)


<br>

## 4.10. Follow american spelling convention

color > colour, etc. It's just more common and causes less confusion, don't take this personally.


<br>

## 4.11. Support diagnostic flag of sorts

When your tool fails someone will try to report it, 

(find example of issue where they ask to rerun the command with some diagnostic switch for easier debugging


<br>

## 4.12. Use short, terse and simple language for describing the program's behaviour

Do not use slang-specific words if they obfuscate otherwise simple meaning. Try to keep the program-specific slang or brand-specific slang to minimum. One example of that is `yargs` which uses pirate-like terminology to explain its meaning and create a brand around it. It's okay to do so as long as it doesn't spread accross the whole help page or documentation.

Simple things should be named simply, known things should be named with known words. Do not reinvent the common terms because you will not be understood by most of the world.

Example: some programs support plugins / addons / extensions and then there's ansible galaxy.


<br>

## 4.13. Be aware of your users, their backgrounds and your environment

Go commands often use `get` for downloading, so your tool should also use it if it needs that feature.

Try to discover, observe and mimic common behaviour of your ecosystem's tools.


<br>

## 4.14. Don’t use color as the only way to convey information

Users who are color blind cannot receive information that is conveyed only through color, such as in a color status indicator. Include other visual cues, preferably text, to ensure that information is accessible.


<br>

## 4.15. Avoid using blinking font (ANSI modifier)

Testing your work:
- assistive Tech
  Familiraize yourself with screen-reader tech like VoiceOver JAWS and NVDA. If you can, get an assistive technology lab and test out your software. If you can't, improvise!
- Content and naming
  Am I saying this in the simplest, most direct way? Do my error messages make sense and give the users a path forward?
- Yes, you too
  Test with differently-abled users!
- Automation
  Add automated a11y linter to your CI pipeline. Design automated tests with a11y in mind.



[https://speakerdeck.com/raqueldesigns/everyone-should-be-able-to-use-your-software-accessibility-for-balanced-teams?slide=42](https://speakerdeck.com/raqueldesigns/everyone-should-be-able-to-use-your-software-accessibility-for-balanced-teams?slide=42)


<br>

## 4.16. Give users enough time to complete an action


<br>

## 4.17. Explain what will happen after completing a service / command


<br>

## 4.18. Make important information clear


<br>

## 4.19. Let users check and change their answers before they submit them

Accessibility is important - make your applications accessible because its the right thing to do, but if you don't want to do it because it's the right thing to do, do it because it's the legally required thing - https://section508.gov/


<br><br><br>

# 5. DOCUMENTATION

At minimum your tool should be able to inform the user on how to use it effectively. This is most commonly done with `--help` flag.


<br>

## 5.1. Provide `--version` flag with understandable versioning scheme

If you're not sure which one to use go with semver, because it's the most popular one.[https://semver.org/](https://semver.org/)


<br>

## 5.2. Document every file that will impact the behaviour

If your app uses something from `/etc` document that. If your app reads `~/.config/<my-app>/config.ini` document that as well. Every file that impacts the CLI has to be documented, or at least mentioned.


<br>

## 5.3. Provide easy way of updating the tool by itself

`$ tool update` should be built in, tested and working out of the box. This might be as simple as downloading appropriate binary and replacing the original one, that's simple enough to support it without any dependencies. If your tool needs more things to update itself, consider simplifing it.


<br>

## 5.4. `--help` is not `man`

If possible, support `man` pages as well as your built-in documentation. Many CLI tools have extensive documentation built in under `help` subcommand which is useful, but also limited in some regards. The same docs can be easily regenerated to different format using tools like `ronn`.

[https://rtomayko.github.io/ronn/](https://rtomayko.github.io/ronn/)

[https://man7.org/linux/man-pages/man7/man-pages.7.html](https://man7.org/linux/man-pages/man7/man-pages.7.html)


<br>

## 5.5. Try to follow man pages when displaying help

```
NAMESYNOPSIS
              CONFIGURATION    [Normally only in Section 4]
DESCRIPTION
              OPTIONS          [Normally only in Sections 1, 8]
              EXIT STATUS      [Normally only in Sections 1, 8]
              RETURN VALUE     [Normally only in Sections 2, 3]
              ERRORS           [Typically only in Sections 2, 3]
              ENVIRONMENT
              FILES
              VERSIONS         [Normally only in Sections 2, 3]
              ATTRIBUTES       [Normally only in Sections 2, 3]
              CONFORMING TO
              NOTES
              BUGS

              EXAMPLES
              AUTHORS          [Discouraged]
              REPORTING BUGS   [Not used in man-pages]
              COPYRIGHT        [Not used in man-pages]
SEE ALSO
```


<br><br><br>

# 6. CONFIGURATION


<br>

## 6.1. Support XDG specification

[https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html)

[https://github.com/aws/aws-sdk/issues/30](https://github.com/aws/aws-sdk/issues/30)

this one drives me crazy, one of the most requested changes and it just hangs there for over three years now.


<br>

## 6.2. Don't invent your own config syntax for configuration

Please, don't. Every config format, even the most basic one like INI is better than your own.


<br>

## 6.3. Support multiple output formats if they are required by the users

It's better to do it correctly on your side than to let the users pipe your json to `yq` just because they need that output format badly for some reason.

You are creating this tool for humans, make it human friendly :)


<br>

## 6.4. If behaviour can be altered with env variables then it should be possible to do the same with flags

If your program uses `XDG_CONFIG_DIR` to place it's config, then you should also support `—config-dir` for doing the same thing.


<br><br><br>

# 7. STATEFULNESS

Keep it low. CLI is a simple state machine, and it should stay simple.


<br>

## 7.1. If your app has to store some state, store it in one, clearly indicated file.

Don't set environment variables to store anything, it just won't work.
The best way is to not have a state, second best is to keep it in one (and only one) file.

Good example: terraform

Bad example: ???


<br><br><br>

# 8. SCRIPTING

Always assume that your tool will be wrapped with some script. Folks from `apt-get` never expected that, but it's the reality.


<br>

## 8.1 Support JSON output

It's the most universal and by far the most popular way of outputing machine-readable data.

Bonus points: support filtering / limiting the output as well.


<br>

## 8.2. Be aware of signals

Expect that your tool will receive signals. Your tool should react most commonly to that.

(how, exaclty?)


<br>

## 8.3. Stderr is for errors, stdout is for output

Stick to it.


<br>

## 8.4. Expect input from stdin, pipe, unix socket, redirection from files etc

In other words: make it input-agnostic. Don't assume anything about the source input because it probably won't be true.


<br>

## 8.5. STDOUT is your API. You have to expect people wrapping it like an api

If you don't provide a machine-readable output (—json), then stdout will be used as a machine-readable output. It will be parsed, processed with various tools and will be relied upon to do the work. This matters *a lot* when your tool is big enough.

—json (or any other switch for machine readable output) is your safety valve, if you don't have that then you have to be way more concius on how output is used.


<br><br><br>

# 9. DEVELOPMENT


<br>

## 9.1. Test your code, heavly. Test your CLI commands heavly

One tip is to don't hook up annonymous function to your CLI framework but instead keep them in separate module and then hook them to framework's API explicitly in some separate module.
That way you will be able to easily test your command runners (functions) without the burden of dealing with CLI framework.


<br>

## 9.2. Connect your command handlers (functions) to the CLI framework boringly

It has to be simple. boring, moundaine and uninteresting code, because it is uninteresting code. The interesting part is in the command handler function, that's where magic happens! Keep the complexity out of the handler<>CLI framework bridge.

<br>

---

<br><br><br>

# 10. RESOURCES USED


<br>

## 10.1. Links

<br>

- [http://mds.is/a11y/](http://mds.is/a11y/)
- [https://www.w3.org/TR/WCAG21/](https://www.w3.org/TR/WCAG21/)
- [https://clig.dev/](https://clig.dev/#further-reading)
- [https://semver.org/](https://semver.org/)
- [https://medium.com/@jdxcode/12-factor-cli-apps-dd3c227a0e46](https://medium.com/@jdxcode/12-factor-cli-apps-dd3c227a0e46)
- [https://github.com/lirantal/nodejs-cli-apps-best-practices](https://github.com/lirantal/nodejs-cli-apps-best-practices)
- [https://www.shopify.com/partners/blog/content-strategy](https://www.shopify.com/partners/blog/content-strategy)
- [https://blog.developer.atlassian.com/10-design-principles-for-delightful-clis/](https://blog.developer.atlassian.com/10-design-principles-for-delightful-clis/)
- [https://devcenter.heroku.com/articles/cli-style-guide](https://devcenter.heroku.com/articles/cli-style-guide)
- [https://zapier.com/engineering/how-to-cli/](https://zapier.com/engineering/how-to-cli/)
- [https://eng.localytics.com/exploring-cli-best-practices/](https://eng.localytics.com/exploring-cli-best-practices/)
- [http://codyaray.com/2020/07/cli-design-best-practices](http://codyaray.com/2020/07/cli-design-best-practices)
- [https://dev.to/nickparsons/crafting-a-command-line-experience-that-developers-love-4451](https://dev.to/nickparsons/crafting-a-command-line-experience-that-developers-love-4451)
- [https://devcenter.heroku.com/articles/cli-style-guide](https://devcenter.heroku.com/articles/cli-style-guide)
- [https://relay.sh/blog/command-line-ux-in-2020/](https://relay.sh/blog/command-line-ux-in-2020/)
- [http://catb.org/esr/writings/taoup/html/ch01s06.html#id2877610](http://catb.org/esr/writings/taoup/html/ch01s06.html#id2877610)
- [https://scis.uohyd.ac.in/~apcs/itw/UNIXProgrammingEnvironment.pdf](https://scis.uohyd.ac.in/~apcs/itw/UNIXProgrammingEnvironment.pdf)
- [https://bitbucket.org/vasudevram/s](https://bitbucket.org/vasudevram/selpg/src/master/DevelopingALinuxCommandLineUtility.pdf)
- [https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap12.html](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap12.html)[elpg/src/master/DevelopingALinuxCommandLineUtility.pdf](https://bitbucket.org/vasudevram/selpg/src/master/DevelopingALinuxCommandLineUtility.pdf)
- [https://man7.org/linux/man-pages/man7/man-pages.7.html](https://man7.org/linux/man-pages/man7/man-pages.7.html)
- [https://www.gnu.org/prep/standards/html_node/Command_002dLine-Interfaces.html](https://www.gnu.org/prep/standards/html_node/Command_002dLine-Interfaces.html)
- [https://www.youtube.com/watch?v=eMz0vni6PAw](https://www.youtube.com/watch?v=eMz0vni6PAw)
- [https://accessibility.psu.edu/legibility/contrast/](https://accessibility.psu.edu/legibility/contrast/)
- [https://accessibility.psu.edu/color/brightcolors/](https://accessibility.psu.edu/color/brightcolors/)
- [https://accessibility.voxmedia.com/](https://accessibility.voxmedia.com/)
- [https://accessibility.18f.gov/](https://accessibility.18f.gov/)
- [https://accessibility-handbook.mybluemix.net/design/a11y-handbook/](https://accessibility-handbook.mybluemix.net/design/a11y-handbook/)
- [https://ukhomeoffice.github.io/accessibility-posters/posters/accessibility-posters.pdf](https://ukhomeoffice.github.io/accessibility-posters/posters/accessibility-posters.pdf)
- [https://www.southampton.ac.uk/~km2/teaching/hci/lec19.htm](https://www.southampton.ac.uk/~km2/teaching/hci/lec19.htm)
- [https://speakerdeck.com/crystalprestonwatson/its-always-sunny-in-mobile-accessibility?slide=46](https://speakerdeck.com/crystalprestonwatson/its-always-sunny-in-mobile-accessibility?slide=46)
- [https://click.palletsprojects.com/en/7.x/](https://click.palletsprojects.com/en/7.x/)
- [https://rome.tools/#technical](https://rome.tools/#technical)


<br>

## 10.2. Collections

<br>

- [https://a11yresources.webflow.io/](https://a11yresources.webflow.io/)
- [https://github.com/brunopulis/awesome-a11y](https://github.com/brunopulis/awesome-a11y)
- [https://github.com/Kikobeats/awesome-cli](https://github.com/Kikobeats/awesome-cli)
- [https://github.com/alebcay/awesome-shell](https://github.com/alebcay/awesome-shell)
- [https://github.com/jlevy/the-art-of-command-line](https://github.com/jlevy/the-art-of-command-line)
- [https://stackoverflow.com/questions/28708037/recommendations-best-practices-on-custom-node-js-cli-tool-config-files-location](https://stackoverflow.com/questions/28708037/recommendations-best-practices-on-custom-node-js-cli-tool-config-files-location)
- [https://stackoverflow.com/a/28708450](https://stackoverflow.com/a/28708450)
- [https://oclif.io/docs/introduction.html](https://oclif.io/docs/introduction.html)
- [https://github.com/matuzo/HTMHell](https://github.com/matuzo/HTMHell)
- [https://github.com/palash25/best-practices-checklist](https://github.com/palash25/best-practices-checklist)


<br>

---

<br>

<small>
Thank you for reading :) You're welcome to comment this list on
hackmd or on github.
</small>


<br>
<small><details><summary><em>
my notes, please ignore for now :)
</em></summary><small></small>

- [x]  TODO: ask `@carolynvs` for more resources, correct errors?
- [ ]  TODO: find more people who care about good CLIs, ask for their help
- [x]  TODO: share this on github (`awesome-cli-practices`? Still thinking on good title, probably something starting with `awesome-` so it's easier to find it among other GH repos).
- [ ]  TODO: talk with a11y folks on their ideas for this specific environment (no resources so far...)
- [ ]  TODO: clean it up, add example of bad/good to every point.
- [ ]  TODO: add footnotes with references to source knowledge

</details></small>
<br>