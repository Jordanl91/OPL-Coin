
<p align="center"> 
<img src="https://opc.site/assets/logo-2.png">
</p>

# Open Platform - Masternodes
OPL (Open Platform) Masternode Guide v0.1.0
OPL Masternode Setup and Maintenance Guide

## Requirements
```
•3000 OPL
•Linux VPS (best ubuntu 16.04)
•OPEN PORT: 5567
```

While it is possible to run masternodes (MN) both in a local wallet as well as in a separate instance on Windows this guide describes the most popular way to set up a MN using a local Windows/Linux qt wallet and a Linux VPS server running a MN instance.

What you will need:

-A qt wallet with at least 3000 coins  
-A VPS instance running Linux, this setup is using Ubuntu 16.04 64-bit.


[Creating MN keys for a VPS instance in your local qt wallet]
=============================================================

1. Start qt wallet. Go to Settings→ Options→ Wallet and check “Enable coin control features” and “Show Masternodes Tab”. You will need to restart the wallet for these to show up.
2. Create a new receiving address. Open menu File→ Receiving addressess… Click “+” button and enter a name for the address, for example mn1.
3. Send exactly 3000 coins to this mn1 address When you send the coins make sure you send the correct address you created above, Verify that "Subtract fee from amount" is NOT checked. Wait for 15 confirmations of this transaction.
4. To check the confirmations of the transaction go to Transactions → right Click "Show Transaction Details" or Hover ove the time clock in the far right of the transaction.
5. Open a debug window via menu Tools→ Debug Console.
6. Execute `masternode genkey` command. This will output your MN priv key, for example: 
`92TPhvQjKd5vMiBcwbRpq3g4CnPVGUAZGrorZJPNJoohgCu9QkF`. Save it in a notepad for later use.
7. Execute `masternode outputs` command. This will output TX and output pairs of numbers, for example:

       {
        “a9b31238d062ccb5f4b1eb6c3041d369cc014f5e6df38d2d303d791acd4302f2”: “0”
       }

Save both of these numbers.   
8. Open the masternode.conf file via menu Tools→ Open Masternode Configuration File. Without any blank lines type in a space-delimited single line:

       mn1 YOUR_VPS_IP:5567 YOURPRIVKEY TX_OUTPUT TX_ID
For example:

    mn1 45.76.250.89:5567 92TPhvQjKd5vMiBcwbRpq3g4CnPVGUAZGrorZJPNJoohgCu9QkF a9b31238d062ccb5f4b1eb6c3041d369cc014f5e6df28d2d303d791acd4302f2 0

9. Restart the wallet and go to the “Masternodes” tab. There in the tab “My Masternodes” you should see the entry of your masternode with the status “MISSING”.
10. It is useful to lock the account holding the MN coins so that it would not be accidentally spent. To do this, if you have not done this yet go to the menu Settings→Options, choose tab Wallet, check the box “Enable coin control features”, then restart the wallet. Go to the Send tab, click “Inputs”, select “List mode”, select the line with your MN and 1000 coins in it, right click on it and select “Lock unspent”. The line should be grayed out now with a lock icon on it. To unlock chose “Unlock unspent”.


[Setting up a VPS]
====================================================

Each MN requires a separate IP address so you would either need a different VPS per each MN or have more than one IP address per VPS and use “-datadir=YOURDATADIR” option to separate MN instances, whichever is cheaper, however having a separate instance has also advantages of more hardware resources available and higher reliability. Recommended node hardware includes 1GB RAM, single core CPU is sufficient, and at least 20 GB hard drive. Such node pricing starts at around USD $3-5/month. 

Popular VPS providers are:

Digital Ocean https://m.do.co/c/0a611163bbe8

https://www.vultr.com/

https://www.woothosting.com/

https://www.time4vps.eu/


You can interact with a VPS via ssh terminal, the most popular app for Windows is PuTTY

http://www.putty.org/

If you are setting a VPS on vultr.com from scratch you can do the following:

Register on the site, login and pay $5 or more.
Go to Servers tab on the left. Click on the “+” button in the top left corner with the tooltip “Deploy New Server”.
1. Select any location 
2. Server type = Ubuntu 16.04 
3. Server size = $5/mo (1 core, 1GB memory)
4. Pick server hostname
Once the server is running in about 5-10 min click on the “...” on the right of the server line and select “Server Details”. What you need from this page is IP Address, Username=root, Password=…
If you are on windows download and install Putty, if you are on linux you don’t need this section :)
Run Putty, enter server IP and connect, clicking “yes” to save the new ssh key. Enter username=root and password from the previously saved details.

5. Exit the putty terminal by typing “exit” or closing the window. Connect again but now user your new username and password. Now to execute any commands that require admin priviledges you need to use prefix “sudo”.
6. Digital Ocean & Vultr already has the popular firewall ufw installed, on other distros or providers you may need to install it. Open ports 22 for ssh and 5567 for the masternode P2P network. Then enable the firewall:

        sudo ufw allow 22
        sudo ufw allow 5567/tcp
        sudo ufw enable

