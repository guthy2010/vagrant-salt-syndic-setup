# Salt Vagrant Development Setup

This is a setup for developing and testing local states with Salt, heavily inspired by https://github.com/UtahDave/salt-vagrant-demo.

## Requirements

* Vagrant is (of course) required
* Virtualbox (default provider for Vagrant)
* A local installation of Salt is also required for the dynamic key generation (`brew install saltstack`, `apt-get install salt-master`, `yum install salt-master`)

## Environment details

#### This environment includes 3 masters and 2 minions:
Master        | Minion(s)
------------- | -------------
mastermaster  | Can talk to all minions
devmaster     | devmaster, devminion1
prodmaster    | prodmaster, prodminion1

All keys are generated dynamically when the master is first provisioned. 
Salt states should be added from files/salt and will be mounted dynamically into the master. 
A similar process happens for salt pillars from files/pillar. 
A highstate will be triggered for all boxes during the Salt provisioning process through Vagrant.

## Usage

#### Bringing up the environment:

```ShellSession
vagrant plugin install vagrant-host-shell
vagrant plugin install vagrant-vbguest
vagrant up
```

#### From the master of masters perspective:
```ShellSession
vagrant ssh mastermaster
root@mastermaster:/home/vagrant# salt '*' test.ping
mastermaster:
    True
prodminion1:
    True
devminion1:
    True
prodmaster:
    True
devmaster:
    True
```
#### From the dev sub master perspective:
```ShellSession
vagrant ssh devmaster
root@devmaster:/home/vagrant# salt '*' test.ping
devminion1:
    True
devmaster:
    True
```

#### Example salt state that differs for each environment:
```ShellSession
vagrant ssh mastermaster
root@mastermaster:/home/vagrant# salt '*minion1' state.sls hello_world
devminion1:
----------
          ID: run hello world
    Function: cmd.run
        Name: echo hello from DEV
      Result: True
     Comment: Command "echo hello from DEV" run
     Started: 15:17:58.358782
    Duration: 6.742 ms
     Changes:
              ----------
              pid:
                  24457
              retcode:
                  0
              stderr:
              stdout:
                  hello from DEV

Summary for devminion1
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:   6.742 ms
prodminion1:
----------
          ID: run hello world
    Function: cmd.run
        Name: echo hello from PROD
      Result: True
     Comment: Command "echo hello from PROD" run
     Started: 15:17:58.358410
    Duration: 6.524 ms
     Changes:
              ----------
              pid:
                  24354
              retcode:
                  0
              stderr:
              stdout:
                  hello from PROD

Summary for prodminion1
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:   6.524 ms
```