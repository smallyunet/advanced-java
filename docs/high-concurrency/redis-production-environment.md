## Interview questions
How is Redis deployed in a production envirnoment?

## Psychnological analysis of interviewers
See if you understand the deployment architecture of your company's redis production cluster. If you don't understand it, then you are really derelict. Is your redis master-slave architecture? Cluster architecture? Which clustering scheme is used? Is there a high avaliability guarantee? Is there a mechanism for enabling persistence to ensure data recovery? Online redis gives a few G's of memory? What parameters are set? How many QPS do your redis clusters carry after the pressure test?

Brother, you must be clear, otherwise you really haven't thought about it.

## Analysis of interview questions
Redis cluster, 10 machines, 5 machines deployed redis master instance, 5 other machines deployed redis slave instances, each master instance hangs slave instance, 5 nodes provide read and write services externally, each node Reading and writing peak qps may reach 50000 per second, and 5 machines can have up to 250000 read/write requests/s.

What is the configuration of the machine? 32G memory + 8 core CPU + 1T disk, but allocated to the redis process is 10g memory, general online production environment, redis memory should not exceed 10g, more than 10g may have problems.

5 machines provide external reading and writing, a total of 50g of memory.

Beacuse each primary instance hangs a secondary instance, it is highly available. Any primary instance is down, and the fault will be automatically migrated. Redis will automatically become the primary instance and continue to provide read and write servides.

What data are you writing into memory? What is the size of each piece of data? Product data, each data is 10KB. 100 data is 1MB, and 100,000 data is 1G. Resident memory is 2 million pieces of commodity data, occupying 20G of memory, only less than 50% of total memory. At the current peak period, it is about 3,500 requests per second.

In fact, in a large company, the team with the infrastructure is responsible for the operation and maintenance of the cache cluster.