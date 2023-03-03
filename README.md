# Nereus

Nereus: A Distributed Stream Band Join System with Adaptive Range Partitioning

Many real-time applications in consumer electronics rely on stream band join as a fundamental operation. With two streams, the band join operation targets at obtaining the pairs of tuples which are separately included in the two streams and have close values within a user specified range. Range partitioning keeps tuples with close values in a partition. For band join, the cost of employing range partitioning is less join cost than that of employing other partitioning strategies. However, the distribution of real-world data from consumer electronics over the range is skewed, causing severe load imbalance among instances in the distributed system that employs existing static range partitioning. Load migration can alleviate the load imbalance. For range partitioning, the migration has two kinds of objectives. Migrating load to the instance with adjacent partition controls the number of partitions. However, it causes an unacceptable high cost for migrating span multiple instances. While directly migrating a split partition to the lightest instance is low cost. However, it leads to an uncontrollable number of partitions. The system for consumer electronic applications cannot tolerate high migration cost and an uncontrollable number of partitions, which result in high latency and low throughput.

In this work, we propose an adaptive range partitioning strategy to ensure a controllable number of partitions and load balancing with low cost. We implement Nereus, a distributed stream band join system. Nereus designs a migration benefit model using queuing theory measure, which integrates the benefits of partition's change and load balancing. Such a design can obtain the most beneficial migration, which achieves low migration cost and an appropriate number of partitions. We conduct comprehensive experiments using large-scale datasets from real-world applications to evaluate this design. The results show that Nereus improves the throughput by 51\% and reduces the processing latency by 99\%, compared to existing designs.

# Introduction
In recent years, an increasing number of consumer electronic devices with high-speed computing combined with emerging modern applications provide a variety of services to consumers. In order to enhance the consumer experience, plenty of modern applications in consumer electronics need to respond to the consumers in real-time. 

For example, an on-demand ride-hailing platform matches orders for passengers and drivers with smartphones in real-time. The platform keeps finding the drivers near the passengers through GPS positioning. To reduce the computation of path distances, the platform usually uses space-filling curves to convert the geographic position into one-dimensional distance. For the order stream of passenger and track stream of driver, this task can be formed by comparing each new passenger's order with the drivers, to find all of the drivers that are within a specified distance of the passenger. The predicate for the stream join is $driver.pos-distance\leq passenger.pos\leq driver.pos+distance$, where $driver.pos$ and $passenger.pos$ are the GPS positioning of driver and passenger in the one-dimensional space, and $distance$ is the preset distance between the passenger and driver.

For another example, online game platforms match players and ensure that consumers can match the opponents of close strength in real-time, such as League of Legends and World of Warcraft.

In these examples of stream join, for each new tuple having a specific value from one stream, the stream join operation finds all of the tuples from another stream within a user specified range. The predicate for stream band join between streams $R$ and $S$ on attributes $R.x$ and $S.x$ is $R.x-c_1\leq S.x \leq R.x+c_2$. The constants $c_1$ and $c_2$ may be equal, and one of the two may be zero. $c_1+c_2$ is the size of the user specified range. The stream band join operation targets at obtaining the pairs of tuples which are separately included in the two streams and have close values. The predicate of stream band join is comparable to that for band join in static relational database. This stream band join processes constantly arriving and highly dynamic data.

## Building Nereus

Nereus source code is maintained using [Maven](http://maven.apache.org/). Generate the excutable jar by running

    mvn clean package

## Running Nereus

Nereus is built on top of [Storm](https://storm.apache.org/). After deploying a Storm cluster, you can launch BiStream by submitting its jar to the cluster. Please refer to Storm documents for how to [set up a Storm cluster](https://storm.apache.org/documentation/Setting-up-a-Storm-cluster.html) and [run topologies on a Storm cluster](https://storm.apache.org/documentation/Running-topologies-on-a-production-cluster.html).
Running 

    storm jar /home/join/Nereus/target/Nereus-1.0-SNAPSHOT.jar soj.biclique.KafkaTopology -n 40 -ks 1 -pr 100 -ps 100 -f1 4 -f2 4  -infile sorted_20161101  -wr 40000 -ws 40000 -tr 46000 -ts 46000 --s band -t 2 -nps 3 -ns 600 -mig -jump -conti -e1 50 -e2 50   
