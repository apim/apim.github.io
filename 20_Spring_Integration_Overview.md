# Spring Integration Overview

Spring Integration, in the simplest of definition, is Spring's implementation of **Enterprise Application Integration (EAI) Patterns**. A quick-start of EAI can be obtained from the original source here - http://www.eaipatterns.com/. Spring Integration enables lightweight messaging within Spring-based applications and supports integration with external systems. It takes the concept of core Spring one step further, where POJOs are wired together using a messaging paradigm and the platform provides a wide selection of channel end-points to communicate with external systems. The messaging systems in Spring Integration typically follow the *pipes-and-filters* model.

**Main components of Spring Integration:**

1. **Message** - It is a generic wrapper for any Java object and consists of a payload and headers.
2. **Channel** - This represents the *pipe* of a pipes-and-filters architecture and may follow either Point-to-Point or Publish/Subscribe semantics. A channel will have below characteristics:
  * Channels are separate from applications
  * Channels are asynchronous & reliable
  * Data is exchanged in self-contained messages
3. **Endpoints** - This represents the *filter* of a pipes-and-filters architecture and following the virtue of *Inversion of Control*, the endpoints connect application code to the messaging framework in a non-invasive manner.

There are **2 types of channels** in Spring Integration - Pollable Channel & Subscribable Channel. A **Pollable Channel** offers buffering by means of a queue which always has a fixed capacity. This is basically a point-to-point messaging channel and subscribers actively poll the channel for their feed. On the other hand, **Subscribable Channel** does not provide any buffering capability however, it maintains a list of subscribers who are intimated in the instant a message arrives in the channel.

You can connect two channels via a Bridge. Bridge is a trivial end-point where two different channels are paired together e.g. a pollable channel is connected to a subscribable channel, which is again connected to an application, via a messaging bridge. In this way, application does not worry about any polling configuration as that is hidden behind the bridge.

There are various types of endpoints supported in Spring Integration and here is a brief listing of them (all are based upon at least one or some of the EAI patterns).

* **Transformer** - Converts a message's content and/or structure and returns the modified message.
![](/images/si_p1.jpg)

* **Filter** - Determines whether a message should be passed to an output channel at all.
![](/images/si_p2.jpg)

* **Router** - Decides what output channel or channels shall receive the input message (if any). It is a proactive solution compared to reactive approach in Filter. 
![](/images/si_p3.jpg)

* **Splitter** - Decomposes the input message into multiple messages and send them to its output channel or channels for prospective different processing. 
![](/images/si_p4.jpg)

* **Aggregtor** - Like mirror image of Splitter. Composes messages from input channel into a single message. This is stateful as it stores the status of final message aggregation. 
![](/images/si_p5.jpg)

* **ServiceActivator** - A particular end-point which connects a service to the messaging system. Upon receiving message in the input channel, service activator invokes a service which may or may not return a response. In the case a response comes, the same is forwarded to an output channel. Request to the external service is always a synchronous call. 
![](/images/si_p6.jpg)

* **Channel Adapter** - A particular end-point which connects an application to the messaging system. This can be either inbound or outbound, essentially, always uni-directional. There are various channel adapter supports inbuilt in Spring Integration for JMS, HTTP, Files, FTP, Mail etc. Below diagram is an example of inbound channel adapter. 
![](/images/si_p7.jpg)

* **Gateway** - Hides messaging API within an application API and essentially, can be bi-directional. An application invokes Spring Integration messaging code by just making a method call and the said method implementation contains all messaging logic (the concerned method is configured in Spring context and from that Spring manages all messaging parts like channels and endpoints etc.). By this way the rest of the parts of an application code get totally decoupled from Spring Integration part.
![](/images/si_p8.jpg)