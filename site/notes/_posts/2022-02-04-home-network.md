---
title: Home network
noindex: true
---

# Home network

## Some old notes about home VPN (not currently being used)

DNS settings don’t seem to be taking effect on iPhone VPN: https://superuser.com/questions/637579/setting-dns-servers-using-openvpn-client-config-file

https://openvpn.net/cloud-docs/changing-dns-servers/ - is this because there’ sa different network, the 10.8.0.6 one?

“DNS servers '10.8.0.1 8.8.8.8' will be used for DNS queries when the VPN is active” - in tunnelblick log

but then how do you tell the DNS server to return something in that range?

http://nixpanic.net/2013/03/use-dnsmasq-for-separating-dns-queries/

OK, it’s just a matter of sorting out the client config to use the same DHCP config as the router

and then allow-listing other networks in `/etc/init.d/dnsmasq`

and now i can connect from phone

### Problem with SSH

I noticed that SSH-ing into the pi, when on the VPN from my work laptop, seemed to break all readline-y type apps like vim / paged less, etc. I will see if I can reproduce.

Couldn't reproduce on home network with either laptop.

Same happened when ssh-ing imto home laptop. Looks like it's an MTU thing (whatever that is):

https://www.sonassi.com/help/troubleshooting/setting-correct-mtu-for-openvpn

Fixed by adding `mssfix 1430` to the client config file. Mystery.

## Internet in Brazil

