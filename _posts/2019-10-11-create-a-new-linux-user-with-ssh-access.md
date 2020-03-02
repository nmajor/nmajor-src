---
layout: post
title: Create a New Linux User With SSH Access
hero: ''
tags:
- linux
- ssh
date: 2019-10-11T23:00:00.000+00:00

---
This is just a personal reference.

Generate the key for the user. This should be done on the local machine:

    ssh-keygen -t rsa -b 4096 -o -a 100

Create a new user on the remote machine. The `-m` adds a deuser fault user home directory.

    useradd -m new-user

{: .lead}  
<!–-break-–>

Switch to the new user:

    sudo su new-user

Make a `.ssh` directory for the user:

    mkdir ~/.ssh

Make an `authorized_keys` file.:

    touch ~/.ssh/authorized_keys

Set the right permissions:

    chmod 700 ~/.ssh
    chmod 600 ~/.ssh/authorized_keys

Add the public key you created to the `authorized_keys` file.

### Resources:

* [https://security.stackexchange.com/questions/143442/what-are-ssh-keygen-best-practices](https://security.stackexchange.com/questions/143442/what-are-ssh-keygen-best-practices "https://security.stackexchange.com/questions/143442/what-are-ssh-keygen-best-practices")
* [https://help.ubuntu.com/community/SSH/OpenSSH/Keys](https://help.ubuntu.com/community/SSH/OpenSSH/Keys "https://help.ubuntu.com/community/SSH/OpenSSH/Keys")