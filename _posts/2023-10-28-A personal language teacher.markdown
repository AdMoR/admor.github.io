---
description: German, French, English and more
tags: python LLM GenerativeAI
img: ai_language_teacher.png
comments: true
---


## Introduction

In this previous [post](https://admor.github.io/Creating-a-virtual-streamer/), we have seen we can have a video agent answering to a written chat. A perfect match for Twitch.

But we could go further in many direction : 
- Allow a personal conversation with the bot
- Scale the architecture to have 10's of conversation in parallel
- Handle multiple personas and languages

 In this detail, we will detail how things could be implemented


## Too Long Didn't Read : 

An example of the language teacher app.


You can interact orally and get the transcript in the chat. A video response is generated to a voice in the language chosen. 

![Example of the interface]({{site.baseurl}}/assets/img/jesus_conversation.png)

You want to discover more ?

Have a try on the [HuggingFace space]() !

<iframe
    src="https://jeanmoulo-virtual-streamer-2.hf.space"
    frameborder="0"
    width="850"
    height="450"
></iframe>





## ML bricks 

The system works with the following elements : 

- Whisper for transcription
- ChatGPT with a prompt template based on the language chosen
- Text to speech
- Wav2Lip to generate the video

In this example, Whisper and ChatGPT use the OpenAI endpoint to limit the VRAM usage on the local machine.


Text to speech uses an [updated Silero server repository](https://github.com/AdMoR/silero-api-server). This offers multi lingual support without being too slow or heavy.

Wav2Lip uses the same [repository](https://github.com/devxpy/cog-Wav2Lip) as the previous post.




## System architecture  

The system can be summarized with the following diagram : 


![architecture diagram]({{site.baseurl}}/assets/img/teacher_jesus_diagram.png)


The components of the service as a whole can be on 3 types of plateforms : 
- Local : a computer with low reliability but cheap to own for a long duration
- Cloud manually setup : a vm or a server owned by a cloud manager
- Cloud fully managed service : Saas like, almost no configuration is required


Why not all cloud ?
This is specific to this project.  In order to be able to deploy it without needing a loan, some tradeoffs were needed : 
- GPU are very expensive on the cloud but kind of cheap at home 
- But some GPU service are cheap : like openAI ChatGPT
- Some cloud services are also free in their most basic service tier. But you need to adapt in order to remain under the usage limits


#### Why do you use queues ?

First question : why would you want to use a queue system for this use case ?

My guesses were the foolowing : 
- GPU ressources will be the scarcest
- Processing time can hugely vary from a few seconds up to a minute depending on the response length
- 1 Gpu may serve several users at the same time

So handling GPU requests in a queue makes sense as long as the communication is simple enough for the web servers.


#### Cloud RabbitMQ and its optimization

In order to have a scalable queue, one has many different options like AWS SQS. But I end up testing [CloudAMQP](https://www.cloudamqp.com/plans.html) because of the free plan and the RabbitMQ interface.

**Is rabbitmq important in the plan ?**
Yes it is. In fact, in the previous design. The GPU workers all share the same input queue but each client might need it own reply channel.
This is an issue for cloud ressource usage intensity.

But with RabbitMQ, there is a feature to escape this issue [Direct reply-to](https://www.rabbitmq.com/direct-reply-to.html). It allows to escape this precise limitation of the design.


#### Implementation with the pika library

After a very long crawl across the internet, I found the way to implement it with a consumer pattern rather than pure callback. This allows a better integration with the web server.

```python
import pika


def handle(channel, method, properties, body):
    message = body.decode()
    print("received:", message)


connection_ = pika.BlockingConnection()
channel_ = connection_.channel()
channel_.queue_declare(queue="test")
channel_out = connection_.channel()


with connection_, channel_, channel_out:
	# 1 - The client prepares the reply channel and send its message
    message = "hello"
    next(channel_.consume(queue="amq.rabbitmq.reply-to", auto_ack=True,
                         inactivity_timeout=1))
    channel_.basic_publish(
        exchange="", routing_key="test", body=message.encode(),
        properties=pika.BasicProperties(reply_to="amq.rabbitmq.reply-to"))
    print("sent:", message)

    # 2 - Server part with the reply-to
    (method, properties, body) = channel_out.basic_get(queue="test")
    channel_out.basic_publish(exchange="", routing_key=properties.reply_to, body="Pouet")
    channel_out.basic_ack(delivery_tag=method.delivery_tag)

    # 3 - The client listen to the reply-to and can get its answer
    for (method, properties, body) in channel_.consume(queue="amq.rabbitmq.reply-to", auto_ack=True):
        handle(channel_, method, properties, body)
```

## Conclusion 

- The ML bricks were easy to set up
- The Queue was the most difficult to set
- The latency remain high without a streaming connection with the language model
