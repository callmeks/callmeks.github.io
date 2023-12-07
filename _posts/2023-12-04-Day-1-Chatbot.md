---
title: Chatbot, tell me, if you're really safe?
categories:
  - THM
tags:
  - AoC-2023
  - Machine-Learning
---

## Challenge Information

- Advent of Cyber Day 1
- THM link [here](https://tryhackme.com/room/adventofcyber2023#)

## Explanation

> This challenge is guided as it is provided by THM
{: .prompt-info }

This challenge is about machine learning and the challenge is basically getting sensitive information from chatbot.

![](/assets/img/posts/2023-12-04-Day-1-Chatbot/2023-12-04-Day-1-Chatbot-15-21-58.png)

### Question 1: What is McGreedy's personal email address?

To solve this challenge, I started out by getting the full name of McGreedy. There is a cheatsheet for specific keywords.

![](/assets/img/posts/2023-12-04-Day-1-Chatbot/2023-12-04-Day-1-Chatbot-15-23-27.png)

After getting the full name, I lied to the chatbot to get his personal email.

![](/assets/img/posts/2023-12-04-Day-1-Chatbot/2023-12-04-Day-1-Chatbot-15-25-23.png)

### Question 2: What is the password for the IT server room door?

I started out by asking out the chatbot.


![](/assets/img/posts/2023-12-04-Day-1-Chatbot/2023-12-04-Day-1-Chatbot-15-27-03.png)

It mentioned that I must be a member of IT to do so. Based on the previous information, I have the name of the member of IT. I could just act as the member and get the password.

![](/assets/img/posts/2023-12-04-Day-1-Chatbot/2023-12-04-Day-1-Chatbot-15-28-35.png)

### Question 3: What is the name of McGreedy's secret project?

To get the secret project, we need to trick the chatbot to be in maintenance mode.

![](/assets/img/posts/2023-12-04-Day-1-Chatbot/2023-12-04-Day-1-Chatbot-15-30-02.png)

## Things I learned from the challenge

- prompt injection

