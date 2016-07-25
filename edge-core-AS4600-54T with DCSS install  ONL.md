##Install ONL##
I installed ONL on AS4600 with DCSS(Data Center Software System) pre-installed, step 1-7 is the way to get switch from DCSS to ONIE.

The key is to load ONIE, the way may be variable with diffrent system.Here is what I have done.

1.Attach a computer with console cable

2.Poweron switch

3.enter minicom in computer,then see the booting log in serial terminal

4.typein `2` to enter units menu,then choose `13` to reboot

5.when see the hint:`Hit any key to stop autoboot:  0`,hit a key to stop it, then you will see the prompt `Loader=>`

6.(optional) `Loader=> printenv` to check for the ONIE details.

7.`Loader=> run onie_rescue` take you to the ONIE environment.The prompt changes from `Loader=>` to `ONIE:/ #`.

8.copy the install file from computer to switch:
`ONIE:/ # scp username@IPAddress:/path/*.installer  ONL.installer`.

9.`ONIE:/ # sh ONL.installer`.

10.The resulting installation has a default account `root` with a default password `onl`

## Connect to controller ##
1.get closed OFDPA for edge-core-AS4600-54T from here `https://github.com/opennetworklinux/ofdpa-2.0-closed-accton`(the latest commit have deleted it, so find it in previous commits) 

2.`mknod /dev/linux-kernel-bde c 127 0`

3.`mknod /dev/linux-user-bde c 126 0`

4.`insmod linux-kernel-bde_powerpc_k3.9.ko`

5.`insmod linux-user-bde_powerpc_k3.9.ko`

6.`cp librpc_client_powerpc.so.1 /usr/lib/librpc_client.so.1`

7.`cd /usr/lib`

8.`ln -sf librpc_client.so.1 librpc_client.so`

9.`./ofdpa-2.0-ea2-powerpc-as4600-54t-r0` to run OFDPA

10.`./brcm-indigo-ofdpa-ofagent-powerpc --controller <controller ip>` to connect to controller
