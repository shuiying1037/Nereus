# Nereus

Nereus: A Distributed Stream Band Join System with Adaptive Range Partitioning

Many real-time stream processing applications in consumer electronics rely on band join as a fundamental operation. For each new data of one stream, stream band join finds all of the data from another stream within given range of it. Efficient band join depends on range partitioning, which maps data to partitions of established range. However, the distribution of real-world data over the range is skewed. This causes severe load imbalance among instances in the distributed system, and influences the system performance. Load migration can alleviate the load imbalance. We migrate load from the heaviest instance in range partitioning, which exists two kinds of migration objectives. Migrating load to the instance with adjacent range controls the number of partitions. However, it causes an unacceptable high cost for migrating span multiple instances. While migrating a split partition to the lightest instance is low cost, because only two instances are involved. However, it result in an uncontrollable number of partitions. The system cannot tolerate high migration cost and uncontrollable number of partitions.

In this work, we propose an Adaptive Range Partitioning to ensure a controllable number of partitions and load balancing with low cost, and implement the distributed stream band join system, Nereus. We design a migration benefit model using queuing theory measure, which integrates the benefits of partition's change and load balancing. Such a design can obtain the most beneficial migration, which achieves low migration cost and self-adjust appropriate number of partitions. Extensive experiment results reported for real-world datasets generated from consumer electronics show that Nereus improves the throughput by 51\% and reduces the processing latency by 99\%, compared to existing method.


## Building Nereus

Nereus source code is maintained using [Maven](http://maven.apache.org/). Generate the excutable jar by running

    mvn clean package

## Running Nereus

Nereus is built on top of [Storm](https://storm.apache.org/). After deploying a Storm cluster, you can launch BiStream by submitting its jar to the cluster. Please refer to Storm documents for how to [set up a Storm cluster](https://storm.apache.org/documentation/Setting-up-a-Storm-cluster.html) and [run topologies on a Storm cluster](https://storm.apache.org/documentation/Running-topologies-on-a-production-cluster.html).
Running 

    storm jar /home/join/Nereus/target/Nereus-1.0-SNAPSHOT.jar soj.biclique.KafkaTopology -n 40 -ks 1 -pr 100 -ps 100 -f1 4 -f2 4  -infile sorted_20161101  -wr 40000 -ws 40000 -tr 46000 -ts 46000 --s band -t 2 -nps 3 -ns 600 -mig -jump -conti -e1 50 -e2 50   
