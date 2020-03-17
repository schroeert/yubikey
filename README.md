# SSH Privatekey Authentication with Yubikey  
This is just a quick writeup on how I use my Yubikey for SSH Authentication.
I decided against OTP because that requires server-side modifications.
All in all it turned out to be a very handy solution.

In my case I used a  [Yubikey 5 NFC](https://www.yubico.com/product/yubikey-5-nfc).

## Dependencies
**In my Installation I used Arch Linux, packagemanagers and directorys may be different**

The Software in this guide is just required on you Client machine.

Open Smartcard Reader (Required on all Clients)
```
sudo pacman -S opensc
```

Yubico-piv-tool (Only required for key generation)
```
sudo yay -S yubico-piv-tool
```

## Pin
The default PIN of your Yubikey is 123456. We are going to verify an change your PIN.

```
yubico-piv-tool --action verify-pin -P <default_pin>
```

Now we are changing you PIN.

```
yubico-piv-tool --action change-pin  --pin <default_pin> --new-pin <your_pin>
```

Verify your PIN.

```
yubico-piv-tool --action verify-pin -P <your_pin>
```

## Key generation
This will take some Time!

```
yubico-piv-tool --slot 9a --action generate -o public.pem
```

Add your PIN
```
yubico-piv-tool --action verify-pin -P <your_pin> --action selfsign-certificate --slot 9a --subject "/CN=SSHKEY/" --input public.pem --output private.pem
```

Import private key

```
yubico-piv-tool --action import-certificate --slot 9a --input private.pem
```
Add this point you may delete the temporary generated keys. The Private Key is stored on your Yubikey. 

## Extract Public Key
Depending on your distro, this path may be different (Ubuntu /usr/lib/x86_64-linux-gnu/opensc-pkcs11.so).
```
ssh-keygen -D /usr/lib/opensc-pkcs11.so > publickey.pub
```
To use your Public Key as default you need to copy it into ```~/.ssh/id_rsa.pub```
```
cp publickey.pub ~/.ssh/id_rsa.pub
```
## Server side
To authenticate with you destination server, you need to copy your ssh key to the server.

```
ssh-copy-id -f user@destination.server
```

## SSH Client
Modify your SSH Config ```/etc/ssh/ssh_config```

Add the following line:
```
PKCS11Provider /usr/lib/opensc-pkcs11.so
```

At this point you should be good to go and be able to access your server with your Yubikey.
SSH into your server will only require your Yubikey PIN.

