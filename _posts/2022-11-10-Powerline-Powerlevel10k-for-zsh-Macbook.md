---
layout: post
title: Powerline Powerlevel10k for zsh on a Macbook
tags: bash zsh git
---

I've still been using my Macbook with `bash`, changed from the system default of `zsh` purely to avoid the chance of anything breaking and needing to set up [Powerline](https://github.com/b-ryan/powerline-shell) again, I have worked with bash looking the way it does for so long I've felt an intertia to change anything.

![powerline-bash]({{ site.baseurl }}/assets/img/powerline-bash.png)

I've been thinking about the [Ventura](https://www.apple.com/au/macos/ventura/) update for my Macbook recently, so last night while trying to study I wisely spent some time instead looking into `zsh` just to accept the inevitable and get with the times.

Quickly running into [oh my zsh](https://ohmyz.sh/) I started banging through the installation guides and messing with colours and themes but not getting quite what I wanted to be completely satisfied to replace [Powerline](https://github.com/b-ryan/powerline-shell).

In the end I ended up with a decent looking shell with some great themes and plugins that I'm happy with, and I'm sharing it here in case anyone else is looking for a similar setup:

## 1. iTerm2 with Solarized and Font Awesome

Super easy this one, I don't really run into any limitations with the built in Mac terminal but iTerm2 is worth looking at either way, I found it a little easier to customise colours and fonts in this case.

### 1.1 Install iTerm2
`brew install --cask iterm2` or download from [iterm2.com](https://iterm2.com/downloads.html)

### 1.2 Install Solarized + Font Aweseome package
Download and double click the `.ttf` to install the [SourceCodePro Powerline Font Awesome](https://github.com/Falkor/dotfiles/blob/master/fonts/SourceCodePro%2BPowerline%2BAwesome%2BRegular.ttf) pack to your system.

## 2. Oh My Zsh

### 2.2  Install Oh My Zsh
`sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"` or [ohmyz.sh](https://ohmyz.sh/#install)

> Note: Installing zsh creates a few different env and aliases, including:
> * `$ZSH`, which is the directory where `oh my zsh` is installed.
> * `$ZSH_CUSTOM`, which is the directory where we will install our custom themes and plugins.

The default oh my zsh theme is clean, but immediately I missed the git status information that I had with [Powerline](https://github.com/b-ryan/powerline-shell) that shows very clearly what number and combination of unstaged & staged files, and commits I am currently working with. Also I realised I like the boldness of the background colour on the prompt text:

![oh-my-zsh]({{ site.baseurl }}/assets/img/ohmyzsh.png)

### 2.2 Custom Aliases with Oh My Zsh
Something worth nothing is that in our `~/.zshrc` file suggests putting any of our own custom aliases in `$ZSH_CUSTOM` - any `.zsh` files in this directory will be loaded in by `oh my zsh` when you open your terminal.

```shell
# Set personal aliases, overriding those provided by oh-my-zsh libs,
# plugins, and themes. Aliases can be placed here, though oh-my-zsh
# users are encouraged to define aliases within the ZSH_CUSTOM folder.
# For a full list of active aliases, run `alias`.
```

## 3. Powerlevel10k

Looking around, very quickly it seems a community favorite was the Powerlevel10k theme, which excites me because anything over 9k must be good.

### 3.1 Install Powerlevel10k
```git clone https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k```

Which installs into `/Users/adam/.oh-my-zsh/custom/themes/powerlevel10k`

To activate the theme, we need to edit our `~/.zshrc` file and change the `ZSH_THEME` variable to `ZSH_THEME="powerlevel10k/powerlevel10k"`.

` vim ~/.zshrc`

```shell
ZSH_THEME="powerlevel10k/powerlevel10k"
prompt_context(){}
```

> `prompt_context(){}` removes the username:hostname from the terminal prompt - thanks [Filipe Macêdo](https://fmacedoo.medium.com/oh-my-zsh-with-powerline-fonts-pretty-simple-as-you-deserve-fbe7f6d23723)

### 3.2 Syntax Highlighting
https://gist.github.com/kevin-smets/8568070#syntax-highlighting

### 3.3 Terminal Suggestions
https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md#oh-my-zsh


### 3.4 VS Code Terminal Integration
Open your [VS Code settings.json](https://code.visualstudio.com/docs/getstarted/settings) and search for `terminal.integrated.fontFamily` and replace that line with the following, I like to leave the previous config commented out so I can easily roll back.

```json
    //"terminal.integrated.fontFamily": "Source Code Pro for Powerline",
    "terminal.integrated.fontFamily": "'SourceCodePro+Powerline+Awesome Regular'",
```

Because I was previously using bash, I also had to change my VS Code Terminal default to `zsh` and I like the terminal icon to be something other than the default:

```json
    "terminal.integrated.profiles.osx": {
      //Other profiles
        "zsh": {
            "path": "zsh",
            "icon": "terminal-bash"
        },
      //Other profiles
    },
    "terminal.integrated.defaultProfile.osx": "zsh",
```
This is what I ended up with, very happy noting:
* the folder paths collapse i.e. `~/git/azenix/my-project/terraform/` becomes `~/g/az/my-project/terraform/` as we `cd` deeper into the folder structure
* the git status shows the number of staged and unstaged files, and the number of commits ahead we are of remote
* the length of time a command takes to run is displayed on the right, in this example `vim` was open for 7 seconds, handy for long running `terraform apply` type stuff
* the exit code/status of the last command is displayed on the right, in this case lots of `ok` (0) and a `127 err`

Love it.

![powerlevel10k]({{ site.baseurl }}/assets/img/final-powerlevel10k.png)

I hope this helps lead someone else to a better terminal experience.

Happy Clouding,

### Contact

- [LinkedIn](https://www.linkedin.com/in/adamcybersec/)<br>
- [GitHub](https://github.com/adamcybersec/)<br>
- [Email](mailto:github@adamcybersec.com)