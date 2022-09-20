# Nereus

Nereus: A Distributed Stream Band Join System with Adaptive Range Partitioning

Many real-time stream processing applications require Band Join as fundamental operations. Band join relies on proper range partitioning for efficient localized join, but the range partitioning in existing systems suffers from the natural skewness and dynamics of stream data. This leads to poor or invalid range partitioning. Our evaluation verify that the existing partitions exhibit severe load imbalance, resulting in low system throughput and high latency. 

Fast load balancing requires a greedy migration scheme, while localized join necessitates a smooth migration scheme. To have both, the system should rationally choose the smooth and greedy schemes by measuring the overall state of the system and the migration benefit of the two schemes. Accordingly, we propose adaptive range partitioning and design Nereus, a distributed stream band join system. Nereus concatenate schemes of Modifying bound, Moving region, Splitting region and Merging region, cooperating load measuring and queuing theory to support decision.

## Building Nereus

Nereus source code is maintained using [Maven](http://maven.apache.org/). Generate the excutable jar by running

    mvn clean package

## Running Nereus

Nereus is built on top of [Storm](https://storm.apache.org/). After deploying a Storm cluster, you can launch BiStream by submitting its jar to the cluster. Please refer to Storm documents for how to [set up a Storm cluster](https://storm.apache.org/documentation/Setting-up-a-Storm-cluster.html) and [run topologies on a Storm cluster](https://storm.apache.org/documentation/Running-topologies-on-a-production-cluster.html).
Running 

    storm jar /home/join/Nereus/target/Nereus-1.0-SNAPSHOT.jar soj.biclique.KafkaTopology -n 40 -ks 1 -pr 100 -ps 100 -f1 4 -f2 4  -infile sorted_20161101  -wr 40000 -ws 40000 -tr 46000 -ts 46000 --s band -t 2 -nps 3 -ns 600 -mig -jump -conti -e1 50 -e2 50   
