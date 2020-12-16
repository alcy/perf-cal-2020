# It Ain't About Performance No More


Performance Calendar articles are mostly about improving web performance. 
That's good because I don't know much about the web: it's very complicated. 
So, I'm going to talk about something much more transparent: cloud performance. 
In that sense, my title should read: *It Ain't About Performance No More...* **in The Cloud**.

Moreover, I'm going to discuss cloud performance using just two figures, viz., 
application throughput and application latency in the cloud. 
Everybody loves to talk about those two metrics so, that should make things easy.


## Tomcat on AWS
The context is a mobile application running on top of a Tomcat thread-server that also 
communicates with other third-party web services, e.g., hotel and car-rental reservation systems. 
A variable number of application instances are active during each 24 hour business cycle, depending 
on the user traffic. The elastic capacity requirements are handled by Amazon Web Services (AWS). 

The AWS cloud configuration can be summarized as:

  * Elastic load balancer (ELB)
  * AWS Elastic Cluster (EC2) instance type `m4.10xlarge` with 20 CPUs or 40 VPUs 
  * Auto Scaling group (A/S)
  * Mobile users make requests to Apache HTTP server (versions 2.2 and 2.4) via ELB on EC2
  * Tomcat thread server (versions 7 and 8) on EC2 makes calls to 3rd-party web services
  * A/S controls the number of active EC2 instances based on incoming ELB traffic and configured A/S policies
  * ELB balances incoming traffic across all active EC2 nodes in the AWS cluster

All the subsequent performance data discussed here are taken from this is a production environment. 




## Performance Profiles

Performance data was collected using combinations of these FOSS tools:

  * JMX (Java Management Extensions) data from JVM
  * jmxterm
  * VisualVM
  * Java Mission Control Datadog dd-agent
  * Datadog — also integrates with AWS CloudWatch metrics 
  * Collectd — Linux performance data collection
  * Graphite and statsd — application metrics collection & storage 
  * Grafana — time-series data plotting
  * Custom data collection scripts
  * R statistical libraries and the RStudio IDE
  * PDQ queueing analyzer tool 

More details about the data collection procedures can be found in References 1 and 2 below. 



### Throughput profile

First, let's consider the application throughput profile shown in Figure 1. 
The most important thing to note about this plot of throughput is that it is **not** a time 
series&mdash;that every performance monitoring tool spits out. 

As general matter, time are rather useless for doing deeper performance analysis. 
Just because a monitoring tool produces time-series plots, doesn't make them particularly useful. 
It's just an obvious and straightforward thing for them to do. 
Instead, Figure 1 shows the steady-state view of the throughput, X(N), as a nonlinear function of 
the mobile user request load, N. In other words, N is the independent variable and X(N) is the 
dependent variable. All steady-state throughput profiles are *concave* functions. 
Since each user-request is assigned to a Tomcat thread, we can label N as "users" or 
"requests" or "threads". They're all logically equivalent. 

![Figure 1](fig1.png)  
<figcaption><b>Figure 1: Throughput profile of Tomcat application on AWS</b></figcaption>


The dots are the measured throughput at a given load. Each dot corresponds to a particular 
timestamp when the data was sampled but, that information has becomes *implicit* rather than 
explicit in Figure 1. 
The data points range approximately between N = 100 
user-threads and N = 500 threads. On reflection, it should be clear that the lower values of N 
correspond to the quiescent period during the 24 hour window and conversely, 
the higher values of N correspond to the heaviest daily traffic. 

What is not apparent from a simple scatterplot of those data is that they tend to fall along 
two line segments:
  1. the diagonal red dashed line (up to N = 300)
  1. the horizontal red dashed line (beyond N = 300)

The knee in the data is indicated by the blue arrow. 

The red lines represent the *statistical mean* of the measured data&mdash;in the sense 
of linear regression analysis.  The variation in the data corresponds to statistical fluctuations
(or "noise") about the mean. 

Moreover, these red dashed line have a particular meaning in queueing theory&mdash;you do know you're queuing theory 
don't you? (see Ref. 4) 
The diagonal line represents the ideal **parallel** performance bound. In other words, you cannot 
have a throughput better than that as you increase the request load; on average. 
Similarly, the horizontal line represents the ideal **saturation** performance bound. 
You cannot have a throughput that exceeds that bound; on average. 
As the data shows, you can have *instantaneous* values that exceed these bounds but 
they only transient. 

