Multi Leader Replication

A natural extension of the single-leader replication model is to allow more than one node to accept writes -> This is known as multi-leader configuration.
The replication still happens the same way: each node that processes a write must forward that data change to all the other nodes. The leader simultaneously acts as a follower
to the other leaders.


# Geographically Distributed Operation

A geographically disributed setup is when you have a database with replicas in several regions (because you can tolerate the failure of an entire region, or perhaps for proximity with your users)

<img width="338" height="212" alt="Screenshot 2026-11-25 at 2 27 48 AM" src="https://github.com/user-attachments/assets/197f351c-1fe6-4e02-8ae3-aee201006a30" />

The photo above shows how between regions, each leader replicates its changes to the leaders in other regions.

  Performance:
      In a multi-leader configuration, every write can be processed in the local region, and then replicated asynchronously to the other regions. Thus, the
      inter-region network delay is hidden from users, which means the perceived performance may be better.

  Tolerance of Regional Outages:
      In multi-leader configuration, each region can continue operating independently of the others, and replication catches up when the offline region comes back online.

  Tolerance of Network Problems:
      A multi-leader configuration with asynchronous replication can tolerate network problems better; during a temporary network interruption, each region's leader can continue independently processing writes

  Consistency
      The biggest downside of multi-leader systems is that the consistency they can achieve is much weaker. For example, you can't guarauntee that a bank account won't go negative or that a username is unique; it's always possible
      for different leaders to process writes that are individually fine (payout out some of the money in an account, registering a particular username) but that violate the constraint when taken together with another write on another leader.

      This is a fundamental limitation of distributed systems.


  Multileader replication is less common than single leader replication, but it's still supported by many databases, including MySQL, Oracle, SQL Server.

    ## Multi-leader Replication Topologies

      A replication topology describes the communication paths along which writes are propagated from one node to another.  If there
      are multiple leaders, various topologies are possible.

  <img width="367" height="136" alt="Screenshot 2026-11-25 at 2 43 33 AM" src="https://github.com/user-attachments/assets/9a9963f4-a842-40f6-b104-f23028c3337b" />

      In circular and star topologies, a write may need to pass through several nodes before it reaches all replicas, and because of this, nodes need to forward data changes they receive from other nodes
      And to prevent infinite replication loops, each node is given a unique identifier, and in the replication log, each write is tagged with the identifiers
      of all the nodes it has passed through.

      When a node receives a data change that is tagged with its own identifier, the data change is ignored, because the node knows that is has already been processed.


      ## Problems With Different Topologies

      If an error occurs in circular and star topologies, it sets back the update of all other laders.
      In all to all topologies, some network links may be faster than others, with the result that some replication messages may "overtake" others, as shown below.


<img width="605" height="353" alt="Screenshot 2026-11-25 at 2 49 00 AM" src="https://github.com/user-attachments/assets/ac32b8e0-57b2-4359-bf72-091fa09e419d" />

  
