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