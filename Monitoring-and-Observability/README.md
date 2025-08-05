# Monitoring and Observability

If you can't see what's happening, you're flying blind.

## Basic Questions

1.  **Q: What is the difference between monitoring and alerting?**

    **A:**
    * **Monitoring** is the process of collecting and analyzing data about a system's performance. It's about having dashboards and logs that you can look at to understand the health of your system.
    * **Alerting** is the automated notification that is triggered when a monitoring system detects a problem. An alert should be actionable and signify a condition that requires human attention. A good alert tells you that something is broken or is about to break.

2.  **Q: What are the three pillars of observability?**

    **A:** The three pillars are the key types of telemetry data you need to understand a system:
    * **Metrics:** A numerical representation of data measured over time. For example, CPU usage, memory usage, error rate, or requests per second. They are great for dashboards and alerting on trends.
    * **Logs:** A timestamped, immutable record of discrete events. A log entry tells you what happened at a specific point in time. They are essential for deep-dive debugging.
    * **Traces:** A representation of the end-to-end journey of a single request as it moves through all the different services in a distributed system. They are critical for understanding latency and debugging issues in microservices.

3.  **Q: What is a "time-series database" (TSDB)?**

    **A:** A time-series database is a database that is specifically designed to store and query data points that are indexed by time. This makes them extremely efficient for storing monitoring metrics. Prometheus and InfluxDB are two of the most popular time-series databases used in the DevOps world.

4.  **Q: What are "white-box" and "black-box" monitoring?**

    **A:**
    * **White-box monitoring** is monitoring based on data exposed from the *inside* of a system, like application logs, metrics from the JVM, or database query statistics. It gives you a detailed view of the internal health of the system.
    * **Black-box monitoring** is monitoring the system from the *outside*, as a user would see it. This involves things like pinging a health check endpoint or running a synthetic test that simulates a user logging in. It tells you if the system is actually available and working from an external perspective. You need both for a complete monitoring picture.

5.  **Q: What is a "health check" endpoint?**

    **A:** A health check endpoint is a specific URL in your application (e.g., `/healthz`) that monitoring systems can hit to determine if the application is healthy. A simple health check might just return a `200 OK` status code if the web server is running. A more advanced health check might check its connection to the database or other downstream services before returning a `200 OK`. These are used by load balancers and container orchestrators to know whether to send traffic to an application instance.

## Intermediate Questions

1.  **Q: Explain the pull vs. push model for metrics collection.**

    **A:** This describes how a monitoring system gets its data.
    * **Push Model:** The application is responsible for *pushing* its metrics to a central monitoring system. Tools like StatsD and Datadog work this way. This can be simpler to set up for short-lived jobs or serverless functions.
    * **Pull Model (Scrape Model):** The monitoring system is responsible for reaching out and *pulling* (or "scraping") metrics from an endpoint exposed by the application. Prometheus is the most famous example of a pull-based system.
    * **Pros of Pull:** It's generally more robust for service-based architectures. You can easily discover all the instances you need to monitor through service discovery, and you can tell from the central system if a target is down (because the scrape will fail). It also makes it easier to test your metrics endpoints locally.

2.  **Q: What is Prometheus and what are its core components?**

    **A:** Prometheus is a leading open-source monitoring and alerting system that is a cornerstone of the cloud-native ecosystem.
    * **Core Components:**
        * **Prometheus Server:** The main component that scrapes and stores the time-series data.
        * **Exporters:** These are small helper applications that sit alongside your main application and expose its metrics in the format Prometheus understands. There are exporters for databases, hardware, messaging queues, and more.
        * **Alertmanager:** This handles the alerting logic. It receives alerts from the Prometheus server, and it's responsible for deduplicating, grouping, and routing them to the correct notification channel (like Slack, PagerDuty, or email).
        * **Pushgateway:** A component for supporting short-lived jobs that can't be scraped. The job pushes its metrics to the Pushgateway, which then exposes them for Prometheus to scrape.

3.  **Q: What is the difference between a liveness probe and a readiness probe in Kubernetes?**

    **A:** Both are types of health checks, but they have different purposes.
    * **Liveness Probe:** This tells the kubelet (the agent on the node) if your container is *alive*. If the liveness probe fails, the kubelet will kill the container and restart it. You would use this to recover from a deadlocked application.
    * **Readiness Probe:** This tells the kubelet if your container is *ready to receive traffic*. If the readiness probe fails, the container is not killed, but it is removed from the Service's endpoint list, so it stops receiving traffic. This is used for situations where your application is running but needs time to initialize (e.g., loading a large cache) before it can serve requests.

4.  **Q: What is structured logging and why is it important?**

    **A:** Structured logging is the practice of writing your log messages in a consistent, machine-readable format, like JSON, instead of just plain text strings.
    * **Plain Text Log:** `[ERROR] User login failed for user 'bob' at 2023-10-27T10:00:00Z`
    * **Structured Log (JSON):** `{"level": "error", "message": "User login failed", "user_id": "bob", "timestamp": "2023-10-27T10:00:00Z"}`
    * **Why it's important:** Structured logs are essential for modern log management systems (like the ELK Stack or Splunk). You can easily search, filter, and aggregate them based on specific fields (e.g., "show me all error-level logs for `user_id: bob`"). This makes debugging and analysis in a high-volume system infinitely easier than trying to parse plain text with regular expressions.

