# expand
Word expansion for [Oh My Fish][omf].

## Overview
This plugin provides the means of adding word-by-word expansions for other plugins, as well as for your own expansions. A word expansion is a real-time substitution of the words you type on the command line. It is a bit like `abbr`, but dynamic and extensible and a whole lot more awesome.


## Install
```fish
$ omf install expand
```


## Using it
Using expansions already provided by your installed plugins couldn't be any easier. Word expansions are triggered when you use the <kbd>TAB</kbd> key when the cursor is over a word. Before checking for regular command completions, the current word will be checked against available word expansions. If a suitable expansion is found, the current word will be replaced with the result of the expansion.

Expansions can be more than simple abbreviations; the current word can be used to introduce patterns and queries to provide dynamic expansions in real-time.


## Writing expansions
Writing your own expansions is incredibly simple. There are two parts to a word expansion: a condition, and an expander. A condition is the means of specifying on what words your expander should be run. The expander is a script or function that takes the current word and produces a replacement as its output. Below is a simple but useful example that lets you expand a substitution syntax (like `s/something/something else`) into the previous command with the substitution applied:

```fish
expand-word -p '^s/..*/.*$' -e 'echo -n "$history[1]" | sed -e (commandline -t)/g'
```

And that's really all that you need! Try running the above command, and then try pressing the tab key on a substitution:

```fish
$ git stats
git: 'stats' is not a git command. See 'git --help'.

Did you mean this?
        status
$ s/stats/status<Tab>
$ git status
```

The `expand-word` function registers a new expander by specifying the condition and the expander itself as option arguments. There are two interesting flags used in this expansion: `-p` and `-e`. First let's look at what `-e` is doing. This flag is short for the `--expander` option, which specifies the expander function or script to run. The expander uses the builtin [`commandline`][commandline] command to print the current _token_, or word, that is being expanded and uses it as a substitution expression with `sed`. The replaced command is then printed out, and the output is used by `expand` to replace the current word.

You can do pretty much anything you like in an expander. All you are required to do is print the desired replacement for the current word to standard output, and `expand` will do the rest. Now let's look closer at how we can use conditions to properly choose when our expander should run.


### Conditions
There are two different types of conditions currently available: patterns, or functions.

A pattern condition is an [extended regular expression][regex] that is matched against the current word. If the word matches, the corresponding expander is used. Let's look at how we can use a pattern to run an expander on words that start with `git`:

```fish
expand-word -p '^git.+' -e 'some_expander_function'
```

The `^git.+` pattern will match any word that starts with `git`, but not `git` itself. Keep in mind that the regular expression format is the POSIX ERE syntax, not PCRE. Check the `grep` manual for a good guide to this syntax.

The second type of condition is more interesting; a function condition is a script or function that is run each time you attempt to expand a word. The function can perform any logic you need to perform in order to correctly determine if your expander can expand the current word. If the word is of a format that your expander is expecting, your function should return `0` to tell `expand` to use your expander. If the function cannot expand the given word, a non-zero value should be returned instead, and expansion will be deferred to another expander.

For performance reasons, you may want to put some effort into optimizing a condition function; if you have a lot of expansions, _all_ of the condition functions could very well be run at one time. If your condition function is slowing down the responsiveness of word expansion, you might want to use a simpler condition.

Below is an example of using a function as an expansion condition:

```fish
function should_expand_home
  if test (commandline -t) != "home" -a "$PWD" != "$HOME"
    return 1
  end
end

expand-word -c 'should_expand_home' -e 'echo $HOME'
```

This expansion will only be used on the word "home", but only if you are not currently in your home directory. Pretty cool, eh?


## Giving up
In addition to conditions, you can also have your expanders "give up" prematurely when trying to expand a word by returning a non-zero exit code. If your expander gives up a word, it will be ignored and the next expander will be used as normal.


## License
[MIT][mit] © [Stephen Coakley][author] et [al][contributors]


[author]: https://github.com/coderstephen
[commandline]: http://fishshell.com/docs/current/commands.html#commandline
[completions]: http://fishshell.com/docs/current/tutorial.html#tut_tab_completions
[contributors]: https://github.com/oh-my-fish/plugin-fasd/graphs/contributors
[license-badge]: https://img.shields.io/badge/license-MIT-007EC7.svg?style=flat-square
[mit]: http://opensource.org/licenses/MIT
[omf]: https://www.github.com/oh-my-fish/oh-my-fish
[regex]: http://pubs.opengroup.org/onlinepubs/009696899/basedefs/xbd_chap09.html