In case you're wondering, yes, Figure 1 only shows measurements from one EC2 instance. 
For the standpoint of PDQ, that's the correct approach. All the instances are supposed to scale 
identically. That's the job of the ECB load balancer and that's the assumption of PDQ. 
If instances are not scaling identically, that's not the fault of queueing theory, that's the 
fault of the load balancer configuration of some other effect in the infrastructure.  

```R
library(pdq)

# Globals
requests <- seq(50, 500, 50) # from mobile users
threads  <- 300   # max threads under AWS auto scale policy
stime    <- 0.444 # measured service time in seconds
xx       <- NULL  # x-axis load points 
yx       <- NULL  # corresponding throughput 
```

```R
# PDQ model
aws.model <- function(nindex) {
  pdq::Init("")  
  pdq::CreateClosed("Requests", BATCH, requests[nindex], 0.0)
  pdq::CreateMultiNode(threads, "Threads", MSC, FCFS) 
  pdq::SetDemand("Threads", "Requests", stime) 
  pdq::SetWUnit("Reqs")
  pdq::Solve(EXACT)
  xx[nindex] <<- requests[nindex] # update global vector
  yx[nindex] <<- pdq::GetThruput(BATCH, "Requests")
}
```

```R
# Loop over mobile requests
for (i in 1:length(requests)) {
  aws.model(i)
}
```

```R
# Plot throughput X(N)
plot(xx, yx, 
     type="p", col="blue", pch=0, 
     xlim=c(0,500), ylim=c(0,800), 
     xlab="Threads, N", ylab="Request rate, X(N)"
)
```

### Latency profile

Next, let's look at the corresponding response time profile. 

![](fig2.png) 
<figcaption><b>Figure 2: Latency profile of Tomcat application  on AWS</b></figcaption>

Figure 2 shows the steady-state view of the response time, R(N), as a nonlinear function of 
the mobile user request load, N. Here, R(N) is the 
dependent variable. All steady-state response time profiles are *convex* functions. 


  * AWS tomcat application
  * IMAGE of throughput scaling under AWS Auto Scaling 
      * plot of X data
      * plot of PDQ overlaid
  * Good as it can be !!! (but that's just throughput)
  * IMAGE of response time (don't forget that)
      * plot of X data
      * plot of PDQ overlaid
  * RT penalty still can't be ignored
  * PDQ code in R sans plot loop
  * But minimal ROI for more tuning
  * It's all about CHARGEBACK 

production application 

queueing theory .... encapsulated in PDQ queueing analyzer

show PDQ code as: loop + subroutine


## The Punchline

Now you can see the meaning of my title. It's not about performance in most cases because 
scaling is taken care of automagically&mdash;more or less. Nobody was more surprised by this 
result than me. This Tomcat application is scaling maximally. The throttling at N = 300 threads 
is a consequence of the Auto Scaling policy that CPU busy not exceed 75% on any EC2 instance. 

So, what is it all about? It's about cost or, more formally, capacity planning. 
... TBC


  
## References
  1. [Tomcat-Applikationsperformance in der Amazon-Cloud unter Linux modelliert](https://www.linux-magazin.de/ausgaben/2019/02/aws-performance/) (Linux Magazin 2019 in German)
  1. [Linux-Tomcat Application Performance on Amazon AWS](https://arxiv.org/abs/1811.12341) (2019 in English)
  1. [How to Scale in the Cloud: Chargeback is Back, Baby!](https://speakerdeck.com/drqz/how-to-scale-in-the-cloud-chargeback-is-back-baby) (2019 Rocky Mountain CMG slides)
  1. [PDQ: Pretty Damn Quick Performance Analyzer](http://www.perfdynamics.com/Tools/PDQ.html)
  1. [PDQ Software Distribution version 7.0](http://www.perfdynamics.com/Tools/PDQcode.html)
  1. [PDQW Tutorial Workshop](http://www.perfdynamics.com/Classes/Outlines/pdqw.html)



