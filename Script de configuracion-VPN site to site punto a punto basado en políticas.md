Script de configuración



**Configuración básica del ISP**

hostname ISP

interface Ethernet0/0

&#x20;description ENLACE\_A\_R1

&#x20;ip address 10.7.25.2 255.255.255.252

&#x20;no shutdown

interface Ethernet0/1

&#x20;description ENLACE\_A\_R2

&#x20;ip address 10.7.25.6 255.255.255.252

&#x20;no shutdown



**Configuración básica de R1**

hostname R1

interface Ethernet0/0

&#x20;description WAN\_HACIA\_ISP

&#x20;ip address 10.7.25.1 255.255.255.252

&#x20;no shutdown

interface Ethernet0/1

&#x20;description LAN\_A

&#x20;ip address 10.7.25.65 255.255.255.224

&#x20;no shutdown

ip route 0.0.0.0 0.0.0.0 10.7.25.2



**Configuración básica de R2**

hostname R2

interface Ethernet0/0

&#x20;description WAN\_HACIA\_ISP

&#x20;ip address 10.7.25.5 255.255.255.252

&#x20;no shutdown

interface Ethernet0/1

&#x20;description LAN\_B

&#x20;ip address 10.7.25.97 255.255.255.224

&#x20;no shutdown

ip route 0.0.0.0 0.0.0.0 10.7.25.6



**Configuración IPSec IKEv1 en R1**

crypto isakmp policy 10

&#x20;encr aes 256

&#x20;authentication pre-share

&#x20;group 5

crypto isakmp key VPN12345 address 10.7.25.5

ip access-list extended VPN-TRAFFIC

&#x20;permit ip 10.7.25.64 0.0.0.31 10.7.25.96 0.0.0.31

crypto ipsec transform-set TS-IKEV1 esp-aes 256 esp-sha-hmac

&#x20;mode tunnel

crypto map VPN-MAP 10 ipsec-isakmp

&#x20;description VPN\_IKEV1\_HACIA\_R2

&#x20;set peer 10.7.25.5

&#x20;set transform-set TS-IKEV1

&#x20;set pfs group5

&#x20;match address VPN-TRAFFIC

interface Ethernet0/0

&#x20;crypto map VPN-MAP



**Configuración IPSec IKEv1 en R2**

crypto isakmp policy 10

&#x20;encr aes 256

&#x20;authentication pre-share

&#x20;group 5

crypto isakmp key VPN12345 address 10.7.25.1

ip access-list extended VPN-TRAFFIC

&#x20;permit ip 10.7.25.96 0.0.0.31 10.7.25.64 0.0.0.31

crypto ipsec transform-set TS-IKEV1 esp-aes 256 esp-sha-hmac

&#x20;mode tunnel

crypto map VPN-MAP 10 ipsec-isakmp

&#x20;description VPN\_IKEV1\_HACIA\_R1

&#x20;set peer 10.7.25.1

&#x20;set transform-set TS-IKEV1

&#x20;set pfs group5

&#x20;match address VPN-TRAFFIC

interface Ethernet0/0

&#x20;crypto map VPN-MAP







