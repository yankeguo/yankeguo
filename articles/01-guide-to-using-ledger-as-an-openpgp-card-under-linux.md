Ledger officially provides OpenPGP app, but the configuration process under Linux is a bit troublesome. I spent a lot of effort to overcome the difficulties, thus I recorded it here for reference.

## Prepare your Ledger device

### 1. Install the OpenPGP app on Ledger device

- Open `Ledger Live` desktop application
- Turn on `Developer Mode` to make OpenPGP app available for installation
- Install the OpenPGP app to Ledger device

See also: https://support.ledger.com/article/115005200649-zd

### 2. Connect Ledger device to Linux host

- Connect your Ledger device to a Linux host
- Open the OpenPGP app on Ledger device

## Prepare your Linux host

### 1. Install required packages

Ubuntu/Debian for example:

```shell
sudo apt update
sudo apt install -y scdaemon pcscd pcsc-tools libccid usbutils
```

### 2. Allow current user to access Smart Card Reader

a Polkit rule is required to allow the current user to access the Smart Card Reader.

Create a file at `/usr/share/polkit-1/rules.d/allow-smartcard.rules` with content below:

```javascript
// replace YOUR_USERNAME with your username
polkit.addRule(function (action, subject) {
  if (
    (action.id == "org.debian.pcsc-lite.access_card" ||
      action.id == "org.debian.pcsc-lite.access_pcsc") &&
    subject.user == "YOUR_USERNAME"
  ) {
    return polkit.Result.YES;
  }
});
```

### 3. Enable and restart `pcscd` service

```shell
sudo systemctl enable pcscd
sudo systemctl restart pcscd
```

### 4. Ensure Ledger device is recognized by `libccid`

Run following command to check if Ledger device is already recognized as a Smart Card Reader

```shell
sudo pcsc_scan -r
```

If your Ledger device is not listed, you need to do following steps:

**1. Gather information of your Ledger device**

Execute `lsusb`, you will find something like this

> Bus 002 Device 005: ID 2c97:4009 Ledger Nano X

This means your Ledger device have:

- `VendorID` = `0x2C97`
- `ProductID` = `0x4009`
- `DisplayName` = `Ledger Nano X`

**2. Edit `/etc/libccid_Info.plist`**

Add `VendorID`, `ProductID` and `DisplayName` to `ifdVendorID`, `ifdProductID` and `ifdFriendlyName` sections.

**3. Restart `pcscd`**

```shell
sudo systemctl restart pcscd
```

**4. Rerun `sudo pcsc_scan -r` to check if your Ledger device is recognized**

### 5. Disable built-in `ccid` support for `scdaemon`

```shell
# ensure .gnupg directory exists
gpg -k
# create scdaemon.conf file
echo "disable-ccid" >> ~/.gnupg/scdaemon.conf
```

## Check if everything is ready

```shell
gpg --card-status
```

This should show the information of your Ledger device as a OpenPGP Card.

You can do future operations like `gpg --edit-key` to manage your OpenPGP keys.

See also: https://support.ledger.com/article/115005200649-zd

## Troubleshooting

### 1. git: gpg failed to sign the data

When you use `git commit -S` to sign your commit, you may encounter the error `gpg: signing failed: No such device`.

Maybe you need to set `GPG_TTY` environment variable to the correct value.

Add these lines to your shell profile file (e.g. `~/.bashrc`, `~/.zshrc`)

```shell
export GPG_TTY=$(tty)
```
