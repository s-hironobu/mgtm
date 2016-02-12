# mgtm

This is a multi-GTM (global transaction manager) system for  [Postgres-XC](http://postgresxc.wikia.com/wiki/Postgres-XC_Wiki) version 0.9.2. It is provided as a Vagrant box.

**Note:** The multi-GTM system I made is just an **old** feasibility study. It was made in 2011, and can be run only with Postgres-XC version 0.9.2, on 32bit Linux.


## Postgres-XC and multi-GTM system

Postgres-XC is a multi-master database cluster system based on PostgreSQL.
It is composed of a single global transaction manager (GTM) and multiple XC nodes, each of which contains a proxy, a coordinator, and a datanode.  The GTM of Postgres-XC is clearly a single point of failure (SPOF).


The multi-GTM system increases the availability of Postgres-XC to remove a SPOF.
It is written in Java from scratch and is composed of two server programs: mgtm (GTM) and mgtm_proxy.

![Figure 1: ](http://www.interdb.jp/blog/pgsql/img/mgtm-01.png)


When a multi-GTM starts up, a leader is elected from among GTMs.
The leader sends all data received from mgtm_proxies on XC-nodes to the other GTMs using *atomic multicast* protocol and hence all GTMs are always in the same state.
If the leader crashes, a new leader is elected, and then mgtm_proxies reconnect to the new one. Therefore, Postgres-XC can run without being affected by failures of GTMs.


## Requirement

* [Vagrant](https://www.vagrantup.com/) 
* [VirtualBox](https://www.virtualbox.org/)


## Installation

Run `git clone` to get this repository.

```
[your-pc]# git clone https://github.com/s-hironobu/mgtm.git
[your-pc]# cd mgtm
```

Issue `vagrant box add` command to install the Vagrant box from [Atlas](https://atlas.hashicorp.com/s-hironobu/boxes/centos67_32_mgtm4pgxc092).

```
[your-pc]# vagrant box add s-hironobu/centos67_32_mgtm4pgxc092
```

## How to run

#### [1] Start Guest VMs

Issue `vagrant up` command to start five guest VMs: mgtm1, mgtm2, mgtm3, node1, and node2.

```
[your-pc]# vagrant up
```

**Note** If ssh-related error has occurred during creating a VM, destroy the VM and recreate new one.

#### [2] Create terminals

Create five terminals on your Desktop, as shown in the figure below:

![Figure 2: ](http://www.interdb.jp/blog/pgsql/img/mgtm-02.png)

Then, access to each guest VM by using `vagrant ssh` command.

For example, issue the following command on a terminal to access to mgtm1:

```
[your-pc]# vagrant ssh mgtm1
[vagrant@mgtm1 ~]$
```

And so on.

#### [3] Start GTMs

Start all GTMs by executing the `mgtm-start` command on three guest VMs: mgtm1, mgtm2 and mgtm3.

```
[vagrant@mgtm1 ~]$ mgtm-start
```

```
[vagrant@mgtm2 ~]$ mgtm-start
```

```
[vagrant@mgtm3 ~]$ mgtm-start
```

Then, the message below will be shown on the  terminal of mgtm1:

```
[vagrant@mgtm1 ~]$ mgtm-start 
I'm running...
I'm LEADER!!!!

```

#### [4] Start other processes

Start other processes (mgtm_proxy, coordinator, and datanode) by executing the `node-start` command on node1 and node2.

```
[vagrant@node1 ~]$ node-start

I'm ready!!!
```

```
[vagrant@node2 ~]$ node-start

I'm ready!!!
```

*Now, Postgres-XC is ready!*


#### [5] Execute pgbench

Execute `pgbench` command on node1 or node2:

```
[vagrant@node1 ~]$ pgbench
```

You can also issue simple queries. (Issue the `vacuum` command when first accessing after startup.)

```
[vagrant@node1 ~]$ psql -c "vacuum"
VACUUM
[vagrant@node1 ~]$ psql 
psql (8.4.3)
Type "help" for help.

pgbench=# 
```

###### <b>Note</b>: In version 0.9.2 almost function has not been implemented, so you can execute only simple queries (e.g. SELECT, INSERT, DELETE, and UPDATE).


## Simulate GTM crash

Let's terminate the leader and see how the other GTMs perform failover.


(1)  Execute pgbench with `-T` option.


```
[vagrant@node1 ~]$ pgbench -T 30
```

(2) Enter `^C` on the terminal of mgtm1 to terminate the leader process while the pgbench process is running.

```
[vagrant@mgtm1 ~]$ mgtm-start
I'm running...
I'm LEADER!!!!
I'm LEADER!!!!
I'm LEADER!!!!

^C
```

Then, mgtm2 will be a new leader.


```
[vagrant@mgtm2 ~]$ mgtm-start 
I'm running...
I'm LEADER!!!!
```

The figure below illustrates this situation:

![Figure 3: ](http://www.interdb.jp/blog/pgsql/img/mgtm-03.png)


## Terminate GTMs and XC-nodes

Stop all GTMs by entering `^C` on each terminal of mgtm,
and execute the `node-stop` command on the terminals of both node1 and node2.

```
[vagrant@mgtm2 ~]$ mgtm-start 
I'm running...
I'm LEADER!!!!
^C
```

```
[vagrant@mgtm3 ~]$ mgtm-start 
I'm running...
^C
```

```
[vagrant@node1 ~]$ node-stop
```

```
[vagrant@node2 ~]$ node-stop
```

If you want to try again, back to `[3] Start GTMs` in "How to run".

## Shutdown VMs

Issue `vagrant halt`.

```
[your-pc]# vagrant halt
```

---

## Benchmark

The result of benchmarking with pgbench is shown in the graph below:

![G2: ](http://www.interdb.jp/blog/pgsql/img/mgtm-g2.png)

"mgtm x 3" means that three GTMs run, and so on. I also did benchmarking using an original GTM (green line).
