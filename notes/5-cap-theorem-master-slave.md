# CAP Theorem

Formulated by google engineer.

* C - Consistency
* A - Availability
* P - Partition Tolerance

> **Consistency**
> 
> On every read the distributed system should return the latest write value or should return an error.
> Never return a stale value.

Consistency in ACID means there will be no violation of the set rules.

> **Availability**
> 
> The system is always available to take a request/answer you request even though the answer might be stale.

> **Partition**
> 
> In a cluster of machines, two machines talk to each other over network and given network is unreliable if for some time,
> 2 machines are not able to talk to each other, we call that event as network partition.
> 
>
> There can be n number of machines in a cluster. 
> Any two machines in a cluster communicate by packet transfer.
> They are unable to communicate means that packets are lost.


## **CAP Theorem**
> 
> Whenever a system faces network partition, we have to prioritize either strong consistency or full availability.

If we guarantee that if network partition never happens then we will never have to choose between consistency or availability.

* banking: C > A
* instagram likes: A > C

>**Spanner Database - Google**
> * proprietary hardware
> * super redundant network cables - veru high quality machines, also the connection will be high quality and 2 machines will be connected via multiple cables
>
> In case ever if network partition happens then spanner will prioritize consistency over availability.

### Consistency Spectrum 

> **Strong Consistency**
> 
> Every read will give latest write.

>**Eventual Consistency**
> 
> Even though we're right now reading a stale value, but it is guaranteed that ultimately after sometime consistency will be achieved.
> And once consistency is achieved it is maintained as well.

>**Weak consistency**
> 
> We may or may not ever see the value of the latest write.
> 
> Something may be lost forever. 

### Availability Spectrum

> **Triple 9**
> 
> 99.9 % of time available. meaning in a year approx downtime os 9hr.

> **Four 9**
> 
> 99.99% of time a system is available, Approx downtime is 1hr/year

## PACELC Theorem

* P - Partition
* A - Availability
* C - Consistency
* E - Even otherwise
* L - Latency
* C - Consistency

Latency will always be a trade-off for Consistency. In case we need lower latency, we need to compromise on consistency. Highly consistent systems usually have slight higher latency.

## Replication

Sharding is distribution of data among the machines.

Replication is copying of data to different machines, to avoid single point of failure. To ensure availability we sue replication.

> **Why do we need Replication?**
> 
> We need replication to achieve fault tolerance.
> 
> **What is fault tolerance?**
>
> If one of the machines has a fault, would you want the data to be lost? (No).
> Unless and until the fault is fixed do you want the data on the machine to be inaccessible? (No).
> 
> Thus fault tolerance ensures the system is able to continue operation without interruption even when one or more machines fails.

#### 1. Master-Salve 
Master keeps the original data whereas the machines that maintain a copy are called slave machines. 

Beyond redundancy and fault tolerance also allows for scaling up the reads. On Master we can write the data and read from the slaves.

>In master-slave replication whenever the master dies, there is a leader election protocol which elects one the slaves as the new master. This is called master election protocol.
> 
>If after sometime, if the dead master comes back up they join as a slave.

**Master-Slave Replication Strategies**
1. Master-Slave is highly available but has weak consistency.
> Step 1. Master takes the write and immediately acknowledges OK (without copying to slaves)
> Step 2. Try to copy to slaves
> 
> In this case if master dies before the copy happens to the slaves then the write operation will be lost.
> 
> ex. Splunk - weak consistency algorithm (for logging)

2. Master-Slave is highly available but has eventual consistency.
> Master takes the write and waits for few of slaves to write it before it acknowledges with OK. This ensures that write is never lost.
>
> This is called Quorum or gossip protocol. Salves can write from master of gossip with other slaves and get the write operation.

3. Master-Slave setup that compromises availability but gives strong consistency.
> Master takes teh writes and waits for all the slaves to finish write before it acknowledges OK.
> 
> PROS - 
> 1. Strong/immediate consistency - clients can ready for any machine master or any slave
> 
> CONS - 
>1. Highest latency since we are sticking to the strongest consistency.
> 2. Availability will be compromised - Even if one slave is unavailable or having a network partition, we will have to reject the user write request by saying we are unavailable right now.