# lpass-add

This is an (unofficial) script which wraps the LastPass CLI [lpass](https://github.com/lastpass/lastpass-cli) to easily add SSH private keys to your `ssh-agent` keychain. In short, it acts as a go-between for `lpass` and `ssh-add` by using ephemeral private key files.

This script is intended as a replacement to storing SSH keys in files on your local filesystem, and it's better because:

* Private keys can be synced across multiple PCs (although SSH best practice is to avoid using the same private key on too many hosts).
* Private keys are not stored on your local filesystem. They exist in LastPass, or in `ssh-agent` memory. Note that an ephemeral key file is created, but it is immediately deleted by the script.
* Private keys can be shared (if absolutely necessary) using LastPass secure sharing.

I built this tool for cases where you have private keys that are moderately important but for historical reasons are not password protected. Note that you should usually password-protect your private keys, in which case you can store the passwords in LastPass instead of the whole key itself. However, sometimes that's just not possible.

Using LastPass adds quite a bit of overhead to a local-filesystem-based key management process. If you aren't familiar with LastPass or the [lpass](https://github.com/lastpass/lastpass-cli) tool, check that out first -- there's a bit of a learning curve.

## Installation

It's just a Bash script. You can install it however you want, but something like this seems pretty popular:

``` bash
curl https://raw.githubusercontent.com/luketurner/lpass-add/master/bin/lpass-add -o /usr/local/bin/lpass-add && chmod +x /usr/local/bin/lpass-add
```

However, `lpass-add` depends on the LastPass CLI tool `lpass`, which must also be installed on your machine. See the [lpass docs](https://github.com/lastpass/lastpass-cli) for more information about that. (Spoiler for OS X users: `brew update && brew install lastpass-cli --with-pinentry`) 

It also depends on `ssh-agent`. You should be able to run the `ssh-add` command on your machine -- if not, you may need to reinstall OpenSSH, or fix whatever you're using. If you don't know how to use SSH, this is not the README for you.

## Usage

The `lpass-add` script is written to have the same basic usage as `ssh-add`, which it calls directly. The script itself is ridiculously short. I recommend reading it, so you see what's going on under the hood. Trust me, there's not a lot in there.

Say you have a LastPass entry called `my-ssh-key` with a SSH private key pasted into the Notes field. Then, to load that SSH key into `ssh-agent`, you just need to do:

``` bash
$ lpass-add my-ssh-key
```

Of course, you must be logged in to `lpass` for this to work. 

### Managing private keys with lpass

The `lpass-add` script does not provide an easy way to add and edit private keys, so users must be familiar with `lpass` and how it's used to create, edit, and view LastPass credentials. Remember that the SSH private key has to be in the Notes field for `lpass-add` to work with the credential.

Consider this example, which adds a new private key to LastPass using `lpass` and then imports it to the local `ssh-agent`:

``` bash
# Create a new private key in LastPass
$ cat | lpass edit --notes my-private-key << EOF
-----BEGIN RSA PRIVATE KEY-----
MYPRIVATEKEYGOESHEREMYPRIVATEKEYGOESHERE
MYPRIVATEKEYGOESHEREMYPRIVATEKEYGOESHERE
MYPRIVATEKEYGOESHEREMYPRIVATEKEYGOESHERE
MYPRIVATEKEYGOESHEREMYPRIVATEKEYGOESHERE
-----END RSA PRIVATE KEY-----
EOF

# Load the key into ssh-agent
$ lpass-add my-private-key

# Now, we can use it to connect to servers
$ ssh my-user@example.com
```

### Organizing Keys In LastPass

I have a practice of putting all my SSH private keys inside an `SSH` folder in my LastPass vault. Within that folder, I create a subfolder for every project/company/whatever that I have SSH keys for. This way, LastPass not only secures my keys, but also helps me organize them similar to a filesystem.

For example, if I make a new private key for testing deployments at work, I might do:

``` bash
$ lpass edit --notes SSH/mycompany/test/deployment
```

Also, I typically store the public keys in LastPass as well, by appending `-pubkey` to the name of the private key. The `lpass-add` script does not interact with these credentials, I just find them handy in case you want to get the public key later.

For example, I might put a public key at:

``` bash
$ lpass edit --notes SSH/mycompany/test/deployment-pubkey
```

I recommend you find an organizational system that works well for you. `lpass-add` doesn't care about anything except the Notes field in the entry, so there is a lot of flexibility for integrating it into any LastPass power-user setup.


---

Copyright (c) 2016 Luke Turner

Released under MIT License (SPDX:MIT)
