---
title: O Data, All Ye Faithful
categories:
  - THM
tags:
  - AoC-2023
  - Log-Analysis
---


## Challenge Information

- Advent of Cyber Day 2
- THM link [here](https://tryhackme.com/room/adventofcyber2023#)


## Explanation

This challenge is about analyzing information from excel file with python **Pandas** and **Matplotlib** library.

<br>
![](/assets/img/posts/2023-12-04-Day-2-VNC-NOLCK/2023-12-04-Day-2-VNC-NOLCK-16-05-41.png)
<br>

### Question 1: How many packets were captured (looking at the PacketNumber)?

Just run the code from example

<br>
![](/assets/img/posts/2023-12-04-Day-2-VNC-NOLCK/2023-12-04-Day-2-VNC-NOLCK-16-06-20.png)
<br>

### Question 2: What IP address sent the most amount of traffic during the packet capture?

Modify the example `COlumnName` to `Source` as this will be the IP address that send packet.

<br>
![](/assets/img/posts/2023-12-04-Day-2-VNC-NOLCK/2023-12-04-Day-2-VNC-NOLCK-16-07-49.png)
<br>

### Question 3: What was the most frequent protocol?

The code is search online after failing for a few times.

<br>
![](/assets/img/posts/2023-12-04-Day-2-VNC-NOLCK/2023-12-04-Day-2-VNC-NOLCK-16-09-33.png)
<br>

## Things I learned from the challenge

- Some python basic

