# OpenVPN Install from scratch  

From [Linux Server Build: OpenVPN From Scratch - Hak5 2019](https://youtu.be/XcsQdtsCS1U?list=PLW5y1tjAOzI3YEamgvUlLvtiiQwMsTj3H), 2016.

Most of this is scripted in Nyr's [install script](https://github.com/Nyr/openvpn-install) on Git.

This describes how to install OpenVPN without a GUI but it has no restrictions on the number of users.  

There is also a [repo on git](https://github.com/StarshipEngineer/OpenVPN-Setup) with an installation for Pi.  

## Server Software Installation

Install:  

- openvpn
- easy-rsa

Then take the templates and modify them. For the *server*, go to `/usr/share/doc/openvpn/examples`.  Under `samples-config-files` there are a whole bunch of config files and a README which says:  

    Sample OpenVPN Configuration Files.
    These files are part of the OpenVPN HOWTO which is located at:
    http://openvpn.net/howto.html

The `server.conf.gz` should be gunzipped and put it in the `/etc/openvpn/server/` directory. But the Nyr script does all this and configures `server.conf`. However, the Nyr config file is undocumented, the gzipped file has very thorough comments throughout so worth a look.  

Darren configures the following: 

- DH modulus: default is 1024, he changes this to 2048.  So change dh.pem to dh2048.pem. Nyr *doesn't* do this.  
- `;push "redirect-gateway def1 bypass-dhcp"` needs to be uncommented. Nyr does this. This *pushes* the redirect directive to the client. Read the gzipped conf file.
- We also need to uncomment the push DNS options to the client to make sure DNS requests go through the tunnel. Nyr does this.  
- Uncomment the `nobody`s to make sure it runs as an unprivileged user. Nyr does this.

That's it.

## Firewall Configuration
First thing to do is to make sure that this machine will forward traffic. This is configured by `/proc/sys/net/ipv4/ip_forward`. Cat this and make sure it's set to '1'. If not, echo 1 > into the file. Nyr's script sets this.

Now we need to enable forwarding to happen at boot. This is done in `/etc/sysctl.conf`. We need to uncomment `net.ipv4.ip_forward=1`. Nyr doesn't do this. I changed it.
Actually the script modifies `/etc/sysctl.d.30-openvpn-forward.conf` to do this.

### IPtables
Darren saves the config as snippets, on his wiki. But looks at `ufw` first, which helps edinting firewall rules. This is not installed on the Google cloud machine. And Nyr's script doesn't seem to set anything with `ufw` **but** it does set up some `iptables` rules. 

Perhaps we just rely on the Google cloud GUI to configure the firewall? But lets install `ufw` anyway and have a look.

`ufw status` then tells us that the firewall is inactive. Darren uses `ufw` to add rules as follows:

- `ufw allow ssh`
- `ufw allow 1194/udp`

Next he edits the ufw config file `/etc/default/ufw`. By default it drops packets. This is set by DEFAULT_FORWARD_POLICY which he chanfes to ACCEPT.

That's it. But, given the Nyr script, I didn't do any of this. I alreadt have a bunch of stuff set up from the script.  

    root@my-f1-micro:/usr/share/doc/openvpn/examples/sample-config-files# iptables -L
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination
    ACCEPT     udp  --  anywhere             anywhere             udp dpt:openvpn

    Chain FORWARD (policy ACCEPT)
    target     prot opt source               destination
    ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
    ACCEPT     all  --  10.8.0.0/24          anywhere

    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination

### NAT and MASQUERADING for Clients

This is all about POSTROUTING. I'm not going to mess with this because the Nyr script sets up `iptables` rules. See above. In fact the script creates a service to set up *persistent* `iptables` rules. The service config is in `/etc/systemd/system/openvpn-iptables.service`and looks like this:

    [Unit]
    Before=network.target
    [Service]
    Type=oneshot
    ExecStart=/usr/sbin/iptables -t nat -A POSTROUTING -s 10.8.0.0/24 ! -d 10.8.0.0/24 -j SNAT --to 10.128.0.3
    ExecStart=/usr/sbin/iptables -I INPUT -p udp --dport 1194 -j ACCEPT
    ExecStart=/usr/sbin/iptables -I FORWARD -s 10.8.0.0/24 -j ACCEPT
    ExecStart=/usr/sbin/iptables -I FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
    ExecStop=/usr/sbin/iptables -t nat -D POSTROUTING -s 10.8.0.0/24 ! -d 10.8.0.0/24 -j SNAT --to 10.128.0.3
    ExecStop=/usr/sbin/iptables -D INPUT -p udp --dport 1194 -j ACCEPT
    ExecStop=/usr/sbin/iptables -D FORWARD -s 10.8.0.0/24 -j ACCEPT
    ExecStop=/usr/sbin/iptables -D FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
    RemainAfterExit=yes
    [Install]
    WantedBy=multi-user.target

## Keys and Certs

Again I'm not going to mess with this, the Nyr script sets it up. It's all done with `easy-rsa`. This generates the certificates and is configured via `/etc/openvpn/easy-rsa.vars` which is copied from the tempalte at `usr/share/easy-rsa`.  The "key name" is the name of the DH file above. It should match because this is what will be generated.

Then in `/easy-rsa` there's a sript to build a CA server, generate a key and add it to the CA's database. The Nyr script does this. The keys and certificates are put in the `/etc/openvpn/server` directory.

There are sample client config files in the openvpn docs folder, same as the server.conf. You can create and edit ca.crt client.crt client.key and set these up for the client. You need to check and configure addresses and nic names. Then combine all three files into a `client.ovpn` file.

## Client Setup

Copy the `client.ovpn` file to your client and your done.