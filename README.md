# Nereus

Nereus: A Distributed Stream Band Join System with Adaptive Range Partitioning

Many real-time applications in consumer electronics rely on stream band join as a fundamental operation. With two streams, the band join operation targets at obtaining the pairs of tuples which are separately included in the two streams and have close values within a user specified range. Range partitioning keeps tuples with close values in a partition. For band join, the cost of employing range partitioning is less join cost than that of employing other partitioning strategies. However, the distribution of real-world data from consumer electronics over the range is skewed, causing severe load imbalance among instances in the distributed system that employs existing static range partitioning.

In this work, we propose an adaptive range partitioning strategy to ensure a controllable number of partitions and load balancing with low cost. We implement Nereus, a distributed stream band join system. Nereus designs a migration benefit model using queuing theory measure, which integrates the benefits of partition's change and load balancing. Such a design can obtain the most beneficial migration, which achieves low migration cost and an appropriate number of partitions. We conduct comprehensive experiments using large-scale datasets from real-world applications to evaluate this design. The results show that Nereus improves the throughput by 51\% and reduces the processing latency by 99\%, compared to existing designs.

# Introduction
In recent years, an increasing number of consumer electronic devices with high-speed computing combined with emerging modern applications provide a variety of services to consumers. In order to enhance the consumer experience, plenty of modern applications in consumer electronics need to respond to the consumers in real-time. 

For example, an on-demand ride-hailing platform matches orders for passengers and drivers with smartphones in real-time. The platform keeps finding the drivers near the passengers through GPS positioning. To reduce the computation of path distances, the platform usually uses space-filling curves to convert the geographic position into one-dimensional distance. For the order stream of passenger and track stream of driver, this task can be formed by comparing each new passenger's order with the drivers, to find all of the drivers that are within a specified distance of the passenger. The predicate for the stream join is $driver.pos-distance\leq passenger.pos\leq driver.pos+distance$, where $driver.pos$ and $passenger.pos$ are the GPS positioning of driver and passenger in the one-dimensional space, and $distance$ is the preset distance between the passenger and driver.

For another example, online game platforms match players and ensure that consumers can match the opponents of close strength in real-time, such as League of Legends and World of Warcraft.

In these examples of stream join, for each new tuple having a specific value from one stream, the stream join operation finds all of the tuples from another stream within a user specified range. The predicate for stream band join between streams $R$ and $S$ on attributes $R.x$ and $S.x$ is $R.x-c_1\leq S.x \leq R.x+c_2$. The constants $c_1$ and $c_2$ may be equal, and one of the two may be zero. $c_1+c_2$ is the size of the user specified range. The stream band join operation targets at obtaining the pairs of tuples which are separately included in the two streams and have close values. The predicate of stream band join is comparable to that for band join in static relational database. This stream band join processes constantly arriving and highly dynamic data.

# Insight
To alleviate load imbalance, the stream join processing system can migrate the load among the instances. Emigration and immigration instances represent the instances of emigrating and immigrating load. The immigration instances are divided into adjacent and non-adjacent instances according to whether they have partitions with an adjacent range to the emigration instance. We take the heaviest instance (i.e., the instance with the heaviest load) as the emigration instance, and conclude two kinds of schemes based on the two immigration instances. One kind of migration scheme emigrates part of the partition to the adjacent instance, and then merges the two adjacent partitions. This scheme controls the number of partitions. However, to eliminate the system bottleneck of the heaviest instance, the migration will span multiple instances since the adjacent instances of emigration instance are sometimes under high load. This results in an unacceptably high migration cost for stream processing. Another kind of scheme emigrates part of the load in the partition to the lightest instance (i.e., the instance with the lightest load)}, which is usually a non-adjacent instance. This migration only involves two instances, which is a low cost migration scheme compared to the migration scheme for adjacent instance. However, this scheme splits a partition, which makes ordered tuples more discrete. Too many partitions will degrade system performance. However, the stream join system for consumer electronics cannot tolerate high migration cost and the uncontrolled number of partitions.

We have two insights for the two schemes. First, the disadvantage's influences of the two schemes are related to the current degree of load imbalance and the number of partitions. In fact, these influences are different at any moment. We can utilize this difference to design the most beneficial migration to the system. It is challenging to translate two different metrics of load imbalance and the number of partitions into the same metric. Second, the migration scheme for adjacent instance causes a high migration cost. If the instance has multiple adjacent instances, this migration cost will be reduced. However, the existing schemes cannot make instances own multiple adjacent instances. 

