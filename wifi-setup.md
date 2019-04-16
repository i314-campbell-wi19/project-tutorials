# WPA Supplicant Configuration Reference

##  Unencrypted Networks
Let's start by looking at the configuration for an unencrypted network. A network configuration block is structured as a list of `parameter=value` pairs enclosed within `network={ }`.

For every network type that we create, we will need to specify a value for the `ssid` parameter. The _service set identifier (SSID)_ is the network name that you see on your device when you connect to a wireless network. Since this name may include whitespace, we encode it in double quotes.

In addition to the _ssid_, _wpa_supplicant_ expects us to provide encryption related parameters or to explicitly disable encryption by setting the parameter `key_mgmt` to `NONE`.

Taking the _University of Washington_ wifi as an example, we would append the following configuration block to the configuration template we began above.

```
network={
    ssid="University of Washington"
    key_mgmt=NONE
}
```

## WPA2 Personal Networks
For a basic (non-enterprise) encrypted network, the configuration of the network block changes only slightly. Rather than specify the `key_mgmt` setting, we assign the network passphrase to the `psk` parameter.

There are two ways to accomplish this task. First, we can assign the passphrase directly to the parameter in plaintext as shown here. __This approach is not recommended.__

```
network={
    ssid="My Fancy Wifi"
    psk="super secret squirrels"
}
```

Security professionals generally frown on plaintext passwords and passphrases being written to configuration files or code. As such, we prefer to write the configuration based on the raw network key (computed using a function called _PBKDF_ in conjunction with _SHA1_).

You can generate the psk directly on your Pi by running the  `wpa_passphrase` utility. 

```bash
# Running on your Raspberry Pi

# Pass your SSID as the first argument
pi@titan:~ $ wpa_passphrase "My Fancy Wifi"

# Enter passphrase at the prompt (you won't see characters echoed back)
```

## WPA2 Enterprise Networks
Unlike home and coffee shop networks, enterprise networks like _Eduroam_, require a bit more setup since they authenticate individual users to the network as part of the process of establishing an encrypted connection. As such, these networks are substantially more secure than networks that are protected by _WPA2 Personal_. 

To configure your Pi for _Eduroam_ wireless, you can use the template below, substituting your own NetID and a hash computed from your UW password.

```
network={
        ssid="eduroam"
        scan_ssid=1
        key_mgmt=WPA-EAP
        eap=PEAP
        identity="netid@uw.edu"
        password=hash:6f9bad2c90b80bd549e595fc91e27806
        phase1="peaplabel=0"
        phase2="auth=MSCHAPV2"
}
```

We can compute the MD4 hash of your password from the command line in Linux as shown below:

```bash
# IMPORTANT: Run these commands in sequence. This prevents your netID password from being saved in the bash history file.
pi@titan:~ $ set +o history

pi@titan:~ $  echo -n 'This is your password' | iconv -t utf16le | openssl md4
# You should see output like 6f9bad2c90b80bd549e595fc91e27806

pi@titan:~ $ set -o history
```

_Note: MD4 does not provide a ton of protection for weak passwords. If you lose your memory card, I recommend updating your password._