# ssh-control-hours
A stack of procedures for register ssh login/logout activities. There is no code here. Its just a recipe to follow. It used to work well, but there is no warranty. 

## What is the purpose?

The purpose is exclusively personal (read: workaround from a lazy guy to other lazy ones). Everybody knows how hard it is to keep track of daily activities. It becomes much worse when you have multiple customers, projects and so on.

## What does it do?

Basically the tool works sending to a file 2 registers: ssh login and logout.

No matter the login method you are using (single user such as "centos" for example).
It sends an envvar thought the ssh connection. This var contains your id. The remote environment receives it and save to a log file that will be sent to a remote storage when you get out of the server. Quite straight forward 

## How do I make it work here?

On your machine, add the follow value on ~/.ssh/config:

```
SendEnv GETUP_EA
```
* The value can be whatever you want. You will consume it. Make it clear for you.

Add it on your ~/.bashrc:
```
export GETUP_EA=edilson.azevedo
```
Update your env:
```
source  ~/.bashrc
```
On the server side, add the following line on /etc/ssh/sshd_config and restart the ssh server:
```
AcceptEnv LANG LC_* GETUP*
```
Yet on the server, add these lines on ~/.bashrc:
```
getupreg() {
    curl -s --output /dev/null --upload-file "$1" http://transfer.easycloud.guru:8080/$(basename "$1") | tee /dev/null;
}
alias getupreg=getupreg
```
Important notes:
* Change the bashrc file of the user that you are used to use to login on the server, ok?
* The transfer.easycloud.guru is a personal storage service based on a project called transfer.sh (https://github.com/dutchcoders/transfer.sh). Again: you can use what you want!

Last but not least, yet on the server side, add these lines to ~/.profile

```
TIME=$(TZ=America/Sao_Paulo date +%Y%m%d.%H:%M)
RANDOM=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 10 | head -n 1)
LOGG=/tmp/$GETUP_EA-$RANDOM-$HOSTNAME.temp

echo "USER $GETUP_EA LOGGED AT $TIME" >> $LOGG
function onexit { echo "USER $GETUP_EA LOGOUT AT $(TZ=America/Sao_Paulo date +%Y%m%d.%H:%M)" >> $LOGG; getupreg $LOGG ; rm -f $LOGG ; }; trap onexit EXIT
```
Update the env:
```
source  ~/.bashrc && source ~/.profile
```
Done!

Output example:
```
USER edilson.azevedo LOGGED AT 20201203.23:10
USER edilson.azevedo LOGOUT AT 20201204.04:39
```

It can be transitive (if you are using a bastion host or proxy or whatever. The key is keep sending the ssh env across the servers. Just it.


Cheers!
