# OpenVPN Setup  

from [OpenVPN Client Tunneling! - Hak5 2308](https://www.youtube.com/watch?v=OlKztGlt8VA), 2017.  

This installs a command line VPN with no restrictions. If you want a GUI then install the OpenVPN Admin Server `opnvpn-as`; but this will restrict you to 2 users.

## On the Cloud Server  

Get the open vpn install script from git.

    10.128.0.3:~$ wget https://git.io/vpn -O openvpn.sh
    --2020-06-15 14:09:48--  https://git.io/vpn
    Resolving git.io (git.io)... 

    10.128.0.3:~$ head openvpn.sh 
    #!/bin/bash
    #
    # https://github.com/Nyr/openvpn-install
    #
    # Copyright (c) 2013 Nyr. Released under the MIT License.

Chmod to execute the file then run it as root.

    # Discard stdin. Needed when running from an one-liner which includes a newline
    read -N 999999 -t 0.001

    10.128.0.3:~$ sudo openvpn.sh  

Accept the defaults and it will download, install and configure.

    Processing triggers for systemd (241-7~deb10u4) ...

    init-pki complete; you may now create a CA or requests.
    Your newly created PKI dir is: /etc/openvpn/server/easy-rsa/pki


    Using SSL: openssl OpenSSL 1.1.1d  10 Sep 2019
    Generating RSA private key, 2048 bit long modulus (2 primes)
    ............+++++
    .....+++++
    e is 65537 (0x010001)

    Using SSL: openssl OpenSSL 1.1.1d  10 Sep 2019
    Generating a RSA private key
    ...........+++++
    ..................................+++++
    writing new private key to '/etc/openvpn/server/easy-rsa/pki/easy-rsa-2899.QDmlSF/tmp.3wFR7l'
    -----
    Using configuration from /etc/openvpn/server/easy-rsa/pki/easy-rsa-2899.QDmlSF/tmp.sclm6z
    Check that the request matches the signature
    Signature ok
    The Subject's Distinguished Name is as follows
    commonName            :ASN.1 12:'server'
    Certificate is to be certified until Jun 13 14:39:00 2030 GMT (3650 days)

    Write out database with 1 new entries
    Data Base Updated

    Using SSL: openssl OpenSSL 1.1.1d  10 Sep 2019
    Generating a RSA private key
    ................................+++++
    .+++++
    writing new private key to '/etc/openvpn/server/easy-rsa/pki/easy-rsa-2974.YD4JZM/tmp.D44a58'
    -----
    Using configuration from /etc/openvpn/server/easy-rsa/pki/easy-rsa-2974.YD4JZM/tmp.i5vCs3
    Check that the request matches the signature
    Signature ok
    The Subject's Distinguished Name is as follows
    commonName            :ASN.1 12:'client'
    Certificate is to be certified until Jun 13 14:39:01 2030 GMT (3650 days)

    Write out database with 1 new entries
    Data Base Updated

    Using SSL: openssl OpenSSL 1.1.1d  10 Sep 2019
    Using configuration from /etc/openvpn/server/easy-rsa/pki/easy-rsa-3032.vX9aZJ/tmp.3Bi2ve

    An updated CRL has been created.
    CRL file: /etc/openvpn/server/easy-rsa/pki/crl.pem


    Created symlink /etc/systemd/system/multi-user.target.wants/openvpn-iptables.service → /etc/systemd/system/openvpn-iptables.service.
    Created symlink /etc/systemd/system/multi-user.target.wants/openvpn-server@server.service → /lib/systemd/system/openvpn-server@.service.

    Finished!

    The client configuration is available in: /root/client.ovpn
    New clients can be added by running this script again.
    10.128.0.3:~$ 

This produces a client configuration file under root - client.ovpn. If you want to add more user accounts just run the script again and it will generate more .ovpn files.  

So copy the client.ovpn file over to your client.  

And the script also starts up the open vpn server, listening on **UDP** port 1194.

    10.128.0.3:~$ sudo netstat -anup
   Active Internet connections (servers and established)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
    udp        0      0 10.128.0.3:1194         0.0.0.0:*                           3117/openvpn        
    udp        0      0 127.0.0.1:323           0.0.0.0:*                           399/chronyd         
    udp        0      0 0.0.0.0:68              0.0.0.0:*                           308/dhclient    `

So on the server, after running the script you should see a tun0 interface.  

    tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 10.8.0.1  netmask 255.255.255.0  destination 10.8.0.1
        inet6 fe80::8cda:84f:d341:4c44  prefixlen 64  scopeid

Note the IP address `10.8.0.1`. When the client connects it will be on this network, usually the next one up, so `10.8.0.2`.

So back on the client you can now run

    $sudo openvpn --config ./client.ovpn 

And you should get  

    tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 10.8.0.2  netmask 255.255.255.0  destination 10.8.0.2

And from the server I can now see my client

    10.128.0.3:~$ ping 10.8.0.2
    PING 10.8.0.2 (10.8.0.2) 56(84) bytes of data.
    64 bytes from 10.8.0.2: icmp_seq=1 ttl=64 time=115 ms

And vice versa.  

Now, from the server, I can ssh in to any host on my network. If I can ping it, I can ssh or rdp or vnc to it.  In other words, if I can set up an openvpn client on a network then I have a backdoor in to that network. And, in this case, I have set up a vpn from my laptop to my cloud server, all my traffic is now going out through the cloud server via the vpn.  

But the Squirrel lets you run the backdoor *without* tunneling network traffic through the vpn, **or** as a vpn tunnel with *no* backdoor. This is configured on line 5 of `payloads.sh` by setting FOR_CLIENTS equal to zero or one. Is this why we need to know which RJ45 port on the Squirrel is 'In' and which is 'Out'? (The 'Out' is next to the USB and goes to the network; the 'In' is connected to the client.)

And you can run the same payload on a pineapple. So you could relay a wifi signal through the tunnel. (?)