To check current rules on an inactive ufw:

    sudo ufw show added

To check current rules on an active ufw:

    sudo ufw status

7. You can now proceed with the rest of the guide specific to installation of the masternode. Very useful commands and tools you will need are:
```
`ls` – list files in the current directory
`ls -la` – list files in the current directory including hidden files 
`pwd` – get current directory
`~` – stands for your home directory, typically /home/YOUR_USERNAME
`cd NEW_DIRECTORY`– change directory
`pushd NEW_DIRECTORY` – change directory and push the old one on the stack
`popd` – change to the previously used directory and pop it from the stack
`screen` – multiple terminals in one
`screen -R` – reconnect to previous screen session after a new login via putty.
```
[Installation of Dependencies]
===============================================

All installation commands require you either being a root or prepending them with sudo. First you need to update Ubuntu 16.04 distro via executing these 3 commands:
```
    apt-get update
    apt-get upgrade
    apt-get dist-upgrade
```
The libraries you need to install: required: libssl, libboost, libevent, miniupnpc, libdb4.8 optional: libzmq3, libminiupnpc Editor: nano (or vim/emacs if you prefer)

```
apt -y install libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-program-options-dev libboost-test-dev libboost-thread-dev libzmq3-dev libminiupnpc-dev libevent-dev software-properties-common unzip ufw fail2ban
add-apt-repository -y ppa:bitcoin/bitcoin
apt update
apt -y install libdb4.8-dev libdb4.8++-dev
```

[VPS node configuration]
==============================================

Download and extract linux binaries:
```
wget https://github.com/opl-coin/opl.coin/releases/download/0.12.3.1/opl.0.12.3.1.linux.x64.zip && unzip -j opl.0.12.3.1.linux.x64.zip -d /usr/local/bin/ && chmod +x /usr/local/bin/opl*
rm -rf opl.0.12.3.1.linux.x64.zip
```	
You should have now the daemon opld and wallet opl-cli files in `/usr/local/bin/` directory. This will allow you to start the wallet in any directory - Start the daemon:
```
    opld --daemon
```
You should see the output: OPL Core server starting

Now stop the server:
```
    opl-cli stop
```
You should see the output: OPL Core server stopping What this should have accomplished is creating a .muncore directory in your home directory and populating it with the config files so that you would not need to create them yourself. Go into .muncore directory:
```
    cd ~/.oplcore
```
You will need to edit 2 files : opl.conf and masternode.conf with nano or any other text editor:
```
    nano opl.conf
    nano masternode.conf
```
In opl.conf you need to create unique user name, user password, masternode priv key (created in the qt local wallet step):
```
   	listen=1
	server=1
	daemon=1
	rpcuser=opluser
	rpcpassword=oplpassword
	rpcallowip=127.0.0.1
	maxconnections=256
	externalip=<YOUR_VPS_IP_ADDRESS>:5567
	masternodeprivkey=<MASTERNODE_PRIVATE_KEY>
	masternode=1
```
In masternode.conf file you need to copy/paste the line from the masternodes.conf file in the qt local wallet:
```
    mn1 YOUR_VPS_IP:12548 YOUR_MASTERNODE_PRIV_KEY TX_OUTPUT TX_ID
```
For example:
```
    mn1 45.76.250.89:5567 92TPhvQjKd5vMiBcwbRpq3g4CnPVGUAZGrorZJPNJoohgCu9QkF a9b31238d062ccb5f4b1eb6c3041d369cc014f5e6df28d2d303d791acd4302f2 0
```
Now you can start the daemon again. Start the daemon:
```
    opld --daemon
```
You should see the output: OPL Core server starting

Let’s observe the node synchronization process. Execute:
```
    opl-cli getinfo
```
The output should look similar to:
```
{
  "version": 120301,
  "protocolversion": 70208,
  "walletversion": 61000,
  "balance": 0.00000000,
  "privatesend_balance": 0.00000000,
  "blocks": 4145,
  "timeoffset": 0,
  "connections": 1,
  "proxy": "",
  "difficulty": 161.3727828863838,
  "testnet": false,
  "keypoololdest": 1526346255,
  "keypoolsize": 999,
  "paytxfee": 0.00000000,
  "relayfee": 0.00001000,
  "errors": ""
}
```

We are looking for the block count to be positive and eventually matching the number of blocks indicated by the local wallet and block explorer.

More checking:
```
    opl-cli mnsync status
```
Should produce an output similar to:
```
    {
     "AssetID": 999,
     "AssetName": "MASTERNODE_SYNC_FINISHED", 
     "AssetStartTime": 1514425867,
     "Attempt": 0,
     "IsBlockchainSynced": true,
     "IsMasternodeListSynced": true,
     "IsWinnersListSynced": true,
     "IsSynced": true,
     "IsFailed": false
    }
```
Periodically running the same command you will be able to see as different phases of synchronization complete making blockchain, MN list, MN winners list synchronized one by one. 

