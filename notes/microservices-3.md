

### What are 2 phased commits?

We have Order service and an inventory service. 
* When a client places an order, they interact with Order service.
* Before the order is placed, the order service will have to communicate with inventory service and then get latest inventory for item and then reduce it by the required quantity.
* A transaction is a series of steps to be taken to make a change in DB.
* In the first phase, we'll check if all the services or participants are ready to make the changes. Take lock on the lamp that needs to be bought in inventory service and order service is ready for the person to place the order.
* In the second phase, we actually go with placing order and reduce the inventory.

In the first phase, we check the feasibility of each of the participating services to perform that transaction.

The Second phase is, once the every participating service has taken the requisite lock, we can commit the transaction.

>Why is this lock required in the first phase?
>
>There can be other users who might want to buy the same item, and if at same time they try to reserve the item one might not be able to add to cart. 

In case if any of the participants fails to commit the transaction, then there can be n number of retying, even after that if we fail.
Then all the participants have to roll back, and none can commit if one fails.

The bigger locks (instead of rows we take on table) we take the worst is our optimization.

**2PC is considered as an antipattern (commonly used pattern which should not be used). Why?**

* It makes the user experience bad
* Makes the services slower cause the lock can be acquired for infinite time. We are taking aggressive locks here.

### Distributed Transactions—SAGA

SAGA means a story, a beautiful long tale.

Assume a transaction can eb broken into multiple smaller transactions over multiple services.
Run each of these transactions over each service.

Image the transaction as a distributed transaction.

In case of failure, we need to think of the fallback/rollback case.

**Orchestration in case of SAGA**

One service acts as an orchestrator service, this service will take the responsibility to make sure all distributed transaction pieces are completed.

In case one distributed transaction fails, the orchestrator service ensures that every other distributed transaction should be rolledback.

Example, UPI payment ecosystem NPCI is the orchestrator. If money is debited from one account but credit has failed to another account then NPCI is responsible to initiate rollback in first service were money was debited.