5.  **Q: What are the RED metrics?**

    **A:** The RED method is a framework for defining the most important metrics for a request-driven service. It was popularized by Tom Wilkie.
    * **R - Rate:** The number of requests your service is handling per second.
    * **E - Errors:** The number of those requests that are failing.
    * **D - Duration:** The amount of time it takes to process a request (the distribution of latency).

    If you are monitoring these three metrics for every one of your services, you will have a very good high-level understanding of its health and performance.



## Advanced Questions

1.  **Q: What are the USE metrics?**

    **A:** The USE (Utilization, Saturation, Errors) method is a framework developed by Brendan Gregg for analyzing the performance of system resources (like CPU, memory, disks). It's a great checklist for debugging infrastructure issues.
    * **U - Utilization:** The percentage of time the resource was busy. For example, "the CPU is 90% utilized." High utilization is not necessarily a problem; it can mean you're being efficient.
    * **S - Saturation:** A measure of how much extra work the resource has queued up that it can't service. For example, the CPU run queue length. Saturation is the real indicator of a performance bottleneck. A resource can be 100% utilized but have no saturation (it's keeping up perfectly). Or it can be 100% utilized and heavily saturated (it's falling behind).
    * **E - Errors:** The number of error events for that resource.

    For every resource in your system, you should ask: what is its utilization, saturation, and error count?

2.  **Q: What is distributed tracing and how does it work?**

    **A:** Distributed tracing is the technique used to monitor requests as they flow through a distributed system like a microservices architecture.
    * **How it works:**
        1.  When a request first enters the system (e.g., at the API gateway), a unique **Trace ID** is generated.
        2.  This Trace ID is then passed along in the headers of every subsequent network call that is part of that original request.
        3.  Each individual service that receives the request generates a **Span** (which represents a unit of work, like a database call or a call to another service) and tags it with the same Trace ID.
        4.  All of these spans are sent to a tracing backend (like Jaeger or Zipkin). The backend can then reconstruct the entire end-to-end journey of the request by stitching together all the spans that share the same Trace ID, showing you a waterfall diagram of how long each step took. This is invaluable for pinpointing latency bottlenecks in a complex system.

3.  **Q: You are seeing a high rate of "500 Internal Server Error" alerts from your application, but the CPU and memory metrics on your servers look normal. How would you debug this?**

    **A:** This is a classic scenario where basic resource monitoring isn't enough.
    * **Step 1: Check the Logs.** This is the first place to go. Filter your structured logs for `level: "error"`. This will likely give you a stack trace or an error message that points you directly to the problem (e.g., "Cannot connect to database," "Null pointer exception in BillingService").
    * **Step 2: Look at the Application Metrics (RED).** Is the error rate high for all endpoints or just a specific one? Is the duration (latency) for a specific downstream dependency (like a database or another service) spiking?
    * **Step 3: Look at a Trace.** Find a trace for one of the failed requests. The trace will show you exactly which service in the call chain returned the error and what the error was.
    * **Step 4: Check for "Hidden" Resource Exhaustion.** The problem might not be CPU or memory. You could be running out of database connections, file handles, or thread pool capacity. White-box monitoring metrics from the application itself are key here.

    The answer is almost never "the server is fine." The problem is in the application or its dependencies, and you need logs, traces, and application-specific metrics to find it.

4.  **Q: What is a Service Level Objective (SLO), and how does it relate to a Service Level Agreement (SLA)?**

    **A:** This is a key concept from Site Reliability Engineering (SRE).
    * **SLO (Service Level Objective):** An internal target for the reliability of your service. It's a precise numerical target for a specific metric. For example: "99.9% of home page requests over the last 28 days will be served in under 200ms." SLOs are what you, the engineering team, use to make decisions.
    * **SLA (Service Level Agreement):** An external contract with your customers that defines the level of service they can expect and what the consequences are if you fail to meet it (e.g., a refund).
    * **The Relationship:** Your internal SLO should always be stricter than your external SLA. For example, you might have a 99.9% internal SLO to give yourself a buffer to meet your 99.5% external SLA. The SLO is the tool you use to manage your "error budget."

5.  **Q: What is an "Error Budget"?**

    **A:** An error budget is a direct result of setting an SLO. If your SLO is 99.9% availability, that means you are "budgeting" for 0.1% unavailability. This is your error budget.
    * **How it's used:** It gives you a data-driven way to balance reliability with feature development.
        * If you have a lot of your error budget left for the month, you can feel comfortable taking more risks and shipping new features quickly.
        * If you have burned through your error budget (due to an outage or a series of buggy releases), the policy should be to freeze new feature releases and dedicate all engineering effort to reliability improvements until the service is back within its SLO.
        
    It's a powerful concept that turns the abstract goal of "reliability" into a concrete metric that engineers and product managers can use to make decisions.