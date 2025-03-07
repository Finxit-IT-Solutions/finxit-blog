---
title: 'The ultimate Git SSH guide'
date: '2025-03-07'
draft: false
---

Developer tooling is _the_ deciding factor for developer productivity. With many moving parts, however, there is also a lot of complexity involved. Today, I aim to share my best practices regarding how to use git with SSH authentication.

If you are asking, why not simply use authentication via `https`, it's because it works somewhat okay with Windows Credential Manager, but absolutely sucks on other platforms. With SSH on the other hand, you can setup a secure, seamless to use authentication method, that you might already be familiar with, if you are used to connecting to Linux based servers. No more annoying passwords. In my setup, we won't be using any extra dedicated hardware, besides the TPM module that probably gets used by Windows or macOS.

## The agony of choice

There are a lot of different key types to choose from. Nowadays, you usually want to aim for `ed25519` (elliptic curves). It is very secure and offers a small key size, therefore performs fast. If you are using a lovely platform called ~Team Foundation Server~, no wait ~Visual Studio Team Services~ – ah damn you Microsoft – Azure DevOps, then you will need to resort to the older `RSA` key type. Make sure you use a size of at least `4096` for `RSA` keys to be on the safe side. Don't waste your time tricking Azure DevOps it into using an `ECDSA` key, invest that time instead in migrating to a better platform.

Overall, this means I have two SSH keys. One based on `ed25519`, which I use for everything, and a `RSA` one, which I use for everything that sucks. I reuse both keys on all my different devices.

So in case you don't have an SSH key yet, or you want to generate a new one, because your `1024` bit sized `RSA` key is insecure, you probably want to use this command:
```
ssh-keygen -t ed25519 -C "your-email@example.com"
```

Enter a secure passphrase to protect your key!

By default, generated keys are usually saved in your user folder under `.ssh`, e.g. `~/.ssh/id_ed25519` and `~/.ssh/id_ed25519.pub`. The `.pub`lic key is the one that needs to be shared to, where you want to use your (private) key for authentication.

## No AI agent needed

Using a key for authentication (that was protected with a passphrase) is quite cumbersome, because you need to enter your password each and _every_ time you want to use it. However, don't make the mistake to go without a passphrase. That works great till the day you accidentally leak your unprotected key by sharing a virtual machine with someone 🙃.

Use an SSH agent instead!

### Windows

On Windows, the PuTTY stack was the dominant one for doing SSH authentication for a long time. Fortunately, you don't need that nowadays anymore as SSH support is baked directly into Windows.

If you still want/need support for PuTTY, be aware that PuTTY expects the public and private key combined into one file. PuTTYgen can help you with that.

First enable the OpenSSH Authentication Agent services on Windows:

```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

Then use `ssh-add <path>` to add your key to the Windows OpenSSH agent:
```
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

With `ssh-add -L` you can verify, that your key in fact was added.

Now you can use this key without ever entering a password again. Microsoft recommends that you backup your private key to a secure location and remove the file from your device.

### macOS

On a macOS system simply add your key with `ssh-add --apple-use-keychain <path>`, enter your passphrase, and you should be good to go.

### Linux

This one is probably the most annoying one to setup and use, besides SSH having its roots in the UNIX world. Might this be the reason why we never had a successful year of the Linux desktop?

I typically work with Ubuntu virtual machines, so your mileage may vary on other systems. Also, I had many different versions of this in the past, based on different Ubuntu versions.

I added the following lines to my `.bashrc`:
```bash
if [ ! -S ~/.ssh/ssh_auth_sock ]; then
  eval `ssh-agent`
  ln -sf "$SSH_AUTH_SOCK" ~/.ssh/ssh_auth_sock
fi
export SSH_AUTH_SOCK=~/.ssh/ssh_auth_sock
ssh-add -l > /dev/null || ssh-add
```

This little code then prompts you the first time you open a terminal session for the passphrase of your key. If you reboot your machine, you will need to reenter the password.

