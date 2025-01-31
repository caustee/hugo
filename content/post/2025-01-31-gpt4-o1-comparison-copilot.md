---
date: "2025-01-31T00:00:00Z"
title: Microsoft announced Think Deeper, free access to GPT o1 model
---
So recently Microsoft [announced](https://www.theverge.com/news/603149/microsoft-openai-o1-model-copilot-think-deeper-free) they're offering ChatGPT o1 model free to their Copilot program. So I've decided to give it a go and see how it goes.

First I've connected to https://copilot.microsoft.com with both a personal account and a work account.
I've tested the announcement this by asking it directly:

![targets](/images/personal-ask.png)

![targets](/images/work-ask.png)

What we can see quickly is that my company-linked account has to update to latest Copilot because it still uses GPT4 as a LLM. While the official Copilot uses the latest model. Even though it doesn't respond directly like its previous model.

Then I wanted to see the difference and gave them both the same task:
```
Give me a simple Ansible code that will deploy nginx with a standard 'hello world' message in index.html. The server that should run nginx has IP: 10.10.10.10. I just want you to create the necessary ansible files for deployment (host vars, inventory, playbook, tasks or roles if needed.
```

I've tried to include some specific requests (hello world, server IP address), gave it some options to see if it takes the easiest route or it expands it a bit (host vars, playbook, tasks or roles).

I was expecting to find a standard cookie-cutter example. And GPT4 didn't dissapoint me :) 

Here is GPT4's answer:

![targets](/images/gpt4-first.png)
![targets](/images/gpt4-second.png)

And here is o1's answer:

![targets](/images/gpt-o1-first.png)
![targets](/images/gpt-o1-second.png)
![targets](/images/gpt-o1-third.png)
![targets](/images/gpt-o1-forth.png)

We can clearly see the difference between the responses. GPT4 gave me a short standard example that couldn't work if applied directly.  Even the instructions provided are insufficient as no Ansible Controller could be able to connect to a managed node without ssh access.
This, of course, is mentioned in the lenghty response that o1 provided. Beside many other things, like providing installation tasks for most popular linux flavors and creating a better formatted html page. And it includes tips on how to improve it.

Of course, all this extra information comes with a cost. In this case, it was time. Because the time took to generate the output for o1 was much longer than GPT4's. But if we take into account also the time taken to actually make the code from GPT4 work I would say o1 was faster :)

Anyway, AI is here to stay and it will have a dramatic impact on tech industry overall. As Meta's CEO said, all current mid-level engineers will be replaced by AIs :mechanical_arm:. Juniors will be left to handle mundane boring non-automatable activities and Seniors will be left to provide guidance and policies for AI models that will handle all the load. So if you're not part of the Seniors team it is time to level-up quickly.
