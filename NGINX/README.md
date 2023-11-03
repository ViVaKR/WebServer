# bind



## `named.conf` file location

* `/opt/homebrew/Cellar/bind/9.18.19/.bottle/etc/bind`
* `/etc/named.conf`


```bash

    sudo rndc-confgen > /etc/rndc.conf

```

#!/bin/bash

# Create a custom launch key for BIND

/usr/local/sbin/rndc-confgen > /etc/rndc.conf

head -n 6 /etc/rndc.conf > /etc/rndc.key

# Set up a basic named.conf file.

192.168.0.8
192.168.0.11
192.168.0.12


code /opt/homebrew/Cellar/bind/9.18.19/.bottle/etc/bind/name.conf
// cat > /usr/local/homebrew/Cellar/bind/9.10.0-P2/etc/named.conf  <<END
//
// Include keys file
//
include "/etc/rndc.key";

// Declares control channels to be used by the rndc utility.
//
// It is recommended that 127.0.0.1 be the only address used.
// This also allows non-privileged users on the local host to manage
// your name server.

//
// Default controls
//
controls {
        inet 127.0.0.1 port 54 allow {any;}
        keys { "rndc-key"; };
};

options {
        directory "/var/named";
};

// 
// a caching only nameserver config
// 
zone "." IN {
    type hint;
    file "named.ca";
};

zone "localhost" IN {
    type master;
    file "localhost.zone";
    allow-update { none; };
};

zone "0.0.127.in-addr.arpa" IN {
    type master;
    file "named.local";
    allow-update { none; };
};

logging {
        category default {
                _default_log;
        };

        channel _default_log  {
                file "/Library/Logs/named.log";
                severity info;
                print-time yes;
        };
};

END

# Symlink Homebrew's named.conf to the typical /etc/ location. 
sudo ln -s /opt/homebrew/Cellar/bind/9.18.19/.bottle/etc/bind/named.conf /etc/named.conf

# Create directory that bind expects to store zone files
```bash
mkdir /var/named
chown -R $USERNAEM /var/named
curl http://www.internic.net/domain/named.root > /var/named/named.ca
```

# 3) CREATE A LuanchDaemon FILE: 

cat > /System/Library/LaunchDaemons/org.isc.named.plist <<END
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>Disabled</key>
        <false/>
        <key>EnableTransactions</key>
        <true/>
        <key>Label</key>
        <string>org.isc.named</string>
        <key>OnDemand</key>
        <false/>
        <key>ProgramArguments</key>
        <array>
                <string>/usr/local/sbin/named</string>
                <string>-f</string>
        </array>
        <key>ServiceIPC</key>
        <false/>
</dict>
</plist>
END

chown root:wheel /System/Library/LaunchDaemons/org.isc.named.plist 
chmod 644 /System/Library/LaunchDaemons/org.isc.named.plist 

# Shutdown bind (if it was running)
#launchctl unload /System/Library/LaunchDaemons/org.isc.named.plist


# Launch BIND and set it to start automatically on system reboot.
launchctl load -wF /System/Library/LaunchDaemons/org.isc.named.plist

To start bind now and restart at startup:
  $ sudo brew services start bind
Or, if you don't want/need a background service you can just run:
  $ /opt/homebrew/opt/bind/sbin/named -f -L /opt/homebrew/var/log/named/named.log



## 역방향 조회 영역 (Reverse Name Resolution Zone)

## (ex) 10.0.1.20 ~ 10.0.1.25 

$ORIGIN 1.0.10.in-addr.arpa.
$TTL 86400
@     IN     SOA    dns1.example.com.     hostmaster.example.com. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

      IN     NS     dns1.example.com.
      IN     NS     dns2.example.com.

20    IN     PTR    alice.example.com.
21    IN     PTR    betty.example.com.
22    IN     PTR    charlie.example.com.
23    IN     PTR    doug.example.com.
24    IN     PTR    ernest.example.com.
25    IN     PTR    fanny.example.com.

## (ex) named.conf

zone "1.0.10.in-addr.arpa" IN {
  type master;
  file "example.com.rr.zone";
  allow-update { none; };
};

---

SOA : defines information about the area. In this case the name of the primary DNS server "sid.example.com." and the email address of technical contact (root.example.com.; the @ is replaced by a dot). It is composed of several fields:
1. Serial : is the whole non-signed 32 bits. This is the serial number to increment with each change of file. It allows the secondary server to reload the information they have. The general purpose is to format it this way YYYYMMDDXX, either for the first amendment 01/04/2007 -> 2007040101, for the second 2007040102.
2. Refresh : defines the data refresh period.
3. Retry : if an error occurs during the last refresh, it will be repeated at the end of time Retry.
4. Expires: the server is considered unavailable after the time expires.
5. Negative cache TTL: set the lifetime of a NXDOMAIN response from us.
NS: information on behalf of nameservers for the domain.
MX.: information on the mail server. Many can be defined. Thus, it is possible to give them a priority, assigning a number. The lower the number, the higher the priority.
A: associates a host name to an IPv4 address (32 bits)
AAAA: associates a host name to an IPv6 address (128 bits)
CNAME: identifies the canonical name of an alias (a name that points to another name)
PTR: This is simply the inverse resolution (the opposite of type A).

---

## `named-checkzone buddham.co.kr /var/named/buddham.co.kr.zone`

## `named-checkzone 0.168.192.in-addr.arpa /var/named/192.168.0.rev`

## `named-checkconf /etc/named.conf`

1차 네임서버 정보
호스트이름 : ns1.ibrain.kr
IP 주소 : 221.143.43.39

2차 네임서버 정보
호스트이름 : ns2.ibrain.kr
IP 주소 : 162.159.24.151


sudo chown -R $USERNAME /opt/homebrew/Cellar/bind/9.18.19/sbin
sudo chown -R $USERNAME /opt/homebrew/Cellar/bind/9.18.19/sbin/named
sudo chown -R $USERNAME /opt/homebrew/opt/bind
sudo chown -R $USERNAME /opt/homebrew/opt/bind/bin
sudo chown -R $USERNAME /opt/homebrew/opt/bind/sbin
sudo chown -R $USERNAME /opt/homebrew/var/homebrew/linked/bind

sudo chown -R $USERNAME /opt/homebrew/Cellar/bind/9.18.19/sbin
sudo chown -R $USERNAME /opt/homebrew/Cellar/bind/9.18.19/sbin/named
sudo chown -R $USERNAME /opt/homebrew/opt/bind
sudo chown -R $USERNAME /opt/homebrew/opt/bind/bin
sudo chown -R $USERNAME /opt/homebrew/opt/bind/sbin
sudo chown -R $USERNAME /opt/homebrew/var/homebrew/linked/bind


controls {
    inet 127.0.0.1 port 54 allow { any; };
    keys { "rndc-key"; };
};


include "/etc/rndc.key";