## Goodbye https authentication

Now that you have setup your SSH authentication, it is time to use it. All the git repos won't automatically switch however, so the URLs for the remote needs to be updated.

This can be done with `git remote set-url origin <url>`. Verify your remote URLs with `git remote -v`.

## Yes, I did this

A best practice for Git is signing commits. It basically confirms, that yes, **you**, have actually created this commit. Azure DevOps again doesn't support this (read as: it doesn't show nor verify it), but it also doesn't do any harm. Other services like GitHub actually verify the signing of commits and display a "Verified" badge next to the commit, if configured.

Recent Git versions not only support GPG for signing, but also SSH keys. So let's do that!

Add the following configurations to your `.gitconfig`:
- `user.signingkey` → `<path to the key>`
- `gpg.format` → `ssh`
- `commit.gpgsign` → `true` (automatically sign every commit)
- `gpg.ssh.allowedSignersFile` → path to a file with the public verified signing keys (Optional, but recommended, if you want to verify commits locally; usually `<userprofile>/.ssh/allowed_signers`)

Now all commits automatically get signed.

## Bonus: Bridging Windows OpenSSH with the PuTTY stack

While I recommend to use new OpenSSH tooling, which is included since Windows 10, there is some awesome software out there, that might still depend on the PuTTY stack. For me that's [WinSCP](https://winscp.net/eng/index.php).

Fortunately, there is a small open source project, that solves this problem: [winssh-pageant](https://github.com/ndbeals/winssh-pageant) (named after the PuTTY pageant).
There are quite some steps involved for the installation (f. e. a different version of Windows OpenSSH), so make sure to follow the guide in the readme file thoroughly. 

For me, I had to configure two more things, to make everything work with the separate installation of Windows OpenSSH (that lives in `C:\Program Files\OpenSSH` vs the one that is in `C:\Windows\System32`):
- Set the `GIT_SSH` environment variable to `C:\Program Files\OpenSSH\ssh.exe`
- Set `gpg.ssh.program` in your `.gitconfig` to `C:\\Program Files\\OpenSSH\\ssh-keygen.exe`

As we are already talking about WinSCP, you might also want to replace the 'Open Console' button, that normally opens PuTTY with Windows Terminal. WinSCP kindly provides a [guide](https://winscp.net/eng/docs/integration_putty#wt) for that in their documentation.

## Some final words

Using an SSH key for authentication is quite straight forward, but getting everything setup including signing, bridging (on Windows) etc. might take some time. However, done once, there is basically no additional maintenance. 

All in all, you should have something like this in our `.gitconfig` on Windows (besides other configuration):
```ini
[user]
    name = First name Last name
    email = your-email@example.com
    signingkey = ~/.ssh/id_ed25519.pub
[gpg]
    format = ssh
[gpg "ssh"]
    allowedSignersFile = ~/.ssh/allowed_signers
    program = C:\\Program Files\\OpenSSH\\ssh-keygen.exe
[commit]
    gpgsign = true
```

Last but not least, let's talk about security and key rotation. Because an SSH key is basically just a password on steroids, and therefore it would be advised to rotate the authentication key from time to time. The main reason to rotate a key becomes especially relevant, when the used key type is no longer considered secure (like `RSA` with a bit size of `1024`).

In the case of rotating authentication keys, it would be advised to have different keys for authentication and signing, to easily rotate the authentication key, as changing the singing key is just pain (for everybody else, as everybody would need to update their `allowed_signers` file, if they want local verification)

Generally speaking: It is a matter of concern how much pain you can endure, if you need to change a key, in case something gets compromised. It is very similar to a password.

## Links
- [GitHub documentation for creating an SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- [Key-based authentication in OpenSSH for Windows](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement)
- [winssh-pageant](https://github.com/ndbeals/winssh-pageant)
- [Comparing SSH Keys - RSA, DSA, ECDSA, or EdDSA?](https://goteleport.com/blog/comparing-ssh-keys/)
