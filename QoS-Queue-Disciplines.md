### About
Starting with 380.59 (for Traditional QoS) and 380.67 (for Adaptive QoS), you can now change the QoS engine to use a different queue discipline (qdisc) than the default SFQ qdisc.

The benefits of codel and fq_codel over sfq is to better handle the delay that can occur in packet transmission under certain scenario (either due to excessive buffering or bandwidth saturation).  This will generally translate into lower latency when you are already transferring at the max speed available from your ISP, and also reduce the so-called bufferbloat effect that can occur when your connection goes through some excessive buffering at some level along the route.


### Overhead
To further improve the accuracy, you can also configure the typical overhead that your ISP adds to your traffic.  Unless you really know what you are doing, you'll generally want to apply one of the available presets.  If unsure, either go with what's closest to your actual connection technology (DOCSIS, VDSL or ADSL2), or leave it to 0.


### Configuration
These settings can be adjusted on the QoS configuration page.

While both qdiscs are available, you will generally want to use fq_codel - codel is only provided for sake of completeness.

It's also always advised to manually configure your max upload and download speed to somewhere between 85% and 95% of your maximum speed you're able to reach with your ISP.  You will have to experiment to determine the best values to use, as this will depend once again on your specific network.


### Results
As not everyone's network is the same, your results may vary depending on numerous factors, including your ISP and its equipment.  fq_codel isn't something that will magically allow you to saturate your Internet connection with zero impact on general latency, nor will it make your connection faster.  It can however lead to a better overall experience while using your Internet connection.


### Availability
Note that due to the age of the kernel used by the RT-N66U and RT-AC66U, these two models do not support the new queue disciplines.
