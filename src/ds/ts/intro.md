Redis supports a time series data structure, allowing Redis to function as a time series database.

Common examples of time series data include:

* Sensor data: e.g., temperatures or fan velocities for servers in a server farm.
* Stock prices.
* The number of vehicles passing through a given road (e.g., counts per minute).

Common examples for applications that use time series databases:

* IoT & Sensor Monitoring: monitor and react to a stream of events emitted by devices.
* Application Performance/Health Monitoring: monitor the performance and availability of applications and services.
* Real-time Analytics: process, analyze, and react in real-time (e.g., for selling equities, performing predictive maintenance, product recommendations, or price adjustments).

You can ingest and query millions of samples and events at the speed of Redis. 

### Prerequisites

[Redis Stack Server](https://redis.io/downloads/?utm_source=redisinsight&utm_medium=app&utm_campaign=timeseries_tutorial) >=7.2.0-v7 \
OR \
[RedisTimeSeries](https://redis.io/learn/modules/redistimeseries/?utm_campaign=timeseries_tutorial&utm_medium=app&utm_source=redisinsight) >=1.10.11 \
OR \
A free Redis Stack instance on [Redis Cloud](https://redis.io/try-free/?utm_source=redisinsight&utm_medium=app&utm_campaign=timeseries_tutorial).
