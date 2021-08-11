# Passwordless SSH Access

To further secure access via SSH it is best practice to utilize SSH keys for logging in.
SSH keys are a matching set of cryptographic keys which are used for authentication. They consist of a public and a private key, you can think of these as the lock (public key) and key (private key) to your house. Just like the lock on your door is publicly available, the public key can be freely shared with anyone without problems. The private key however needs to be kept secret, just as you wouldn't share your housekey.

## How does the authentication work?

Where the analogy kind of breaks down is that to login the client has to supply the public key (lock) to the server instead of the private key. The SSH server compares the public key to a list of public keys, one per line, that is saved in the user's home directory at `~/.ssh/authorized_keys`.
The server then generates a random string and encrypts it using the public key. This encrypted string can only be decrypted with the associated private key. Upon receiving the string, the client decrypts it using the private key and combines the string with a previously negotiated session ID. 
Next, it generates a MD5 hash of this value and transmits it back to the server. The server now makes sure that the MD5 matches with the information it already has (the random strin and the session ID) and if it matches it can conclude that the client has a private key and is allowed to login.

## Generating SSH key pairs

A key pair can be generated using a number of different cryptographic algorithms, for example RSA, DSA or ECDSA. RSA keys are the default.

To generate a RSA key pair we use the `ssh-keygen` utility. Below it is done on a UNIX distro but it works the same on windows.

ssh-keygen will prompt you to choose the location to store the RSA private key. You can choose any location that you have write access to but it is recommended to leave it in the default location. This ensures that your SSH client will find the keys automatically if you are logging in from this computer.

```bash
$ ssh-keygen
Generating public/private key pair.
Enter file in which to save the key (/home/<username/.ssh/id_rsa>):
```

Next you are prompted to choose a passphrase. This phrase is used as additional security in so that you need to know it in order to use the private key. You can choose to leave this blank to not have a passphrase but this is not recommended.

NOTE: If you choose to enter a passphrase nothing will be displayed as you time. This is a security feature so that no one can see your phrase as you enter it.

```bash
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

After setting a passphrase ssh-keygen will print your public key's finger and location. You will even get a "helpful" randomart image of it.

```bash
Your public key has been saved in /home/pi/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:znZKfLsgiosaYat+QNixmGHPWSWJAkVHi++natq9AC8 pi@raspberrypi
The key's randomart image is:
+---[RSA 2048]----+
|ooo.+.o.         |
|.o.+ +.          |
|o==o+            |
|+.o=             |
|+.  .   S        |
|.=..   +         |
|E.+ . o B o      |
|.=.= + + = .     |
|B+*o=.  . o.     |
+----[SHA256]-----+
```

Now you have generated a matching set of keys that can be used for authentication. If you used the default settings the following files will have been created:

- `~/.ssh/id_rsa`: This is your private key. **NEVER SHARE THIS WITH ANYONE**
- `~/.ssh/id_rsa.pub`: This is your public key. Sharing is not an issue and can be shared without consequence.

## Changing the passphrase for a private key

You can change and even remove the passphrase of a private key, provided that you know the original phrase of course.
To do so you simply use `ssh-keygen` with a flag. Specify the key you want to edit, enter the old passphrase and change to what you want, including blank for nothing.

```bash
ssh-keygen -p
Enter file in which the key is (/home/<username>/.ssh/id_rsa):
Enter old passphrase: 
Key has comment '<username>@<hostname>'
Enter new passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved with the new passphrase.
```

## Displaying the SSH key fingerprint

Why would you want to do that? Simple, each SSH key pair share a single cryptographic "fingerprint" that can be used to identify the keys. This fingerprint is unique for each pair.

Again, we use the `ssh-keygen` utility with a flag and specify which SSH key's fingerprint you want to display. It will also display the key length (2048 in the code block below) and the cryptographic algorithm (RSA in this case).

```bash
ssh-keygen -l
Enter file in which the key is (/home/pi/.ssh/id_rsa): 
2048 SHA256:znZKfLsgiosaYat+QNixmGHPWSWJAkVHi++natq9AC8 <username>@<hostname> (RSA)
```

## Copying the SSH keys

When you have a pair of SSH keys you need to copy them to servers and clients, as appropriate, to actually use them. Most of the time you will only need to copy your public key to your user on the servers you want to connect to but sometimes you want/need to copy the public key to another client you are using.
The best way of distributing the public keys in a larger environment would be to use some kind of configuration tool like Ansible, Chef or Puppet.

Depending on your client's OS you have to go about it differently.

After distributing the keys you should be able to login without a password with the simple `ssh`command:

```bash
ssh <username@remote_host>
```

### UNIX

For your general UNIX distro you will use the `ssh-copy-id` utility. It is by far the easiest to use, just run the command below and use your password to login. This will copy the default key (`~/.ssh/id_rsa.pub`) to the target host and append it to the user account's `~/.ssh/authorized_keys` file.

```bash
ssh-copy-id <username>@<remote_host>
```

You can now login to that account without a password:

```bash
ssh <username>@<remote_host>
```

If you don't have access to the `ssh-copy-id` utility you will need to output the contents of the **public** key and pipe it into the ssh command. The command below does exactly that as well as ensuring that the `~/.ssh` folder is present.

```bash
cat ~/.ssh/id_rsa.pub | ssh username@remote_host "mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys && cat >> ~/.ssh/authorized_keys"
```

### Windows 10

In the current implementation of Windows 10's OpenSSH client there is no equivalent of the `ssh-copy-id` but we can replicate it with the following one-liner that I've completely ripped from Christopher Hart's [guide](https://www.chrisjhart.com/Windows-10-ssh-copy-id/ "Windows 10 OpenSSH Equivalent of "ssh-copy-id"") on his website.

```powershell
type $env:USERPROFILE\.ssh\id_rsa.pub | ssh <username>@<remote_host> "mkdir -p ~/.ssh && cat >> ..ssh/authorized_keys"
```

## Disabling password authentication

When you've created, distributed and tested the SSH keys it is recommended to disable the standard password authentication. This is to increase the security and it shouldn't be a problem since you've already setup the SSH keys.

To do this you need to change the configuration of the SSH server. This requires root/sudo privileges:

```bash
sudo nano /etc/ssh/sshd_config
```

Inside the file, look for the `PasswordAuthentication` directive, make sure it is uncommented and set it to `no`.

```bash
PasswordAuthentication no
```

After editing the file you will need to restart the ssh service.