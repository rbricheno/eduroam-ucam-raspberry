# eduroam-ucam-raspberry

This covers [Raspbian](https://www.raspberrypi.org/downloads/raspbian/) [Buster](https://www.raspberrypi.org/blog/buster-the-new-version-of-raspbian/) and the [eduroam](https://www.eduroam.org/) network provided at the [University of Cambridge](https://help.uis.cam.ac.uk/service/wi-fi).

## The problem

Out of the box, Raspbian Buster on the Raspberry Pi will not connect to eduroam in Cambridge. It might not be obvious how to make it connect. The eduroam network appears on the list of wireless networks in the desktop GUI but is 'greyed out' or disabled.

## Prerequisites

 * Clean install of Raspbian Buster
 * Decent eduroam signal (be somewhere vaguely near an access point)
 * Raspberrry Pi with a working wireless card - All Pis from version 3 onwards have built in wifi which works fine. I tested this on a Pi 4 Model B.

You will also need the following Cambridge-specific things:

Prerequisite | Details
-------------|--------
CRSid | You got it when you joined the University. It's your ["Common Registration Scheme Identifier"](https://www.itservices.cam.ac.uk/services/user-accounts-and-security/user-accounts-and-passwords/accounts-and-passwords/common-registration-scheme-identifiers-crsids). It probably starts with your initials then a number. It looks like "spqr2"
eduroam identifier | Derived from your crsid. It's your crsid with "@cam.ac.uk" on the end. It looks like "spqr2@cam.ac.uk"
network access token | This is like a password, but used to access network resources. You can collect it from the [UIS Tokens Service web site](https://tokens.csx.cam.ac.uk/)
Cambridge root CA certificate | For the purposes of this document we will be using the Cambridge CA certificate which you can download [here ](https://help.uis.cam.ac.uk/service/wi-fi/other/wireless-ca.crt). For a better understanding of what your options are, and for information about how to verify the CA certificate, please read [the official documentation](https://help.uis.cam.ac.uk/service/wi-fi/other).

## Steps to connect

The GUI does not support connecting to networks like eduroam. You will need to edit some text files instead. **Do not click on the GUI while you are making the edits.** I haven't worked out the exact circumstances, but I have noticed that attempts to write eduroam config end up either disabled or deleted when interacting with the GUI.

All of the configuration that we need is stored in a file called `/etc/wpa_supplicant/wpa_supplicant.conf` (but that is not the only file we will need to edit...!).

Download the CA certificate mentioned in the prerequisites, and copy it somewhere memorable. I've copied mine (as root) to `/etc/wpa_supplicant/wireless-ca.crt`

Make a copy of `/etc/wpa_supplicant/wpa_supplicant.conf` before you start so you can restore it if something does wrong.

Now (as root) edit your `/etc/wpa_supplicant/wpa_supplicant.conf` file using your favourite text editor. The updated file will contain the line `update_config=0` to prevent the GUI from attempting to modify the contents of the file. It will also contain a new entry for the "eduroam" network. Once eduroam is working, take a copy of the edited file so you can easily restore the "eduroam" settings in future if you need to make changes to the config.

## wpa_supplicant.conf for eduroam in the University of Cambridge

The file should look like this:
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=0
country=GB

network={
        ssid="eduroam"
        proto=RSN
        key_mgmt=WPA-EAP
        eap=PEAP
        pairwise=CCMP
        group=CCMP
        identity="spqr2@cam.ac.uk"
        anonymous_identity="_token@cam.ac.uk"
        password="asdftokenqwertyu"
        ca_cert="/etc/wpa_supplicant/wireless-ca.crt"
        subject_match="/C=GB/ST=England/L=Cambridge/O=University of Cambridge/OU=University Information Services/CN=token.wireless.cam.ac.uk"
}
```
The lines you need to change are `identity` which should be your own eduroam identifier (containing your CRSid instead of `spqr2`) and `password` which should be your network access token from the tokens web site. You should not change the `anonymous_identity` from `_token@cam.ac.uk`.

## One weird trick

This config is now correct, but there is a problem with Raspbian Buster which prevents it from working. The easiest way to fix this is to edit one of the files which is part of Buster. *This fix only applies to Raspbian Buster* and is not required for earlier versions of Raspbian as far as I know.

Fortunately this is only a one line change and is the last step to get your eduroam working!

Take a copy of the file `/lib/dhcpcd/dhcpcd-hooks/10-wpa_supplicant` so you can restore it if something does wrong.

Now (as root) edit `/lib/dhcpcd/dhcpcd-hooks/10-wpa_supplicant` using your favourite text editor.

Around line 58, you should see the line:
```
    wpa_supplicant_driver="${wpa_supplicant_driver:-nl80211,wext}" 
```    

Replace that line with the following:
```
    wpa_supplicant_driver="${wpa_supplicant_driver:-wext,nl80211}"
```
That is to say, reverse the order of `wext` and `nl80211`.

Reboot your Raspberry Pi and your eduroam now works!







