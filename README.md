## IKEv2 VPN Server

IKEv2 VPN Server based on Alpine image. Support OS clients - macOS, iOS, Linux, Windows, Android.

#### 1. Install IKEv2 VPN Server on vps

#### 1.1 Clone this repository

    git clone https://github.com/VictorBurtsev/ikev2-vpn-server.git && cd ikev2-vpn-server

#### 1.2 Build docker image

    docker build --no-cache -t ikev2-vpn-server:alpine .

#### 1.3 Start the IKEv2 VPN Server

    docker run --cap-add=NET_ADMIN -d --name ikev2-vpn-server --restart=always -p 500:500/udp -p 4500:4500/udp ikev2-vpn-server:alpine

#### 2. Configure client for iOS / macOS

#### 2.1 Generate the .mobileconfig (for iOS / macOS) to the current path

    docker exec -it ikev2-vpn-server generate-mobileconfig > ikev2-vpn.mobileconfig

Transfer the generated `ikev2-vpn.mobileconfig` file to your local computer via SSH tunnel (`scp`) or any other secure methods.

#### 2.2 Install the .mobileconfig (for iOS / macOS)

- **iOS 9 or later**: AirDrop the `.mobileconfig` file to your iOS 9 device, finish the **Install Profile** screen;

- **macOS 10.11 El Capitan or later**: Double click the `.mobileconfig` file to start the *profile installation* wizard.

#### 3. Configure client for Linux

#### 3.1 Install Strongswan

    apt install strongswan

#### 3.2 Copy config in */etc/ipsec.conf*

    # ipsec.conf - strongSwan IPsec configuration file
    
    conn %default
        type=tunnel
        keyexchange=ikev2
        authby=secret
        ike=3des-sha1-modp1024!
        esp=3des-sha1!
        
    conn ikev2-vpn
        right=xxx.xxx.xxx.xxx # server ip 
        rightsubnet=0.0.0.0/0,::/0
        left=xx.x.x.x # client ip
        leftsubnet=10.8.0.0/16
        leftsourceip=%config
        auto=start

#### 3.3 Copy config in */etc/ipsec.secrets*

    : PSK "1235qQ28AG+Jtw68kQUJS/M8VDZ8GXMWQAHJJHGJHG="

#### 3.4 Option - if apparmor work on system add in file */etc/apparmor.d/usr.lib.ipsec.charon* line

    /etc/resolv.conf          w,

#### 3.5 Start linux client

    ipsec reload && ipsec restart && ipsec up ikev2-vpn

#### Technical Details

Upon container creation, a *shared secret* was generated for authentication purpose, no *certificate*, *username*, or *password* was ever used, simple life!

#### License

Copyright (c) 2018 Mengdi Gao, Nebukad93, vl_burtsev  This software is licensed under the [MIT License](LICENSE).
