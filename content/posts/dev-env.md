+++
date = "2019-09-02T19:00:00+00:00"
draft = true
showDate = true
tags = []
title = "Dev Env"

+++
When I first heard of people using a remote server as their development environment, I might have laughed it off. It sounded like a worse experience for the sake of being able to develop on any machine. But how often are you switching machines? _I thought as I closed my work laptop and opened my personal laptop._ A lot of time passed, but eventually I was reminded of [Mosh](https://mosh.org) and it all clicked. I can have my cake (a single development machine) and eat it too (a local experience everywhere). 

Now, there's some caveats because I haven't _really_ made a single development machine, but added another. 🤦🏻‍♂️So, I can't really say yet if there's a benefit to this approach or if I've just made things even more complicated. I can say, however, that it was _fun_.

## Server Setup
_TODO: Create a Terraform template_

I chose to go with DigitalOcean because I'm familiar with them and it's dead simple to get a new server up and running. All I had to do was select a few options and it was off and running.

* Ubuntu 18.04
* Standard 2GB RAM / 50GB SSD / 1CPU
* Location near me
* Add SSH key

After I was in, there was a bit more that needed to be done to finish setup. For the most part, I followed [thoughtbot](https://thoughtbot.com/blog/remote-development-machine)'s guide here.

### Create a User
I don't want to be logging in as root all the time as that is dangerous.
```bash
adduser USER_NAME
```

### Move SSH Keys
The SSH key that was added by DigitalOcean when the droplet was created was added to the `root` account. So, we're just gonna move that over to my new user account for logging in there--`root` won't need it any more.
```bash
mkdir /home/USER_NAME/.ssh
cat ~/.ssh/authorized_keys >> /home/USER_NAME/.ssh/authorized_keys
chown -R USER_NAME:USER_NAME /home/USER_NAME/.ssh
```

### Sudo Privileges
Give myself the ability to run a command as root because I'm definitely going to need to do that sometimes.
```bash
visudo
```
This will open up the `/etc/sudoers` file in `vi`, apparently the only way to edit this file.

Add a line giving my user full sudoer privileges.
```bash
USER_NAME ALL=(ALL:ALL)
```

Now, I can close my connection as `root` and ssh in as the user I setup.

### Extra packages
Let's update everything first
```bash
sudo apt update
sudo apt upgrade
sudo apt-get install build-essential curl file git # Some basic build tools for later
```

#### Mosh
So [Mosh](https://mosh.org) is this pretty neat alternative to SSH. Mosh gives me a durable connection that survives switching WiFi, poor networks, etc. Also, it gives me 'optimistic updates' as I interact with the session. It doesn't wait to hear a response back from the server so it will update my local terminal with 0 latency.

```bash
sudo apt install mosh
# You'll also need to install mosh on the machine you're mosh-ing from
```

I can now use `mosh` as if it were `ssh`.

```bash
mosh user@server
```

### Networking
_TODO: Figure out how to set this up to only work from a private network via VPN._

This dev machine is on the public internet 😱unlike my laptop which is usually safely hidden behind a router. So I've got to setup some firewalls to prevent anyone getting in who shouldn't.

At the moment, I haven't figured out how I want to expose development ports publicly for temporary sharing of work, so I'm just locking everything down.

```bash
sudo apt install ufw

# No requests are allowed from the outside world, a whitelist approach
sudo ufw default deny incoming

# All outgoing requests are allowed, though I could probably get by with just ssh and port 80
sudo ufw default allow outgoing

# Allow incoming ssh requests from anywhere
sudo ufw allow ssh

# Allow incoming udp for the mosh port range
sudo ufw allow 60000:61000/udp

# Turn on the firewall
sudo ufw enable

# Verify that the firewall was setup correctly.
sudo ufw status verbose
```

More guidance is available on [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server).

## Development Environment
Now that the server is there and running and secure(?), we can get to work. Kidding! Time to fiddle with tools.

### Linuxbrew
I had no idea this was a thing before I started this project!  It doesn't have everything, but [linuxbrew](https://docs.brew.sh/Homebrew-on-Linux) has much more than the `apt` repo that it really made the transition from macOS relatively painless. Follow their instructions to get setup.

I did not follow their instructions for adding linuxbrew to my path because it was causing problems with my `$PATH` variable. I realized the way I had my `.profile` setup, each time I would source the file as I was developing it, it would grow. When I looked later after some time of developing, I noticed that the linuxbrew paths were in there over 80 times 😬. To get around this, I removed their integration and hardcoded the paths they were attempting to add. Check it out in my [dotfiles](https://github.com/benortiz/dotfiles).

Now that that's out of the way, let's try installing some packages.
```bash
brew install hub bat diff-so-fancy
```

Neat!

### Copy/Paste
Oh boy. Now this was a fun one.

Copy/Paste. It sounds so simple. but oh dear god no. Layer on:

1. My local MacOS clipboard
1. The tmux buffer running on the server
1. The vim registers running inside of tmux on the server
1. A 'local' clipboard for the server??

It gets very challenging to find a way to communicate between all these layers, but I think I found a mostly reasonable compromise. 

Rather than try to get every clipboard in sync, I decided on a strategy to add specific commands at each layer that takes its clipboard and pushes it somewhere else. For example, instead of making `y` yank to my MacOS clipboard directly, I have a command that can take the last thing yank'd and push it to my MacOS clipboard.

Okay, let's set this up.

```bash
# On macOS
brew install clipper
```
[Clipper](https://github.com/wincent/clipper) is this very nice tool that lets you, among other things, add to the system clipboard over tcp. To work, `clipper` needs to be running locally, so I start it with `brew services start clipper`. `clipper` is now running on port 8377, but I still need to expose it to my server so the server can access it. 

This next part is easily [the part that I'm least happy with](https://github.com/wincent/clipper#configuring-mosh). To expose the port, I'm going to open another ssh connection because, as far as I can tell, `mosh` doesn't support reverse port forwarding.

```bash
# This maps localhost:8377 on the server to localhost:8377 on my mac.
ssh -R localhost:8377:localhost:8377 user@host
```

To make this a little easier:
```bash
# From clipper documentation
# In ~/.ssh/config, add a host definition as a shortcut
Host sandbox-clipper
  ControlMaster no
  ControlPath none
  ExitOnForwardFailure yes
  Hostname sandbox.example.com
  RemoteForward 8377 localhost:8377
  
# I can run this now _in addition_ to moshing in to get clipboard sync.
ssh -N -f sandbox-clipper

# And to make it a little easier, an alias for that.
alias clip-sandbox='ssh -N -f sandbox-clipper'
```

I'm not very happy to be in a situation where my connection to the server is durable, but my clipboard sync is very fragile on the network.

To use clipper on the server, its very similar to `pbcopy` if you're familiar with the mac tool.

```bash
echo 'something' | nc -N localhost 8377
```

Bam! It's on my local clipboard.

To make this easier to use, I've added the following convenience functions.

```bash
# ~/.bash_aliases
alias clip="nc -N localhost 8377"

# ~/.vimrc
# Send register to Clipper clipboard
nnoremap <silent> <leader>Y :call system('nc -N localhost 8377', @0)<CR>

# ~/.tmux.conf
# Send buffer to Clipper clipboard
bind-key -T copy-mode-vi Enter send-keys -X copy-pipe-and-cancel "nc -N localhost 8377"
```

Okay, this is great. I now have a way of getting any buffer anywhere to my local clipboard. But what was that bit about earlier? A local clipboard for the server?

So I wanted another clipboard just to make this all more complicated. I didn't want to install and configure X on this headless server just to get access to its clipboard `xclip`. But I want a clipboard that stays on the machine, in the case I want to get something out of ... I dunno, `vim`? and into maybe ... `mutt`?

Whatever, it was a fun dumb tool to write. It's two 'scripts', `ctcopy` and `ctpaste`. All they do is take input from the stdin, write to a tmp file, then read from the tmp file, and write to stdout. 

Here's their source:

```bash
export CT_CLIPBOARD=/tmp/clipboard
# ~/.local/bin/ctcopy
buf=$(command cat "$@")
echo "$buf" > $CT_CLIPBOARD;

# ~/.local/bin/ctpaste
touch "${CT_CLIPBOARD}"
command cat $CT_CLIPBOARD
```

Please clap.

Some helper functions:
```bash
# ~/.vimrc
# Send register to local clipboard
nnoremap <silent> <leader>y :call system('ctcopy', getreg('"'))<CR>
# Set register from local clipboard
nnoremap <silent> <leader>p :call setreg('"', system('ctpaste'))<CR>

# ~/.tmux.conf
# Send buffer to local clipboard
bind-key -T copy-mode-vi 'y' send-keys -X copy-pipe-and-cancel "ctcopy"
# Set buffer from local clipboard
bind C-p run "ctpaste | tmux load-buffer - ;"
```

That's about all I can take of copy/paste for now.