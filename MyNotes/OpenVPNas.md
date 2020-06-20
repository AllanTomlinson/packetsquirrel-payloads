# OpenVPN Admin Server  

*Bear in in mind that although this has a nice GUI, it restricts you to only two clients connected.*  

Install instructions can be found [here](https://openvpn.net/vpn-server-resources/installing-openvpn-access-server-on-a-linux-system/)  

Software downloaded [here](https://openvpn.net/vpn-software-packages/)  

and install instructions are  

    apt update && apt -y install ca-certificates wget net-tools gnupg
    wget -qO - https://as-repository.openvpn.net/as-repo-public.gpg | apt-key add -
    echo "deb http://as-repository.openvpn.net/as/debian buster main">/etc/apt/sources.list.d/openvpn-as-repo.list
    apt update && apt -y install openvpn-as

Once `openvpn-as` is installed you will see:  

    Please enter "passwd openvpn" to set the initial
    administrative password, then login as "openvpn" to continue
    configuration here: https://10.128.0.3:943/admin

    To reconfigure manually, use the /usr/local/openvpn_as/bin/ovpn-init tool.

    +++++++++++++++++++++++++++++++++++++++++++++++
    Access Server 2.8.3 has been successfully installed in /usr/local/openvpn_as
    Configuration log file has been written to /usr/local/openvpn_as/init.log


    Access Server Web UIs are available here:
    Admin  UI: https://10.128.0.3:943/admin
    Client UI: https://10.128.0.3:943/
    +++++++++++++++++++++++++++++++++++++++++++++++

So make a hole in your firewall and see if you can log on. Make sure you connect using https. And accept the self-signed cerificate.  

NOTE: The web services by default actually run on port TCP 943, so you can visit them at https://192.168.70.222:943/ and https://192.168.70.222:943/admin/ as well. The OpenVPN TCP daemon that runs on TCP port 443 redirects incoming browser requests so that it is slightly easier for users to open the web interface by leaving the :943 part out.

## First time logging into Admin Web UI

### Administrative User

For the first use of the Admin Web UI, a single administrative user is initially added to the system. However, this user does not have a password set. In order to login, you must first run the following command on your server in order to set that:

    passwd openvpn

Once you’ve set your administrative user’s password, you can now open a browser and enter your Admin Web UI address.

    root@my-f1-micro:/home/dr_allan_tomlinson# passwd openvpn
    New password: 
    Retype new password: 
    passwd: password updated successfully
    root@my-f1-micro:/home/dr_allan_tomlinson# 

The admin account username is the default user `openvpn`

Go to Authenticaton/General and make sure Primary Authentication is set to 'local. This *is* the default  

See YouTube [Access Internal Networks with Reverse VPN connections - Hak5 1921](https://www.youtube.com/watch?v=b7qr0laM8kA) for how Darren uses this to set up an account for the turtle.  

He sets up users with 'autologin'  
