---
title: Profiling zsh start-up time
date: 2023-08-04
lastmod: 2023-08-26
tags:
-   zsh
-   nvm
---

My shell (`zsh`) takes a while to start up. Not ages, but it's noticeable when
it's more than half a second or so every time I open a new terminal window. So,
I decided to try to work out what was taking so long and if I could fix it.

I came across [this blog post](https://stevenvanbael.com/profiling-zsh-startup)
from Steven Van Bael and followed the steps therein to work out what was going
on.

## step 1: do I even have a problem?

First of all, how much of a problem do I have?

```console
$ time  zsh -i -c exit
zsh -i -c exit  0.53s user 0.94s system 73% cpu 2.009 total
```

Oh dear. 2s is very not good. I'm definitely not imagining the slow
start-up time!

## step 2: the culprit

The next step was to work out why, using `zmodload zsh/zprof` and `zprof` at the
start and end my `.zshrc` file:

```txt
num  calls                time                       self            name
-----------------------------------------------------------------------------------
 1)    2        1092.82   546.41   74.47%    620.98   310.49   42.32%  nvm
 2)    1         408.07   408.07   27.81%    339.54   339.54   23.14%  nvm_ensure_version_installed
 3)    1        1406.00  1406.00   95.81%    313.18   313.18   21.34%  nvm_auto
 4)    1          68.53    68.53    4.67%     68.53    68.53    4.67%  nvm_is_version_installed
 5)    1          63.30    63.30    4.31%     51.96    51.96    3.54%  nvm_die_on_prefix
 6)    2          17.40     8.70    1.19%     17.40     8.70    1.19%  compaudit
 7)    1          11.63    11.63    0.79%     11.49    11.49    0.78%  _zsh_highlight_load_highlighters
 8)    1          10.66    10.66    0.73%     10.66    10.66    0.73%  nvm_grep
 9)    1          10.58    10.58    0.72%     10.58    10.58    0.72%  zrecompile
10)    1          26.71    26.71    1.82%      9.30     9.30    0.63%  compinit
11)    1           6.16     6.16    0.42%      6.16     6.16    0.42%  _zsh_highlight_bind_widgets
12)    7           1.58     0.23    0.11%      1.58     0.23    0.11%  add-zsh-hook
13)    1           1.45     1.45    0.10%      1.45     1.45    0.10%  colors
14)    1           1.26     1.26    0.09%      1.26     1.26    0.09%  regexp-replace
15)    3           1.01     0.34    0.07%      1.01     0.34    0.07%  is-at-least
16)    4          11.34     2.84    0.77%      0.68     0.17    0.05%  nvm_npmrc_bad_news_bears
17)    4           0.64     0.16    0.04%      0.64     0.16    0.04%  compdef
18)    1           0.47     0.47    0.03%      0.47     0.47    0.03%  nvm_has
19)    1           0.21     0.21    0.01%      0.21     0.21    0.01%  (anon) [/usr/local/share/zsh-autosuggestions/zsh-autosuggestions.zsh:458]
20)    1           0.21     0.21    0.01%      0.11     0.11    0.01%  complete
21)    2           0.09     0.04    0.01%      0.09     0.04    0.01%  is_plugin
22)    2           0.08     0.04    0.01%      0.08     0.04    0.01%  bashcompinit
23)    2           0.04     0.02    0.00%      0.04     0.02    0.00%  env_default
24)    1           0.04     0.04    0.00%      0.04     0.04    0.00%  detect-clipboard
25)    1        1406.04  1406.04   95.81%      0.03     0.03    0.00%  nvm_process_parameters
26)    1           0.01     0.01    0.00%      0.01     0.01    0.00%  nvm_is_zsh
27)    1           0.01     0.01    0.00%      0.01     0.01    0.00%  __starship_get_time
```

(I've truncated that output for clarity)

This shows that the biggest culprit is `nvm`. By a long way. I don't use `nvm`
very often, perhaps only a few times a year, so I don't need it to be available
in every shell sessions. However, I don't want to make it *un*available because
I'm very unlikely to how to get to it when I eventually need it.

This situation would benefit from *lazy loading*: don't initialise `nvm` at
start-up, but wait until just before I need it. It turns out this is pretty easy
to do with a `zsh` function called `nvm` that will be overwritten by the real
tool when it loads it:

```sh
function nvm () {
  export NVM_DIR="$HOME/.nvm"
  [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
  [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
  nvm $@
}
```

The first three lines are what I've now removed from my `.zshrc` and the last
line forwards any options on to the real `nvm`. This means the first time I use
it in a session it will be 2s slower than usual, but since that' rare it's a
cost I'll happily pay to speed up start-up every other day.

## step 3: did it work?

```console
$ time zsh -i -c exit
zsh -i -c exit  0.14s user 0.17s system 71% cpu 0.432 total
```

Success! 400ms isn't the fastest, but it's quick enough that it isn't bothering
me so a good point to stop for now.

---

**Update** (2023-08-6): I forgot about `npm` and `node`!

The trick above made it easy to have `nvm` available when I need it but did not
account for `npm` and `node`, which are also missing until `nvm` is initialised.
I have fixed this by updating the block in my `.zshrc`:

```sh
function _nvm_init(){
  unalias nvm npm node

  export NVM_DIR="$XDG_CONFIG_HOME/nvm"
  . "$NVM_DIR/nvm.sh"
  . "$NVM_DIR/bash_completion"
}

alias nvm='_nvm_init; nvm'
alias npm='_nvm_init; npm'
alias node='_nvm_init; node'
```
