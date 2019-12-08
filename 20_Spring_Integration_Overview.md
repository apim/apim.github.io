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
![](/images/sp_p1.jpg)

* **Filter** - Determines whether a message should be passed to an output channel at all.