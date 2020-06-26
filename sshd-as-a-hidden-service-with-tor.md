## SSHD as a Hidden Service with Tor on a RaspiBolt

[![tippin.me](https://badgen.net/badge/%E2%9A%A1%EF%B8%8Ftippin.me/@kristapsk/F0918E)](https://tippin.me/@kristapsk)

This is mainly based on a [Running SSH on a Raspberry Pi as a Hidden Service with Tor](https://rusnak.io/running-ssh-on-a-raspberry-pi-as-a-hidden-service-with-tor/) guide by Pavol Rusnak.

### Introduction

Very often your RaspiBolt will be behind firewall without real IP address or even with external IP address that changes over time. That means it will be hard to remotely connect to your node from outside. This can be handled using NAT port forwarding in router and in case of not having fixed IP address from ISP using some [Dynamic DNS](https://en.wikipedia.org/wiki/Dynamic_DNS) service. But another way is to set up an `.onion` Tor Hidden Service for SSH daemon on your RaspiBolt.

As a nice bonus with this approach you get extra anonimity.

### Preparations

This guide assumes you have Tor installed, configured and running on your RaspiBolt. That's a default for RaspiBolt 2, but was optional for RaspiBolt 1. Follow [RaspiBolt Tor guide](https://stadicus.github.io/RaspiBolt/raspibolt_69_tor.html) on how to install Tor on your RaspiBolt 1.

### Configure Hidden Service

All the steps below must be done from the "admin" user.

* Edit Tor configuration file (`sudo nano -w /etc/tor/torrc`) and add following lines to it:
```
HiddenServiceDir /var/lib/tor/sshd/
HiddenServicePort 22 127.0.0.1:22
```

* Create hidden service directory and set right permissions to it:
```
$ sudo mkdir /var/lib/tor/sshd
$ sudo chown debian-tor:debian-tor /var/lib/tor/sshd
$ sudo chmod 700 /var/lib/tor/sshd
```

* Restart Tor service:
```
$ sudo systemctl restart tor
```

* Check that everything went ok and get the hidden service hostname to connect to:
```
$ sudo cat /var/lib/tor/sshd/hostname
5rraorbx5dd3cuutxcue36cp4oschvlmltzzelzlr7yokf2m77h5vgyd.onion
```

The `5rraorbx5dd3cuutxcue36cp4oschvlmltzzelzlr7yokf2m77h5vgyd.onion` in this example is the hostname you will need to connect to.

### Connecting to your RaspiBolt via Tor from Linux (should work on BSD / macOS too)

You must have Tor running and netcat (`nc` command) installed on your machine.

* Create or modify `~/.ssh/config` (`mkdir -p ~/.ssh; nano -w ~/.ssh/config`) and add following lines to it:
```
Host *.onion
    ProxyCommand nc -xlocalhost:9050 -X5 %h %p
```

* Now you should be able to connect to your RaspiBolt via SSH hidden service:
```
$ ssh admin@5rraorbx5dd3cuutxcue36cp4oschvlmltzzelzlr7yokf2m77h5vgyd.onion
```

#### Adding alias to SSH configuration

Remembering .onion hidden service name is hard, you can simplify connecting by adding alias to your SSH configuration.

* Add following lines to the `~/.ssh/config` file (`nano -w ~/.ssh/config`):
```
Host raspibolt
    HostName 5rraorbx5dd3cuutxcue36cp4oschvlmltzzelzlr7yokf2m77h5vgyd.onion
    ProxyCommand nc -xlocalhost:9050 -X5 %h %p
```

* Now you should be able to connect to your RaspiBolt with shorter command:
```
$ ssh admin@raspibolt
```

### Connecting to your RaspiBolt via Tor from Windows

TODO