Now you can check the masternode status:
```
    opl-cli masternode status
```
The output from an uninitialized MN will be similar to:
```
    { 
     "outpoint": "0000000000000000000000000000000000000000000000000000000000000000-4294967295",
     "service": "45.76.250.89:5567",
     "status": "Not capable masternode: Masternode not in masternode list"
    }
```

[Node start]
=========================================

The simplest way to start the masternode is from the local qt wallet. Go to your qt wallet “Masternodes” tab. Go there, switch to the tab “My Masternodes”, select the line with your MN and click the button “Start alias”, or right click on the line and use the context pop-up menu. Alternatively when starting the node(s) for the first time you can click the button “Start MISSING” to start all nodes that currently have the status “MISSING”. If you have some already enabled nodes and want to start a new one do not click the button “Start all” because this will restart the already enabled nodes and place them at the end of the paying queue. The status should change to “PRE_ENABLED” and some time later to “ENABLED” (varies, allow for up to 30 minutes). 

Check the masternode status on the VPS:
```
    opl-cli masternode status
```
The output from an uninitialized MN will be similar to:
```
       { 
        "outpoint": "112f474f1e9701bfa424f5837dda3c6b3ae2454d44cabc593f299879fd790527-1",
        "service": "45.76.250.89:5567",
        "payee": "MbhkCopfkW7HVycCw7NkY7SAdFK9uVmjSr",
        "status": "Masternode successfully started"
       }
```
If the masternode appears healthy but you are worried about payments not being when expected you can check if your node address is on the list of masternode winners from either the cli wallet or the debug window:
```
       opl-cli masternode winners
```
The output should have all block assignments to MNs that will be paid from these blocks chosen by consensus vote by all MNs, similar to this snapshot from the testnet:
```
       { 
        "1652": "yd6EWWKDqfcHCCWxVYPuNV7fmgRrSNZra7:10",
        "1653": "yQyKNbW3mUcS21xGXpab1Da3Whs64YEprJ:10",
        "1654": "yMitxDKhpRbFjLC7ss73Cs5oqN3ncPeERG:10",
        "1655": "yMitxDKhpRbFjLC7ss73Cs5oqN3ncPeERG:10",
        "1656": "yhMXFAHb1hzipdnWqeuqiEyGce85jPDpQJ:10",
        "1657": "yhMXFAHb1hzipdnWqeuqiEyGce85jPDpQJ:10",
        "1658": "yizneqYKemPyLwW31c7daHZbBq5EPzrgbo:10",
        "1659": "ybhkCopfkW7HVycCw7NkY7SAdFK9uVmjSr:10",
        "1660": "ybhkCopfkW7HVycCw7NkY7SAdFK9uVmjSr:10",
        "1661": "yT26jdEqqnVDJBwtHmTLSaDbJbV8PfyjsQ:10",
        "1662": "yT26jdEqqnVDJBwtHmTLSaDbJbV8PfyjsQ:10",
        "1663": "yTJsi4beBctD1k7EPqmWhW63dA1Wkmmaii:10",
        "1664": "yTJsi4beBctD1k7EPqmWhW63dA1Wkmmaii:10",
        "1665": "ygm3LiDvvy5cqDjHrhb7CMWteQEt5Nyx1c:10",
        "1666": "ygm3LiDvvy5cqDjHrhb7CMWteQEt5Nyx1c:10",
        "1667": "yVY8cgAuj2boJ3rM4S1g7QQgc8ebLC7TkM:10",
        "1668": "yVY8cgAuj2boJ3rM4S1g7QQgc8ebLC7TkM:10",
        "1669": "yd6EWWKDqfcHCCWxVYPuNV7fmgRrSNZra7:10",
        "1670": "yd6EWWKDqfcHCCWxVYPuNV7fmgRrSNZra7:10",
        "1671": "yQyKNbW3mUcS21xGXpab1Da3Whs64YEprJ:10",
        "1672": "yQyKNbW3mUcS21xGXpab1Da3Whs64YEprJ:10",
        "1673": "Unknown",
        "1674": "Unknown",
        "1675": "Unknown",
        "1676": "Unknown",
        "1677": "Unknown",
        "1678": "Unknown",
        "1679": "Unknown",
        "1680": "Unknown",
        "1681": "Unknown"
       }
```

On the mainnet you should see a series of upcoming 20 blocks all assigned to MNs in roughly a reverse chronological order of when they got paid the last time.

[Running wallet automagically after Reboot]
==========================================

Create a Crontab
```
crontab
2. /bin/nano        <---- easiest
```
Choose number 2 nano to edit, enter the command below and save (ctrl+o, [Enter],ctrl+x). 
```
@reboot /usr/local/bin/opld -daemon
```

### Did this help you? Please donate OPL to **Vz16PcurfENgodexZhpqRufEk5NmUt3Fhv** Help me, help others!
