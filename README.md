# USG-DsLite
Dual Stack Lite workaround for Unifi Security Gateway

## Purpose
This workaround adds Dual Stack Lite (defined in RFC 6333 and RFC 6334) capabilities to the Unifi Security Gateway. For this connection type no useable IPv4 address is provided by the ISP. Instead the Gateway has to create an ipip6 tunnel to a so called B4 elements which does a NAT and provides a IPv4 address to the Gateway. The IPv4 address is shared between several Gateways. This connection type is widely used in Germany.
Unifi devices do not support this connection type out of the box.

## Installation
Simply copy the content of this reporitory to the /config/scripts folder of your Unifi Security Gateway. Restart the device or reconnect the pppoe interface and check wether the ipip6 tunnel is available. You can do this by runnign:
```
/sbin/ifconfig
```
You should see an interface of the form dslite_pppoe0. The pppoe interface can still be configured using the Unifi Network Application. Also Prefix Delegation can be configured normally.

## How it works
By default the USG uses the dhcp6c client from the whide-dhcpv6 client. This client does not support requestion dhcp option 64. This option contains the address of the B4 element and requesting this option is mandatory according to the specification. To overcome this limitaion the [dhcpcd](https://github.com/NetworkConfiguration/dhcpcd) was cross compiled for the USG. The ppp directory contains hooks which run the dhcpcd client and creates an ipip6 tunnel when the ppp interface starts up.
This implementation is not perfect. The cleanest solution would be an implementaiton of dhcp option 64 into the wide-dhcpv6 package.

## Troubleshooting
+ The dhcpcd client writes the aftr address (Address of the B4 element) to /var/aftr-addr-\<interface_name\>. Make sure that dhcpcd runs and the file exists.
+ This implementation expects the ISP to provide an IPv4 address (which is assigned, but not useable). If you do not receive an IPv4 address from your ISP you can either assign a static one (The address does not matter as it will never be used) or you can have a look at [this tutorial](https://novag.github.io/posts/ubiquiti-usg-dual-stack-lite/) for a workaround.
+ The dhcpcd client was compiled for mips64 architecture. For different hardware the package might have to be recompiled.

A big thanks to [this tutorial](https://novag.github.io/posts/ubiquiti-usg-dual-stack-lite/) which shows how to set up the ipip6 tunnel on unifi devices.
