# OpenSSH Setup
## Installing the server and setting it to run on boot.
Setting up the OpenSSH on windows is easy enough, first install with Powershell using 

`Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0`. 

To start it on boot, 

`Set-Service sshd -Startup-Type Automatic`.

## Key pair generation
If your user is an administrator, create the file 'administrators_authorized_keys' in C:\ProgramData\ssh. Otherwise create 'authorized_keys'. OpenSSH defaults to only reading ~/.ssh/authorized keys, but this can be overriden with the setting 'AuthorizedKeysFile \_\_PROGRAMDATA\_\_/ssh/authorized_keys'. You can generate a 4096 bit rsa key pair with `ssh-keygen -t rsa -b 4096` and place the public key (file extension .pub) in the C:\ProgramData\ssh directory. It is *recommended* to use a passphrase. Copy the contents into the authorized key file corresponding to your user privileges. Each key entry must be on a new line.

Move the private key (no file extension) to the device you wish to access the server from. Set its permissions to 400 as keys with open permissions will be rejected in the key exchange.

# Extra SSH security - C:\ProgramData\ssh\sshd_config
Important security features: set 'PasswordAuth no', 'PermitRootLogin no', PermitEmptyPasswords no'. Using the website ssh-audit.com, you will probably score an F if you leave it at that. I've included my 'sshd_config' file in the repository that scores an A+.

# Connecting
Manually connecting to the remote PC is simple.

`ssh -i <path to private key> <username>@<remote address> -p <port number>`.

We will actually want to also create a local port binding to be able to access Jupyter notebook. Port 8889 will work nicely. The command is

`ssh -L 8889:localhost:8889 -i <path to private key> <username>@<remote address> -p <port number>`.

In order to access a Jupyter notebook instance, you run it on the remote system with

`jupyter notebook --no-browser --port=8889`.

After this, copy the address with the token into your local browser and you should be able to use the notebook remotely. It will look something like `http://localhost:8889/?token=6eabcb9d2a8cd5cd5e7c8ea0125b3592c3f9a53a133f7cf9`

# Automation and Anaconda
If you are running Jupyter straight from Windows without any containers, you can create a Batch script with the contents `jupyter notebook --no-browser <target directory>` and that'll be that. Then your connect.sh script will simply be the command

`ssh -L 8889:localhost:8889 -i <path to private key> <username>@<remote address> -p <port number> -t "<path to script>"`

OR just

`ssh -L 8889:localhost:8889 -i <path to private key> <username>@<remote address> -p <port number> -t "jupyter notebook --no-browser --port=8889 <target directory>"`

However, in order to run jupyter from Anaconda *and* use a custom drive (it defaults to C), there is a shortcut for 'Jupyter Notebook (Anaconda3)' in C:\Users\\<username\>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Anaconda3 (64-bit). Copy the 'Target' field into a Batch script and change the '%USERPROFILE%/' into whatever directory you wish the notebook to launch in. Add '--no-browser --port=89' after the quote.

Having saved the batch script, you can use the same connect.sh script now to automatically connect to the remote pc with immediate Jupyter access.

## End
The purpose of this write-up is to simplify the process I spent the better part of two days going through. [Here is the windows resource for configuring openSSH](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_server_configuration).
