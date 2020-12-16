# It Ain't About Performance No More


Performance Calendar articles are mostly about improving web performance. 
That's good because I don't know much about the web: it's very difficult. 
So, I'm going to talk about something much more transparent: cloud performance. 
In that sense, my title should read: *It Ain't About Performance No More...* **in The Cloud**.

Moreover, I'm going to discuss cloud performance using just two images, viz., 
application throughput and latency in the cloud. 
Everybody talks about those two metrics so, that should make it easy.

## Tomcat on AWS
The context is a mobile application running on top of a Tomcat thread-server that also 
communicates with other third-party web services, e.g., hotel and car rental reservation systems. 
The AWS configuration can be summarized as:

	* Elastic load balancer (ELB)
	* AWS Elastic Cluster (EC2) instance type `m4.10xlarge` with 20 CPUs or 40 VPUs 
	* Auto Scaling group (A/S)
	* Mobile users make requests to Apache HTTP server (versions 2.2 and 2.4) via ELB on EC2
	* Tomcat thread server (versions 7 and 8) on EC2 makes calls to 3rd-party web services
	* A/S controls the number of active EC2 instances based on incoming ELB traffic and 
	configured A/S policies
	* ELB balances incoming traffic across all active EC2 nodes in the AWS cluster



## Performance Profiles

![Figure 1](fig1.png)  
<figcaption><b>Figure 1: Throughput profile of Tomcat application  on AWS</b></figcaption>



![](fig2.png) 
<figcaption><b>Figure 2: Latency profile of Tomcat application  on AWS</b></figcaption>


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





