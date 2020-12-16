# It Ain't About Performance No More


Perf Calendar articles are about the web. 
I don't know much about the web (too obscure) so I'm going to say something about the cloud (more transparent). 
It Ain't About Performance No More ... in the cloud.

And I'm going to do it with with just two images.

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


  
  ## References
  * [Tomcat-Applikationsperformance in der Amazon-Cloud unter Linux modelliert](https://www.linux-magazin.de/ausgaben/2019/02/aws-performance/) (Linux Magazin 2019 in German)
  * [Linux-Tomcat Application Performance on Amazon AWS](https://arxiv.org/abs/1811.12341) (2019 in English)
  * [How to Scale in the Cloud: Chargeback is Back, Baby!](https://speakerdeck.com/drqz/how-to-scale-in-the-cloud-chargeback-is-back-baby) (2019 Rocky Mountain CMG slides)


foo bar 