# Architecture of Nereus
<div align=center>
<img src="https://github.com/shuiying1037/Nereus/blob/main/src/main/resources/architecture.jpg" width="600" height="400" alt="Proportional incremental strategy"/>
</div>

 
# Adaptive Range Partitioning
## The four migration operations
<div align=center>
<img src="https://github.com/shuiying1037/Nereus/blob/main/src/main/resources/migrationschemes.png" width="400" height="400" alt="Proportional incremental strategy"/>
</div>

## Migration Benefit Model

The total benefit: $B=B_n+B_l$

<div align=center>
<img src="https://github.com/shuiying1037/Nereus/blob/main/src/main/resources/router-instance.png" width="400" height="200" alt="Proportional incremental strategy"/>
</div>

# About the source code and data sets

## Building Nereus

Nereus source code is maintained using [Maven](http://maven.apache.org/). Generate the excutable jar by running

    mvn clean package

## Running Nereus

Nereus is built on top of [Storm](https://storm.apache.org/). After deploying a Storm cluster, you can launch BiStream by submitting its jar to the cluster. Please refer to Storm documents for how to [set up a Storm cluster](https://storm.apache.org/documentation/Setting-up-a-Storm-cluster.html) and [run topologies on a Storm cluster](https://storm.apache.org/documentation/Running-topologies-on-a-production-cluster.html).
Running 

    storm jar /home/join/Nereus/target/Nereus-1.0-SNAPSHOT.jar soj.biclique.KafkaTopology -n 40 -ks 1 -pr 100 -ps 100 -f1 4 -f2 4  -infile sorted_20161101  -wr 40000 -ws 40000 -tr 46000 -ts 46000 --s band -t 2 -nps 3 -ns 600 -mig -jump -conti -e1 50 -e2 50 
    
# Evaluation Result
Here, we show the result on dataset of the real-world DiDi Chuxing gaia initiative.

We compare the performance of the basic method, Nereus, and BiStream. The basic method employs join-biclique model and static range partitioning. The basic method allocates a partition in an instance. For Nereus, we adopt adaptive range partitioning. By default, we initialize the number of partitions in each instance to three. We compare the throughput and processing latency of the basic method, Nereus, and BiStream systems, and collect the values in four primary metrics across Nereus runs. The metrics include the load, $LI$, the diameter of the partition connection network, and the number of partitions.

<div align=center>
<img src="https://github.com/shuiying1037/Nereus/blob/main/src/main/resources/throughput-4700032000.png" width="200" height="150" alt="Proportional incremental strategy"/>
<img src="https://github.com/shuiying1037/Nereus/blob/main/src/main/resources/latency-47000.png" width="200" height="150" alt="Proportional incremental strategy"/>
<img src="https://github.com/shuiying1037/Nereus/blob/main/src/main/resources/load.png" width="200" height="150" alt="Proportional incremental strategy"/>
<img src="https://github.com/shuiying1037/Nereus/blob/main/src/main/resources/numPartitions.png" width="200" height="150" alt="Proportional incremental strategy"/><br/>
</div>


<div align=center>
<img src="https://github.com/shuiying1037/Nereus/blob/main/src/main/resources/latency-onlyModifying-JumpConti-NonMig.png" width="200" height="150" alt="Proportional incremental strategy"/>
<img src="https://github.com/shuiying1037/Nereus/blob/main/src/main/resources/latency-throughput-numPar.png" width="200" height="150" alt="Proportional incremental strategy"/>
<img src="https://github.com/shuiying1037/Nereus/blob/main/src/main/resources/parallelism-throughput.png" width="200" height="150" alt="Proportional incremental strategy"/>
<img src="https://github.com/shuiying1037/Nereus/blob/main/src/main/resources/parallelism-latency.png" width="200" height="150" alt="Proportional incremental strategy"/><br/>
</div>
 
 ## Publication

If you want to know more detailed information, please refer to this paper:

Shuiying Yu, Hanhua Chen, Hai Jin. Nereus: A Distributed Stream Band Join System with Adaptive Range Partitioning. IEEE Transactions on Consumer Electronics 2023.

## Authors and Copyright

Auxo is developed in National Engineering Research Center for Big Data Technology and System, Cluster and Grid Computing Lab, Services Computing Technology and System Lab, School of Computer Science and Technology, Huazhong University of Science and Technology, Wuhan, China by Shuiying Yu (shuiying@hust.edu.cn), Hanhua Chen (chen@hust.edu.cn), Hai Jin (hjin@hust.edu.cn).

Copyright (C) 2021, [STCS & CGCL](http://grid.hust.edu.cn/) and [Huazhong University of Science and Technology](https://www.hust.edu.cn/).

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

```
  http://www.apache.org/licenses/LICENSE-2.0
```
Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

