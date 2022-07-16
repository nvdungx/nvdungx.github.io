---
layout: post
title: Event-driven programming
date: 2022-12-22 00:00:00
categories: [programming]
tags: [development, beginner]
last_modified_at: 2022-12-28
---

# Event-driven programming
  Event-driven is a program structure that can react to multiple event sequences, which can arrive at unpredictable times.
Event-driven application is not in control while waiting for an event(i.e. not active). Once the event arrives, the application is called to handle the event and then it return to waiting state. This allows an event-driven system to wait for multiple events, so the system remains responsive to all events it needs to handle.  

Event-driven program typically include: event loop and applications(event process function)  
The event loop: is the only place where the program waits (BLOCK) for events.  
The event-driven application: is not allowed to block(NO BLOCKING), because it must quickly return control after handling each event back to the event loop.  

The event-driven systems shall have key characteristic such as:  
1. asynchronous event delivery  
2. run to completion  
3. inversion of control: event drive the application(i.e. from application point of view, the control is resided in event-loop or the sequences of occurred event in the system = inverted compare to sequential program)  
4. non-blocking application  

General structure of an event-loop:
{% highlight c %}
while (1)                /* event-loop */
{
    Event e = get_event();  /* BLOCKING: the event-loop get_event from event queue */
    dispatch(e);          /* NO-BLOCKING: application process the incoming event*/
}
{% endhighlight %}

> **event object:** signals(ID) and parameters(Data)  
> **producer:** actor that push event object to event queue(e.g. os)  
> **consumer:** actor that pop event object from event queue and dispatch to corresponding event handler(e.g c++ asio io_context, python asyncio)  
> **subscriber:** actor that handle event  
> sequential programming shall not be mixed with event-driven programming  

# Best practices for concurrency & active object pattern

Best practices when to use threads correctly:
  - isolation of private data between threads
  - asynchronous messages

### Active object design pattern
- Active Object = private-data + private-thread + private-queue
- To communicate with Active Object: asynchronously posting event to its queue
- Events are processed to completion on private thread
- Strictest form of OOP

⟹ Active Object: combination of RTOS, OOP, Event-Driven programming


