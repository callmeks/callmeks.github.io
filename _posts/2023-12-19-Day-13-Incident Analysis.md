---
title: To the Pots, Through the Walls 
categories:
  - THM
tags:
  - AoC-2023
  - Intrusion-Detection
---

## Challenge Information

- Advent of Cyber Day 13
- THM link [here](https://tryhackme.com/room/adventofcyber2023#)


## Explanation

Today's challenge is more on theory where it explain about Incident Analysis, firewall and honeypot.

### Question 1: Which security model is being used to analyse the breach and defence strategies?

The answer is `Diamond Model`.

### Question 2: Which defence capability is used to actively search for signs of malicious activity?

The answer is `Threat Hunting`.

### Quetsion 3: What are our main two infrastructure focuses? (Answer format: answer1 and answer2)

The answer is `firewalls and honeypots`.

### Question 4: Which firewall command is used to block traffic?

The answer is `deny`.

### Question 5: There is a flag in one of the stories. Can you find it?

To get the flag, just went to their website which should be running in port 8090 and get the flag.

flag:`THM{P0T$_W@11S_4_S@N7@}`

## Things I learned from the challenge

- Incident Analysis
- Setting up firewall with `ufw`
- creating honeypot with `pentbox`