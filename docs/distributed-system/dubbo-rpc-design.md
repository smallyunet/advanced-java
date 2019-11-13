## Interview questions
How to design a Dubbo like RPC framework?

## Psychnological analysis of interviewers
To be honest, this question is the same as asking you how to design an MQ by yourself. There are two question:
- Do you have very deep understanding of the principles of an RPC framework.
- Can you think about how to design an RPC framework and test your system design ability as a whole.

## Analysis of interview questions
In fact, when I ask you this question, at least you can't recognize it. Because it's knowledge literacy, I can't explain to you in depth what Kafka source code analysis and Dubbo source code analysis are. Besides, I'll tell you that you should really digest, understand and absorb it at least a month later. 

So I'd like to give you a suggestion. If you encounter this kind of problem, at least start with the principle of similar framework you know, and talk about the principle of Dubbo by yourself. Let's design it. For example, isn't there so many layers in Dubbo? And what does each layer do? Do you know? Let's talk about it according to this idea. At least you can't be fooled. It's better than those who come up and don't know what to say.

For example, let me give you a simple answer.
- You have to go to the registration center to register your services. Do you have to have a registration center to keep the information of each service? You can do it with zookeeper, right.
- Then your consumers need to go to the registration center to get the corresponding service information, right, and each service may exist on multiple machines.
- Then you should make a request. How> Of course, it is based on the dynamic agent. You can obtain a dynamic agent for the interface. The dynamic agent is a local agent of the interface, and then the agent will find the corresponding machine address of the service.
- Which machine can I find to send the request? There must be a load balancing algorithm, for example, the simplest one can be random polling.
- Then find a machine and send a request with it. How to send the first question? You can say that netty is used, NIO way; the second question is what format data is sent? You can say that we use Hessian serialization protocol, or something, right. Then the request passed.
- The server side is the same. You need to generate a dynamic agent for your own service, listen to a certain network port, and then proxy your local service code. When you receive a request, call the corresponding service code, right.

This is the most basic idea of RPC framwork. Let's not say how powerful your technical skills are, even if you give the simplest idea first?