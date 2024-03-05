## Steps to follow to target a high level design problem:

1. **Features / Minimum Viable Product (MVP)** - gather minimum set of feature required to launch the products
2. **Estimation of scale** - if the product is small scale or large scale (number of users) this helps to make design decisions
    * Sharding is a necessity or not? - depend on the TB of data or not (1 machine handles 4TB of data)
    * System is Read heavy (FB post) or Write heavy (IoT, sensor based system) or Read + Write heavy (messenger)
3. **Decide on design trade-offs** - Mark or note all the important behaviors needed
    * Stateless or stateful app server
    * SQL or no-sql DB
    * Horizontal or vertical scaling
    * Consistency or availability - CAP theorem
    * Latency (High, Low, Super low)
    * Caching (where to and its trade-offs)
4. **Design deep dive** -
    * What APIs to expose?
    * What components to create?
    * Data flow of the system