- [Getting out of ISP’s CGNAT](https://www.reddit.com/r/InternetBrasil/comments/kvzetr/amigos_hoje_em_dia_ano_2021_é_poss%C3%ADvel_ter_a/)
- If I can get IPv6 working then that removes some of these issues
- Static IPs are usually for CNPJs

Update, 16 May 2022. I believe, with Claro, I am on a CGNAT. e.g. my access point believes its Internet IP address to be 192.168.0.10 (which is a local address), but it's actually 189.100.71.62

According to the cable modem configuration page, I have IPv6:

```
    IPv4 100.79.209.179
    IPv6 NA 2804:14c:10a::10d4/64
    IPv6 PD 2804:14c:10a:90cd:8601:12ff:febd:6b32/64
```

(No idea what this means)

? https://en.wikipedia.org/wiki/Prefix_delegation

Taking the IPv6 of something on Mac - http://v6decode.com/#address=fe80%3A%3Aaede%3A48ff%3Afe00%3A1122 shows it's a _link-local unicast_

I turned on IPv6 Internet on the access point, hasn't seemed to acquire an IP addess.

I also accidentally turned on, I think, an IPv6 DHCP server on the Claro modem. Erm.

I wonder what happens if I change from router mode to bridge mode in the router settings. I don't fully understand the difference.

I changed to bridge - who knows what will happen. OK, that seems to have disabled the DHCPv6 settings in the modem, and the access point now has an IPv6 address! (acquired by "Internet Connection Type: Dynamic IP (SLAAC/DHCPv6)")

And, erm, my computer now has two IPv6 addresses, with prefixes?!

And what does this mean in terms of people outside the network being able to see stuff inside it? Do I need a firewall or something?

https://blogs.infoblox.com/ipv6-coe/you-thought-there-was-no-nat-for-ipv6-but-nat-still-exists/ this addresses security myths

https://test-ipv6.com/ - it all works now!

Why has resolving optiplex.internal.forooghian.com broken again? Ah, because for some reason it's now trying to use IPv6 DNS.

Changed them to Google - first thing I don't understand is how to update the IPv6 DNS servers macOS uses? refreshing DHCP lease doesn't seem to do it. I had to turn Wi-Fi off and on.

Tried setting router's DNS as IPv4-mapped IPv6 address, but it just said invalid format. Ditto for trying to set a local address.

IPv6 uses Neighbor Discovery Protocol in place of IPv4's ARP.

https://datatracker.ietf.org/doc/html/rfc8106 what are the options for an IPv6 server to get DNS configuration?

OptiPlex doesn't appear in access point's list of clients for setting up firewall rules.

SLAAC does not provide DNS and Domain name information

> If you do not have any requirement to restrict the hosts that can attach to the network using IPv6, then SLAAC, combined with DNS advertisement in the RA, and possibly with DDNS (if needed), would be the right choice.

https://en.wikipedia.org/wiki/Radvd

> The Router Advertisement Daemon (radvd) is an open-source software product that implements link-local advertisements of IPv6 router addresses and IPv6 routing prefixes using the Neighbor Discovery Protocol (NDP) as specified in RFC 2461

> Radvd also supports the recursive DNS server (RDNSS) and DNS search list (DNSSL) options for NDP published in RFC 6106.[4]

https://linux.die.net/man/8/radvdump

I wish we could just disable the IPv6 DNS configuration for now but I don't see a way to do that with this router. Of the options it offers for IPv6 LAN:

- ND Proxy
- DHCPv6
- SLAAC+Stateless DHCP
- SLAAC+RDNSS

… I'm pretty sure that all except for ND Proxy will supply DNS server information. And I have no idea what ND Proxy is. Does it let me run my own Router Advertisement server? But then how would I know when the gateway's IP address changes to tell my clients?

I wonder if it needs to be something like modem -> some machine of mine with firewall etc running -> access point

(But is there any way to totally disable the IPv6 firewall on the access point? People complaining about this: https://community.tp-link.com/en/home/forum/topic/220864?page=1 and suggestions of how to modify your firmware to allow it)

Need to understand better the various types of IPv6 local / private addresses. (e.g. ULA Unique Local Addresses)

I guess instead of having a local DNS server I could also just dyndns everything, but that doesn't seem right.

`https://en.wikipedia.org/wiki/Unique_local_address`

AAAA record is like an A record but for IPv6

I believe that dnsmasq is also a DHCPv6 server

I wonder if I can completely just turn this router into an access point

For now I've turned off IPv6 on the router again, so that at least DNS works again

NordVPN claims that most VPN providers don't support IPv6 (including them)

To get a public IPv4 address there would also be the option of connecting to a VPN (one that offers a non-NAT-ed one e.g. https://www.azirevpn.com/) from inside the network. You can also buy a dedicated IP from NordVPN for 70 USD / year.

Perhaps we could run OpenWRT from a VM for now, as an easy way to play around?

## Using Docker for home server

Seems like using docker-compose is what most people on Reddit are doing

- Watchtower for automatic updates

## Setting up OptiPlex

let's consider all of the preceding a first prototype

- Make sure it doesn't sleep
  - for now, did that through GUI settings
  - also paired bluetooth keyboard and mouse through GUI

I've set up static IP for OptiPlex in router

So, SSH doesn't work out of the box

- https://ubuntu.com/server/docs/service-openssh
  - sudo apt install openssh-server (starts automatically then)

I don't know why the computer turns itself off a few mins after startup before I login at the GUI

Copied key with `ssh-copy-id`

Followed these instructions to install Plex: https://support.plex.tv/articles/200288586-installation/ (seems like I need to access it by IP address instead of optiplex.internal.forooghian.com)

After install, it says:

```
PlexMediaServer install: WARNING: The Intel GMM library, required for Intel Compute Runtime support, is missing.
PlexMediaServer install:          Please install package:  'intel-gmmlib' from https://github.com/intel/compute-runtime/releases
PlexMediaServer install:
PlexMediaServer install: WARNING: The Intel IGC Core, required for Intel Compute Runtime support, is missing.
PlexMediaServer install:          Please install package:  'intel-igc-core' from https://github.com/intel/compute-runtime/releases
PlexMediaServer install:
PlexMediaServer install: WARNING: The Intel IGC OpenCL library, required for Intel Compute Runtime support, is missing.
PlexMediaServer install:          Please install package:  'intel-igc-opencl' from https://github.com/intel/compute-runtime/releases
PlexMediaServer install:
PlexMediaServer install: WARNING: The Intel OpenCL library, required for Intel Compute Runtime support, is missing.
PlexMediaServer install:          Please install package:  'intel-opencl' from https://github.com/intel/compute-runtime/releases
PlexMediaServer install: Intel Compute Runtime packages are available from:  https://github.com/intel/compute-runtime/releases
PlexMediaServer install: Please be certain to install them in the listed order.
```

I don't know if I need those for anything - QuickSync? Well, let's leave it for now.

Turned on in web GUI "Scan my library automatically:

"Generate video preview thumbnails" as a scheduled task and when media is added. Ditto chapter thumbnails and analyzing audio tracks for loudness

The Plex library directories are `/var/lib/plexmediaserver`

Did `sudo chmod 777 .` in that directory so that I could copy to it over SSH (pretty sure that's not the best way to do it)

Why is scp-ing movies to the server suuuuuper slow? Like 24MB/s

How to set up SSL access to the server?

Installed transmission just to have something for torrenting: https://linuxconfig.org/how-to-set-up-transmission-daemon-on-a-raspberry-pi-and-control-it-via-web-interface (TODO add this file to my config repo)

It's available on http://192.168.1.151:9091/

Let's try setting up the DNS server?

Let's also set up VPN again. For now, manually updated Namecheap's home.forooghian.com to current IP. Will set up something dynamic later.

OK, for whatever reason, that's failing to connect. I wonder if it's related to whatever I'd seen before about struggling to get an IP address for your home in Brazil? Will have to investigate.

Hmm, computer still seems to be going to sleep after some inactivity, after which point I can't SSH into it. Mashing on the keyboard means I can again.

(BIOS: hold F2 on startup)

Tried turning on "Block Sleep" in BIOS.

Also turned on automatially powering back on after power loss.

Erm, it still seems to become unresponsive after a while, but now the power light remains solid. what's that?

Following this: https://www.tecmint.com/disable-suspend-and-hibernation-in-linux/

```
lawrence@lawrence-OptiPlex-3080:~$ systemctl status sleep.target suspend.target hibernate.target hybrid-sleep.target
● sleep.target - Sleep
     Loaded: loaded (/lib/systemd/system/sleep.target; static; vendor preset: enabled)
     Active: inactive (dead) since Fri 2022-05-13 19:16:15 -03; 2min 33s ago
       Docs: man:systemd.special(7)

May 13 19:11:50 lawrence-OptiPlex-3080 systemd[1]: Reached target Sleep.
May 13 19:16:15 lawrence-OptiPlex-3080 systemd[1]: Stopped target Sleep.

● suspend.target - Suspend
     Loaded: loaded (/lib/systemd/system/suspend.target; static; vendor preset: enabled)
     Active: inactive (dead)
       Docs: man:systemd.special(7)

May 13 19:16:15 lawrence-OptiPlex-3080 systemd[1]: Reached target Suspend.
May 13 19:16:15 lawrence-OptiPlex-3080 systemd[1]: Stopped target Suspend.

● hibernate.target - Hibernate
     Loaded: loaded (/lib/systemd/system/hibernate.target; static; vendor preset: enabled)
     Active: inactive (dead)
       Docs: man:systemd.special(7)

● hybrid-sleep.target - Hybrid Suspend+Hibernate
     Loaded: loaded (/lib/systemd/system/hybrid-sleep.target; static; vendor preset: enabled)
     Active: inactive (dead)
       Docs: man:systemd.special(7)
```

Let's try `sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target` - that seems to have done the trick. Undid the BIOS change.

I've installed the Samsung SSD 840 EVO (500GB) that was in my now-blown-up caddy. It just seems to contain iPhone backups. Happy to get rid of that. Gonna reformat to ext4.

I want to eventually figure out how to full-disk encrypt this (LUKS?), though. But for now will live with it as is. Mounting on `/mnt/storage`. Used https://www.linuxbabe.com/desktop-linux/how-to-automount-file-systems-on-linux to add to `etc/fstab` to auto-mount.

Let's move our Plex directory there.

Let’s set up Dnsmasq for running a DNS server.

```
dnsmasq: failed to create listening socket for port 53: Address already in use
lawrence@lawrence-OptiPlex-3080:~$ sudo ss -lp "sport = :domain"
Netid    State      Recv-Q     Send-Q         Local Address:Port           Peer Address:Port    Process
udp      UNCONN     0          0              127.0.0.53%lo:domain              0.0.0.0:*        users:(("systemd-resolve",pid=1041,fd=12))
tcp      LISTEN     0          4096           127.0.0.53%lo:domain              0.0.0.0:*        users:(("systemd-resolve",pid=1041,fd=13))
```

what on earth is systemd-resolve?

https://unix.stackexchange.com/questions/304050/how-to-avoid-conflicts-between-dnsmasq-and-systemd-resolved

let's disable the stub listener:

https://unix.stackexchange.com/a/358485

and then use localhost as primary nameserver:

https://unix.stackexchange.com/a/304078

Erm, now DNS is broken on that machine, which is nice

Added to `/etc/dnsmasq.conf` (for Google DNS):

```
server=8.8.8.8
server=8.8.4.4
```

OK, well, I didn't manage to change the /etc/resolv.conf (it got reverted). But, erm, something about the Optiplex being plugged in makes the whole network messed up (Internet light on access point goes off, can't access any sites over Wi-Fi, etc). And then unplugging Optiplex from Ehternet fixes things. NO IDEA what happened.

That's deeply confusing - how can plugging in a device bring the network down? As far as I can see, nothing was configured to use the Optiplex for anything...?

- 192.168.1.1 is the access point
- 192.168.0.1 is the Claro modem

I can still access 192.168.1.1, I can no longer access 192.168.0.1 (ping times out)

I can't ping my Optiplex, I can ping the Apple TV

Now the Optiplex has vanished from clients list on router

Plugged it into TV, now Ubuntu isn't even getting as far as a GUI on startup!

Press ESC on startup to get to GRUB and then boot into recovery mode

Bottom of screen is cut off in recovery mode, that's nice

Ah, /etc/resolv.conf is currently a symlink to ../run/systemd/resolve/stub-resolv.conf. We need to I think just create our own file isntead

Ctrl-alt-F1 switches to first tty where you can then log in

This is so wildly confusing

Well, can I restore to the factory settings of the OS? I know it's nuclear but not sure what else to do, not sure how I broke it. (There's a recovery option on the GRUB menu)

I've also reserved the Wi-Fi interface an address on the router:

10-51-07-8C-DA-39 / 192.168.1.133

Now, to turn off Wi-Fi, use [NetworkManager](https://wiki.archlinux.org/title/NetworkManager) apparently:

`nmcli radio wifi off` / `on`

OK, the OptiPlex is at

```
interface-name=optiplex.internal.forooghian.com,enp2s0
interface-name=optiplex-wifi.internal.forooghian.com,wlp3s0
```

```
cname=transmission.internal.forooghian.com,optiplex.internal.forooghian.com
```

Wow, OK, that was all working. And then I did an `apt update && apt upgrade && reboot` and the network is broken again. What the heck? I wonder if something got reverted after an update.

Hmm, this time the GUI isn't messed up, though.

Seems like `/etc/resolv.conf` has been replaced by a file (no longer a symlink)

```
# Generated by NetworkManager
nameserver 127.0.0.53
```

dnsmasq is running perfectly happily

`/etc/systemd/resolved.conf` is as before (`DNSStubListener=no`)

So it's just that resolv.conf, why has it been replaced?

I've tried changing it back, let's see if it gets replaced again. I rebooted, and it seems not.

OK, looking at `man NetworkManager.conf`, the `dns` option seems relevant. One of the options is `dnsmasq`, which is probably a good thing, but I already have a dnsmasq set up and don't want to faff with it now. So let's choose `none`, which apparently means it won't touch `resolv.conf`.

Now (possibly because not connected to Internet) dnsmasq is returning REFUSED, even when I try to get transmission.internal.forooghian.com. Possibly because of `interface-name` not having an IP address.

OK, that was due to lack of Internet. Still transmission.internal.forooghian.com giving something unexpected by the looks of it. But let's reconnect and see if things are working.

Grr, well, the whole network hasn't come down, which is nice. But for some reason I can't make DNS requests from my laptop to the OptiPlex.

Ah, perhaps dnsmasq started on Wi-Fi and then broke once I turned off Wi-Fi or something. Yup, restarting dnsmasq sorted that out.

OK, cool. I don't know how to double-check that `NetworkManager` isn't going to play those tricks again, I don't know if the only way to trigger it is a software update, which there isn't any available right now.

Eh?

```
dig transmission.internal.forooghian.com

; <<>> DiG 9.10.6 <<>> transmission.internal.forooghian.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 39168
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;transmission.internal.forooghian.com. IN A

;; AUTHORITY SECTION:
forooghian.com.		1800	IN	SOA	dns1.registrar-servers.com. hostmaster.registrar-servers.com. 1652641991 43200 3600 604800 3601

;; Query time: 23 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sun May 15 16:19:46 -03 2022
;; MSG SIZE  rcvd: 135
```

but

```
dig @192.168.1.151 transmission.internal.forooghian.com
```

returns the right thing

Erm and now the aforementioned command is working...?! OK, I've reset DNS cache on Mac and things _seem_ okay.

## Setting up Let’s Encrypt again

So, I don't want:

- to need to allow-list my IP address with Namecheap to be able to use their API
- my server to have full access to my Namecheap DNS settings

So I'm thinking about using some other DNS provider with a more permissive API, and hosting the DNS for a subdomain there (using an NS record, I think), e.g. acme.forooghian.com, and then using DNS alias mode (https://github.com/acmesh-official/acme.sh/wiki/DNS-alias-mode) to delegate validation for internal.forooghian.com to that domain.

Well, Cloudflare doesn't even let me add a subdomain

http://freedns.afraid.org seems to want you to pay else people can use subdomains of your domain?!

Gonna try https://desec.io/ - a free, security-minded, open-source, donation-sustained service. I don't know how to set up DNSSEC - Namecheap doesn't seem to accept the records that deSEC is asking me to add. But oh well, I can at least resolve subdomains of acme.forooghian.com.

https://pypi.org/project/certbot-dns-desec/

https://www.eff.org/deeplinks/2018/02/technical-deep-dive-securing-automation-acme-dns-challenge-validation is an article all about this sort of security

OK, let's try with acme.sh

I have a suspicion I just accidentally deleted lawrence's crontab. Hope there was nothing important there.

I ran

```
export DEDYN_TOKEN=<token>

acme.sh --issue --dns dns_desec --domain 'internal.forooghian.com' --domain '*.internal.forooghian.com' --challenge-alias acme.forooghian.com
```

and got a lot of `Processing, The CA is processing your order, please just wait. (26/30)` and eventually `internal.forooghian.com:Timeout`

(It's trying to use ZeroSSL, not Let’s Encrypt: https://community.letsencrypt.org/t/the-acme-sh-will-change-default-ca-to-zerossl-on-august-1st-2021/144052)

Let's try with Let's Encrypt (add `--server letsencrypt`)

add `--debug` to see more logs from acme.sh

Well, it's still failing, now don't even know why. Yay

```
[dom mai 15 20:03:22 -03 2022] Checking internal.forooghian.com for _acme-challenge.acme.forooghian.com
[dom mai 15 20:03:23 -03 2022] Domain internal.forooghian.com '_acme-challenge.acme.forooghian.com' success.
[dom mai 15 20:03:23 -03 2022] Checking internal.forooghian.com for _acme-challenge.acme.forooghian.com
[dom mai 15 20:03:23 -03 2022] Domain internal.forooghian.com '_acme-challenge.acme.forooghian.com' success.
[dom mai 15 20:03:23 -03 2022] All success, let's return
[dom mai 15 20:03:23 -03 2022] Verifying: internal.forooghian.com
[dom mai 15 20:03:24 -03 2022] Pending, The CA is processing your order, please just wait. (1/30)
[dom mai 15 20:03:28 -03 2022] internal.forooghian.com:Verify error:No TXT record found at _acme-challenge.internal.forooghian.com
```

So, `dig -t txt _acme-challenge.acme.forooghian.com` is returning the nice long thing

```
;; ANSWER SECTION:
_acme-challenge.acme.forooghian.com. 3600 IN TXT "Vj3yPAIwQ6ch_XPkAwijJHaV06Zd8xtg6TM7d9afTZM"
```

but `dig -t txt _acme-challenge.internal.forooghian.com` returns no records

`dig -t txt _acme-challenge.forooghian.com` returns the CNAME (don't know how to make it follow that)

OK, was a problem with my DNS records. Sorted it out.

Rebooted, now DNS not working? What's going on? Wonder if it's starting on wrong interface.

Hmm, after doing one lookup on the machine, it then starts to work remotely too. Errrrrm.

I wonder if related to

```
mai 15 20:38:54 lawrence-OptiPlex-3080 dnsmasq[1183]: Ignoring query from non-local network
```

Very confusing.

I added `interface=enp2s0` to dnsmasq config, _seems_ to have done something useful, who knows?

Not sure if the auto renew works, nor if my certificates are right. everything is in `~/.acme.sh/internal.forooghian.com`. there's a `.conf` file, which doesn't mention the `DEDYN_TOKEN`. does this need to be in the environment when the cron job runs?

`openssl x509 -in fullchain.cer -noout -text`

```
        Subject: CN = internal.forooghian.com
        ...
        X509v3 Subject Alternative Name:
        DNS:*.internal.forooghian.com, DNS:internal.forooghian.com
```

Also will I get warned if the cron job fails?

Let's try setting up HAProxy.

and will it then auto install to haproxy after I do it once?

```
frontend default
        bind :80

        acl ACL_transmission.internal.forooghian.com hdr(host) -i transmission.internal.forooghian.com
        use_backend be_transmission if ACL_transmission.internal.forooghian.com

backend be_transmission
        server transmission 127.0.0.1:9091
```

Tried to then deploy SSL to HAProxy:

```
acme.sh --deploy --domain internal.forooghian.com --deploy-hook haproxy
```

but failed

```
[dom mai 15 21:29:24 -03 2022] Moving new certificate into place
/home/lawrence/.acme.sh/deploy/haproxy.sh: line 163: /etc/haproxy/internal.forooghian.com.pem: Permission denied
```

I think I might want to run it as root, so that I can restart HAProxy afterwards, etc. Otherwise, will need to sort out directory permissions / sudoers file to allow me to do stuff. https://github.com/shellrent/acme.sh/wiki/sudo

used visudo to add:

```
lawrence ALL = NOPASSWD: /bin/systemctl reload haproxy
```

this lets me reload without password (https://www.cyberciti.biz/faq/linux-unix-running-sudo-command-without-a-password/)

and then

```
export DEPLOY_HAPROXY_RELOAD="sudo /bin/systemctl reload haproxy"
export DEPLOY_HAPROXY_PEM_PATH=/home/lawrence/haproxy-certs

acme.sh --deploy --domain internal.forooghian.com --deploy-hook haproxy
```

Seems to have saved that config to the acme.sh config file for the domain:

```
Le_DeployHook='haproxy,'
Le_Deploy_haproxy_pem_path='/home/lawrence/haproxy-certs'
Le_Deploy_haproxy_reload='sudo /bin/systemctl reload haproxy'
```

Then to set up HAProxy SSL termination:

https://www.haproxy.com/blog/haproxy-ssl-termination/

TODO: how to make those services (transission etc) ONLY available from the machine? so that e.g if I remove basic auth and let HAProxy handle it, I can only get to the service through HAProxy outside of the machine?

TODO: is there a risk of Transmission being able to write to random locations? like overwriting system files and stuff

Now the frontend looks like:

```
frontend internal
	bind :80
	bind :443 ssl crt /home/lawrence/haproxy-certs/internal.forooghian.com.pem

	http-request redirect scheme https code 301 unless { ssl_fc }
	http-response set-header Strict-Transport-Security "max-age=16000000; includeSubDomains; preload;"

	acl ACL_transmission.internal.forooghian.com hdr(host) -i transmission.internal.forooghian.com
	use_backend be_transmission if ACL_transmission.internal.forooghian.com
```

Ahhh, the dedyn token is stored in ~/.acme.sh/account.conf. Cool, so I think that the renewal stuff should all be working. I won't know if the cronjob fails, but that's another thing to figure out sometime in general.

## Plex

```
Update available for Plex Media Server
You are currently running version 1.26.0.5715 on the server "OptiPlex". Version 1.26.1.5798 is now available. This update will need to be installed manually after download.
```

> That said, Plex in Docker with watchtower or ouroboros will keep things up to date.

How to enable auto updates?

Apparently maybe the Snap version does

I'm confused about how to make it so that Plex can delete stuff - at the moment things seem to be created by debian-transmission and can't be deleted by plex.

## Installing Flood

```
sudo apt install nodejs npm
```

oh, that's ancient version, using NodeSource distrib (because too lazy to faff with untarring right now):

https://github.com/nodesource/distributions/blob/master/README.md#debinstall

```
curl -fsSL https://deb.nodesource.com/setup_17.x | sudo -E bash -
sudo apt-get install -y nodejs
```

(18 complains about my glibc version too old)

erm, seem to be unable to reach that

Seems to have some storage somewhere, because I configured some options (username, where to find Transmission) and they're still configured on next launch. I think it's stored in `~/.local/share/flood`.

`flood --host '0.0.0.0'`

and then you set up in the web interface (would be nice if could do with text):

- RPC URL: http://localhost:9091/transmission/rpc
- username and password currently in Bitwarden, probably don't need to be any more

OK, now in Transmission

- turned allow lists back on, bound to 127.0.0.1
- changed default downloads directory to the Plex one

Flood has a cool little button to register to handle magnet links, that's nice

Followed the [steps](<https://github.com/jesec/flood/wiki/Install-Flood-as-a-service-(systemd)>) for setting up as a systemd service (including creating a user called `download`)

Got haproxy pointing to it

Weirdly it seemed to have 100kb/s speed limits turned on by default?! I changed that to unlimited in the UI and now downloads just seem to have stopped…

Well, flicked that to some non-unlimited and back again, and seems to work.

https://trac.transmissionbt.com/wiki/MagnetLinks - there's also transmission-remote, command line app that lets you add stuff.

rTorrent + ruTorrent also seems to be another widely-recomended option if this one isn't good.

there's also desktop clients for transmision (e.g. transmission-qt, transmission-remote-gui)

how to enable SSL for transmission RPC?

playing around with Transmission Remote GUI - it seems nice

Suffers from not having access to the paths when adding files - but you can set up a local to remote path mapping, which I think means that I'd need to set up a file server

## Time Machine

Sounds like Samba (SMB protocol plus some Apple extensions you need to enable) plus Avahi (for service discovery)

https://en.wikipedia.org/wiki/Samba_(software)

https://cloudinfrastructureservices.co.uk/nfs-vs-smb/

Samba is an open-source implementation of Microsoft's SMB protocol

## How to upgrade server

Let's see what `sudo apt upgrade` does

What about Plex? OK, so I set it up with https://support.plex.tv/articles/235974187-enable-repository-updating-for-supported-linux-server-distributions/ and then I was just able to upgrade it with `sudo apt upgrade`. Nice.

What about Ubuntu? We're currently on 18.04.2 LTS and the latest in this major version is 18.04.6 LTS. But there’s now 22.04 LTS too.

There's some upgrade instructions here: https://ubuntu.com/blog/how-to-upgrade-from-ubuntu-18-04-lts-to-20-04-lts-today

Ran `sudo do-release-upgrade -d` and it said

```
Checking for a new Ubuntu release
In /etc/update-manager/release-upgrades Prompt
is set to never so upgrading is not possible.
```

So let's try to set it in that file to `lts` and run again.

It asked me about `/etc/resolved.conf`, I just kept mine. See screenshot of diff. Ditto for `/etc/haproxy/haproxy.cfg`, `/etc/dnsmasq.conf`

Erm, it did all its stuff, and then after restart it's still saying

```
Welcome to Ubuntu 18.04.2 LTS (beaver-osp1-ygritte X37) (GNU/Linux 5.4.0-125-generic x86_64)
```

Hmm, `lsb_release` does show 20 though. Hmm. It's the scripts in `/etc/update-motd.d` that set it; for some reason they're getting the older version. Dunno. Seems like it's come from `/etc/lsb-release`. Not sure why that wasn't updated. Well, just updated that file with the contents stolen from [here](https://www.cloudbooklet.com/how-to-upgrade-to-ubuntu-22-04-simple-steps/)

https://askubuntu.com/questions/1099247/wrong-lsb-release-after-release-upgrade

https://www.reddit.com/r/FindMeADistro/comments/d3atzf/what_linux_distro_for_headless_home_server/ talks about unattended-upgrades for installing upgrades automatically

## How to upgrade BIOS of server

The fan of the OptiPlex is really noisy, Internet suggests that a BIOS update might fix it.

[Looks like](https://www.youtube.com/watch?v=BHNG_ls68s0) `fwupdmgr` is the tool for doing this.

`fwupdmgr get-updates` shows a bunch of stuff. I _think_ I currently have 2.3.1.

Well, I updated it to 2.15.0. Then it seemed to not connect over SSH on reboot. I plugged it in to the TV and it started up, maybe just was doing a filesystem check or something?

## Getting the fan speed

I wonder how to find the fan speed. I tried installing `lm-sensors`, then did `sudo sensors-detect` and then `sensors`, but it didn't show anything about fans:

```
coretemp-isa-0000
Adapter: ISA adapter
Package id 0:  +32.0°C  (high = +80.0°C, crit = +100.0°C)
Core 0:        +28.0°C  (high = +80.0°C, crit = +100.0°C)
Core 1:        +32.0°C  (high = +80.0°C, crit = +100.0°C)
Core 2:        +31.0°C  (high = +80.0°C, crit = +100.0°C)
Core 3:        +29.0°C  (high = +80.0°C, crit = +100.0°C)

acpitz-acpi-0
Adapter: ACPI interface
temp1:        +27.8°C  (crit = +119.0°C)

iwlwifi_1-virtual-0
Adapter: Virtual device
temp1:            N/A

nvme-pci-0100
Adapter: PCI adapter
Composite:    +26.9°C  (low  = -273.1°C, high = +82.8°C)
                       (crit = +84.8°C)
ERROR: Can't get value of subfeature temp2_min: I/O error
ERROR: Can't get value of subfeature temp2_max: I/O error
Sensor 1:     +26.9°C  (low  =  +0.0°C, high =  +0.0°C)
```

```
Found unknown chip with ID 0xfe00
Probing for Super-I/O at 0x4e/0x4f
```

## Issues

After some upgrade, I'm now getting a 503 from torrents.internal.forooghian.com.

Flood isn't starting, because

```
set 21 22:18:37 lawrence-OptiPlex-3080 env[48687]: /usr/bin/env: ‘node’: No such file or directory
set 21 22:18:37 lawrence-OptiPlex-3080 systemd[1]: flood@download.service: Main process exited, code=exited, status=127/n/a
set 21 22:18:37 lawrence-OptiPlex-3080 systemd[1]: flood@download.service: Failed with result 'exit-code'.
```

Well, I re-followed my old steps for installing Node (18, this time)

## iOS 16 / macOS Ventura

DNS stopped working - seemed to be using Google DNS (my secondary) instead of Optiplex. https://www.reddit.com/r/pihole/comments/xcu3tu/ios_16_dns_issue/ – sounds like "secondary" is a misnomer, it's not a fallback. So removed the secondary from TP-Link router DHCP

### Getting permissions working for Plex and Transmission

Going to create a group `media-library-editors` of users who can edit the stuff in `/mnt/storage/Plex`:

```
lawrence@lawrence-OptiPlex-3080:/mnt/storage$ sudo addgroup media-library-editors
Adding group `media-library-editors' (GID 1000) ...
Done.
```

And will add the `plex` and `debian-transmission` users (as well as myself) to it:

```
lawrence@lawrence-OptiPlex-3080:/mnt/storage$ sudo adduser plex media-library-editors
Adding user `plex' to group `media-library-editors' ...
Adding user plex to group media-library-editors
Done.
lawrence@lawrence-OptiPlex-3080:/mnt/storage$ sudo adduser debian-transmission media-library-editors
Adding user `debian-transmission' to group `media-library-editors' ...
Adding user debian-transmission to group media-library-editors
Done.
lawrence@lawrence-OptiPlex-3080:/mnt/storage$ sudo adduser lawrence media-library-editors
Adding user `lawrence' to group `media-library-editors' ...
Adding user lawrence to group media-library-editors
Done.
```

Now I need to fix up the permissions on the existing files. First set the group owner of everything to `media-library-editors`:

```
lawrence@lawrence-OptiPlex-3080:/mnt/storage$ sudo chown -R :media-library-editors /mnt/storage/Plex
```

We now want group owners and owner to have:

- `rwx` access to directories
  - `sudo find /mnt/storage/Plex -type d -exec chmod 770 '{}' \;`
- `rw` access to files
  - `sudo find /mnt/storage/Plex -type f -exec chmod 660 '{}' \;`

Hmm, no, something's wrong, for example it's giving me permission denied when I try to `cd` into `/storage/Plex/Movies/Trainspotting (1996) [1080p]`. It's like it doesn't think that I’m part of the group.

Interestingly, the `groups` command doesn't think I’m in the group:

```
lawrence@lawrence-OptiPlex-3080:/mnt/storage/Plex/Movies$ groups
lawrence adm cdrom sudo dip plugdev lpadmin sambashare
```

but in `/etc/group` I am:

```
lawrence@lawrence-OptiPlex-3080:/mnt/storage/Plex/Movies$ grep media /etc/group
media-library-editors:x:1000:plex,debian-transmission,lawrence
```

Hmm, here's what `man groups` says (emphasis mine):

```
Print group memberships for each USERNAME or, if no USERNAME is specified, for the current process **(which may differ if the groups database has changed)**.
```

If I add my name:

```
lawrence@lawrence-OptiPlex-3080:/mnt/storage/Plex/Movies$ groups lawrence
lawrence : lawrence adm cdrom sudo dip plugdev lpadmin sambashare media-library-editors
```

OK, so I re-logged in over SSH and now it's OK.

Final thing to do is to set the setgid bit on all of the directories, so that anything created within them is also owned by the `media-library-editors` group:

```
sudo find /mnt/storage/Plex -type d -exec chmod g+s '{}' \;
```

Erm, but now I’m confused. I deleted Sherlock (a bunch of shows) using the Plex UI, and it vanished in Plex but it's all still there in the file system. What? Scanned library and it's re-appeared. Erm. Oh, but after an upgrade of Plex it all is well. So there's clearly something going on weird with group membership not taking place until process restarted.

Ditto, I tried downloading a torrent and it's getting a permission denied error. Let's restart transmission:

```
lawrence@lawrence-OptiPlex-3080:/mnt/storage/Plex/TV Shows$ sudo systemctl restart transmission-daemon.service
```

Hmm, there's one final problem, though, which is that the Transmission daemon is not granting write permissions to group owners for new files:

```
-rw-r--r-- 1 debian-transmission media-library-editors 1954505535 dez 28 23:07 foo.mp4
```

I think that this has something to do with `umask` (https://en.wikipedia.org/wiki/Umask), which I had not quite got to in the book yet.

Current Transmission daemon process is running with `umask` 0022 (`cat /proc/141119/status | grep -i umask`) - this gives `r-x` permission for world and group

I don't fully understand `umask` – what I want is to take away `rwx` permissions for world, and to leave all permissions for owner and group. I think that's `umask 007`. I tried running that in my terminal and then creating a file, and it appears like this:

```
-rw-rw----  1 lawrence media-library-editors    0 dez 29 17:49  bar
```

So I think that's what I want. OK, in Transmission [I can set the `umask` property in the config file](https://github.com/transmission/transmission/blob/main/docs/Editing-Configuration-Files.md):

```json
"umask": "007"
```

No idea where my Transmission config file is (even though pretty sure edited it before). Ah, manpage + `/etc/password` and looking at `debian-transmission` user shows us it’s `/var/lib/transmission-daemon/.config`

Well, I'm still getting some weird permissions:

```
drwxr-sr-x 3 debian-transmission media-library-editors       4096 dez 29 18:01  bar
```

Compared to other things:

```
drwxrws--- 2 debian-transmission media-library-editors       4096 jun 24  2022 baz
```

It seems to have ignored my `umask`:

```
lawrence@lawrence-OptiPlex-3080:/mnt/storage/Plex/Movies$ cat /proc/1427487/status | grep -i umask
Umask:	0022
```

Hmm, it says in the config file documentation that umask is meant to be a string, but here it says it's a decimal number:

https://askubuntu.com/a/157471

So let's try

```json
"umask": 7
```

Wait, no, I just want to take away `w` permissions for world, so that's `umask 002`. Let's try

```json
"umask": 2
```

OK, now it looks good:

```
drwxrws---  3 debian-transmission media-library-editors       4096 dez 29 18:17  bar
```

I don't know why the directory has the setgid bit set, is that something that’s inherited? According to the [Wikipedia page for setgid](https://en.wikipedia.org/wiki/Setuid), yes:

> `Created subdirectories also inherit the setgid bit.`

I've noticed that when Plex deletes things, it only deletes the movie file and not the containing directory, which sometimes contains some other text files and stuff like that. [Sounds like](https://www.reddit.com/r/PleX/comments/7225si/delete_folder_containing_media_when_deleting_in/) there isn't much I can do about this.

## Some notes about network speed

Measure it using `iperf` (had to install on my laptop and on server). With my laptop on Wi-Fi (in same room as access point) and the server on Ethernet, it gives:

```
[lawrence@Lawrences-MacBook-Pro:~] iperf -c optiplex.internal.forooghian.com
------------------------------------------------------------
Client connecting to optiplex.internal.forooghian.com, TCP port 5001
TCP window size:  128 KByte (default)
------------------------------------------------------------
[  1] local 192.168.1.181 port 50190 connected with 192.168.1.151 port 5001 (icwnd/mss/irtt=11/1448/2000)
[ ID] Interval       Transfer     Bandwidth
[  1] 0.00-10.05 sec   749 MBytes   625 Mbits/sec
```

That's very fast indeed (about 13s for a gigabyte, I guess). I wonder how close to theoretical max that is?

The router is "TP-Link AX1500 Wi-Fi 6 Router":

- 5 GHz: 1201 Mbps (802.11ax)
- 2.4 GHz: 300 Mbps (802.11n)

But there seems to be something where it can combine both to get 1500 Mbps, not looked into it.

- 1× Gigabit WAN Port
- 4× Gigabit LAN Ports

And the Wi-Fi menu on my Mac says "Tx Rate: 1,200 Mbps" (seems to vary though) and "PHY Mode: 802.11ax"

So data is transferring at about half the theoretical max. I wonder where the bottleneck is. The `iperf` man page seems like an interesting place to read sometime.

But weirdly, fast.com tells me that my Internet speed is 830Mbps, which is faster than the internal speed, eh?

I wonder what I'd get if I used the USB LAN adapter on my computer.

## New config of home network (2023-06-17)

This is in response to having to take the TP-Link box out of the equation, due to:

> e.g. curl -vvv https://gamefaqs.gamespot.com/boards/718564-metal-gear-solid-v-the-phantom-pain/73084364 is both locally and on Optiplex giving “Connection reset by peer” on Optiplex. gateway 192.168.1.1 (run `route`), what is that? is it the wireless router?. TCP RST packet? Try opening a TLS connection to GameFAQs with ` openssl s_client -connect 199.232.208.194:443` and notice that it normally prints out very little and sometimes prints a load more. does not happen when using my hotspot. let’s turn back to connecting straight to modem and see what happens over ethernet. It seems to not happen, or happen very seldom. which is weird. Have reset modem to factory settings, probably for no good reason. then reset the access point to factory settings. Hmm, and now it’s happening again. Hmm, it seems to only be happening over (Wi-Fi + TP-Link) — that is (LAN + TP-Link) seems to work. No, but wait — that can’t be right, it was happening on OptiPlex too — yeah, and the curl command is failing over LAN with TP-Link too. OK, have just taken the TP-Link box away, and lovelyhorse is on the Claro box now. Hmm, I can’t find anything that looks like DHCP server settings. Well, Plex and OptiPlex and torrents are all inaccessible now.

So, what can I do without buying any new equipment?

- Have Claro box serve as modem, gateway, Ethernet switch, and wireless access point
  - No idea what we can expect speed-wise
- Have Optiplex now also serve as DHCP server
  - If I do that, and disable the DHCP server on the Claro box, then what IP will the Claro box use? Will it self-assign 192.168.0.1 or will it use DHCP?

OK, let's first of all [configure OptiPlex to have a static IP](https://sites.cns.utexas.edu/oit-blog/blog/how-set-static-ip-linux-machine) (192.168.0.22):

1. `hostnamectl set-hostname optiplex.internal.forooghian.com`
2. `echo "192.168.0.22 optiplex.internal.forooghian.com" >> /etc/hosts`
3. TODO their instructions aren't very clear, take a look at my book
   - https://wiki.debian.org/NetworkConfiguration#Configuring_the_interface_manually
   ```
   auto enp2s0
   iface enp2s0 inet static
    address 192.168.0.22/24
    gateway 192.168.0.1
   ```
   (`/etc/network/interfaces`)
   Oops, did `ip link set enp2s0 down` (with the intention of following it by `up`) but then that dropped my SSH and I had to power cycle the machine
   well, after plugging in to TV and `ip link set enp2s0 up` the interface didn't seem to get an IPv4 address
   After reboot of machine it comes back

## Setting up DHCP server

If multiple `DHCPOFFER` received, then it's [usually first one wins](https://superuser.com/questions/1370188/how-do-dhcp-clients-know-which-of-multiple-dhcpoffers-to-accept).

So, I'm curious — if we don't set up a DHCP server, and just statically assign OptiPlex an IP, what will happen? I _think_ that the router’s DHCP server [should use an ARP probe](https://networkengineering.stackexchange.com/questions/60272/arp-after-dhcp-discover) to check whether the address is available before assigning 192.168.0.22 to anyone else.

TODO

## Home Internet speed

It seems to have gone down recently. Speedtest.net’s CLI is showing me around 112.55 Mbps (independent of local connection type, and on both OptiPlex and MacBook Pro.

Weirdly their website suddenly now shows higher numbers (around 380). Eh? Ditto for their app on iPhone.

For example, [here](https://www.speedtest.net/result/14903690625) is a browser one, and [here](https://www.speedtest.net/result/c/f05154d3-0b2d-46da-a0a9-69b7abda21f2) is a command-line one, both taken on MBP over Ethernet. Hmm, well, now they're both really high again!

We're paying for a 500 Mbit connection.

Claro has a diagnostic tool in the account portal.

## Setting up Jellyfin

I don’t want to have to pay Plex Pass to get downloads. I’m sure that this’ll be clunkier, though (I have no idea how downloads work).

Following [these instructions](https://jellyfin.org/docs/general/installation/linux#debuntu-debian-ubuntu-and-derivatives-using-apt).

Remember the server is currently at 192.168.0.1.

After running that script, I get:

```
> Waiting 15 seconds for Jellyfin to fully start up.

-------------------------------------------------------------------------------
● jellyfin.service - Jellyfin Media Server
     Loaded: loaded (/lib/systemd/system/jellyfin.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/jellyfin.service.d
             └─jellyfin.service.conf
     Active: active (running) since Wed 2024-02-28 21:32:29 -03; 20s ago
   Main PID: 42843 (jellyfin)
      Tasks: 25 (limit: 4246)
     Memory: 96.7M
        CPU: 3.912s
     CGroup: /system.slice/jellyfin.service
             └─42843 /usr/bin/jellyfin --webdir=/usr/share/jellyfin/web --restartpath=/usr/lib/jellyfin/restart.sh --ffmpeg=/usr/lib/jellyfin-ffmpeg/…

fev 28 21:32:32 optiplex.internal.forooghian.com jellyfin[42843]: [21:32:32] [INF] ServerId: 48932916fd9440f7baebf229bbceaeff
fev 28 21:32:32 optiplex.internal.forooghian.com jellyfin[42843]: [21:32:32] [INF] Executed all pre-startup entry points in 0:00:00.0855854
fev 28 21:32:32 optiplex.internal.forooghian.com jellyfin[42843]: [21:32:32] [INF] Core startup complete
fev 28 21:32:32 optiplex.internal.forooghian.com jellyfin[42843]: [21:32:32] [INF] Executed all post-startup entry points in 0:00:00.1346874
fev 28 21:32:32 optiplex.internal.forooghian.com jellyfin[42843]: [21:32:32] [INF] Startup complete 0:00:02.9088764
fev 28 21:32:34 optiplex.internal.forooghian.com jellyfin[42843]: [21:32:34] [INF] StartupTrigger fired for task: Update Plugins
fev 28 21:32:34 optiplex.internal.forooghian.com jellyfin[42843]: [21:32:34] [INF] Queuing task PluginUpdateTask
fev 28 21:32:34 optiplex.internal.forooghian.com jellyfin[42843]: [21:32:34] [INF] Executing Update Plugins
fev 28 21:32:36 optiplex.internal.forooghian.com jellyfin[42843]: [21:32:36] [INF] Update Plugins Completed after 0 minute(s) and 1 seconds
fev 28 21:32:36 optiplex.internal.forooghian.com jellyfin[42843]: [21:32:36] [INF] ExecuteQueuedTasks
-------------------------------------------------------------------------------

You should see the service as 'active (running)' above. If not, use https://jellyfin.org/contact to find us for troubleshooting.

You can access your new instance now at http://:8096 in your web browser to finish setting up Jellyfin.

Thank you for installing Jellyfin, and happy watching!
```

So to finish setup now I go to http://192.168.0.22:8096/ and follow the wizard.

Let's set up a user account (details in 1Password).

Does this support https too?

Hmm, Plex doesn't seem to be able to access that folder. What's going on there?

Perhaps it’s a case of getting Jellyfin’s user into that `media-library-editors` group.

There’s a user called `jellyfin`.

```
sudo adduser jellyfin media-library-editors
```

Well, that doesn't seem to have worked; it still can't find `/mnt/storage/Plex/Movies`.

Oh, is it a case of restarting jellyfin? Yep, that worked

Sounds like Infuse is the client of choice for iOS / Mac (there's also the web interface).

OK, seems to be roughly working. Looks like I can use the same directory for Plex and Jellyfin and run them side by side for now, cool.

I think there’s a fair bit of Jellyfin understanding to get, though.

Also, how to make Jellyfin work over HTTPS?

(I told Jellyfin that it was allowed to poke my router to do some UPnP something for remote access.)
