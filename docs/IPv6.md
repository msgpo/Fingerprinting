### IPv6 MAC address "leakage" fingerprinting

#### Do I leak my MAC address by using IPv6 and how is that related to fingerprinting?

There is a lot misinformation regarding the security of IPv6. So let me get some things streight right from the beginning. IPv6 is compared to IPv4 not more insecure, it's true that the benefit in using IPv6 is (for most users) not existing but this does not make IPv6 "more insecure". The [information spread on the internet that IPv6 is more "attackable" compared to IPv4 is wrong](https://community.cisco.com/t5/networking-documents/understanding-ipv6-eui-64-bit-address/ta-p/3116953) and does not come from network experts, such statements are unqualified. 

IPv6 allows you to configure certain things, as always, the defaults aren't the strongest settings. The [network manager](https://developer.gnome.org/NetworkManager/unstable/nm-settings.html) can be controlled within the OS (e.g. under Linux) in order to "harden" IPv6. There are privacy concerns but that is all, we excluding in our statement any real attacks (CVE's & PoCs because that is _not always_ related to fingerprinting).

IPv6 offers "Privacy Extensions" which you can change or configure, this is basically what the user or IT-Admin can do to prevent any leakage". By design, the so called "leakage" can't be abused by advertiser to track your device or your location (we are going to explain this later in detail).


### Claim: "IPv6 addresses generated from your MAC address are easy to identify"

The address 2000:1234:5678:0000:0000:12**FF:FE**34:5678 is our example discussion scenario, which is left behind whenever you visit a website like e.g. Amazon.

This is not incorrect however the truth lies (as so often) between unqualified comments and the [RFC itself](https://www.tldp.org/HOWTO/Linux+IPv6-HOWTO/ch06s05.html). It's true that EUI-64 (which is basically the main concern we're going to adress here) might be a privacy problem, but not in the sense as you might think. 


### What is really going to happen?

[EUI64](https://community.cisco.com/t5/networking-documents/understanding-ipv6-eui-64-bit-address/ta-p/3116953).

Its a "mode" and our main privacy concern, in a sense that the MAC address could be used by an attacker/advertiser to track your real location or identify you or your device. The lower 8 bytes (depending on your physical MAC address) could be abused to get your real location or to identify you (uniqueness). Theoretically you could use a token instead ``ethernet.cloned-mac-address=$VALUE of the MAC, however that is not possible on each OS, that would basically obfuscate the MAC by itself. There are other techniques to avoid or to "spoof" the MAC address but let's assume that we don't want to involve third-party software or any scripts in order to do that.


### Can my random MAC address been used to fingerprint me?

In theory the MAC address changes (assuming we did not changed the configuration), so you're unqiue because you get a random number every xxxx seconds. This could be used to track you, the attacker/store/tracker could use this unqiue identifiers, to allow them to track my activities. No, they can't! That's the good thing. 

The addresses are not always unique, assuming nothing was changed or spoofed in the network segment (layer number 2). Some vendors do not known to not giving away the same MAC addresses, which means layer 2 of routing the packages are "broken". The outcome is that we see two identical MAC addresses on the same network, this also happens _sometimes_ on desktop machines and is not limited to e.g. tablet devices. Basically there is no gurantee that manufacturer x does not (by accident) reuses MACs.

Router [(auto)-generated MAC adddresses aren't routable](http://en.wikipedia.org/wiki/IPv6_address#Modified_EUI-64). They look like `fe80::`. 


### Protection
* [Configure your network manager](https://www.usenix.org/system/files/login/articles/105438-Barrera.pdf) (net config).
* [RFC 7217](https://tools.ietf.org/html/rfc7217) allows the primary, static address to be generated from an opaque hash which does not reveal any information.
* Use a VPN which supports IPv6 (they usually do not leak any private data), the ones which do not supporting IPv6 typical blocking the entire traffic. It's not needed to disable IPv6 entirely, there is also no surface for attacks because the interface will not submit anything.
* [RFC 4941](https://tools.ietf.org/html/rfc4941) or known as "Privacy Addressing", basically lets outbound connections use temporary, randomly generated addresses (which are rotated every few hours).
* Most operating systems (by default) already migrating the privacy concerns and there is no reason to be worried since the possible [leaked information is not enough to track you](https://tools.ietf.org/html/rfc7721).


### Protection under Windows (Vista+)

This feature should be active by default, if not overwritten by any script or "tweak" or GPO/registry setting.

* First we are going to check if the privacy features are currently been used, this can be done via `Get-NetIPv6Protocol | fl RandomizeIdentifiers`
* Configure the features manually `Set-NetIPv6Protocol -RandomizeIdentifiers Enabled` or `Set-NetIPv6Protocol -RandomizeIdentifiers Disabled` (as per needs).


### Protection under Linux (Network Manager 1.2.0+)

Linux supports RFC 7217 within the network manager. The tweak is usually not needed because it should be configured by default.

```
nmcli con modify "<your-profile-name>" ipv6.addr-gen-mode stable-privacy
nmcli con modify "<your-profile-name>" ipv6.addr-gen-mode eui64
```

Several different kernel versions providing additional RFCs e.g. starting with 4.1.0+ we can use SLAAC. 


### Protection on other operating systems

There is a real good guide avbl. [here](https://superuser.com/questions/243669/how-to-avoid-exposing-my-mac-address-when-using-ipv6). 


### What have we learned

* MAC addresses are just not unique enough to be reliably used for tracking individuals, sometimes they are not even unique. This is a "manufacturer problem" to deal with. Manufacturer reuses MACs (_it's not only a theory_).
* Proving that an MAC is unique is difficult (not in the same network) but we need to consider that the single
* Using a NAT/Proxy/VPN makes it _impossible_ to foreign hosts to see who is really behind, we are not talking about IP routing!
* There are better tracking mechanism which are more reliable out there.
* MAC addresses do not leak beyond your local link, which means machines on the same LAN as you see your MAC address, other people do not! 
* [Depending on the OS, the implementation is different](https://arstechnica.com/gadgets/2016/09/macos-10-12-sierra-the-ars-technica-review/6/) which means you're protected out of the box or they reduce the possible tracking scenario (no user-interaction needed).
* Some routers using other [RFCs or implementation](https://labs.ripe.net/Members/johanna_ullrich/ipv6-addresses-security-and-privacy) (this is not covered in this overview), please check your router and how it handles IPv6 on your network. 
* Some ISPs might use [larger blocks than 48 bits](http://superuser.com/questions/788717/debugging-ipv6-routing-issues/788724#788724), this is also not covered here, but it's irrelevant in terms of fingerprinting. 