---
layout: post
title:  "Documenting the architecture"
date:   2024-08-12 18:18:22 +0800
categories: software-engineering
published: false
---
One of the most painful things to do as a Software Engineer is to join a new job. At first, you are given an onboarding path that may or may not have been designed with the idea of making you successful in understanding how things run here. 

The problem is that this onboarding path most of the time has been designed by people who know a little bit too much about things and might leave some key information behind as they might be deemed not important. That's why everything should be documented. 

Everything? Yes, everything. From the overall architecture, to some key buisiness rules everything needs to be documented
on a wiki. The reason behind it?
- Wiki is far more searchable than JIRA
- You may think this is not important but the guy in the future most likely thinks it is
- Having to go to one teammate who then redirects you to another teammate and at this end, this person doesn't even know the answer and the person who knew the answer is gone... 
- You are as good as a team as your weak link, if your weakest links is your documentation you know for sure it will blow up later down the road

You need to have strong documentation, which clearly states the:
- The intention of the document
- Step by step - as if the other person didn't know anything about the topic - with commands ready to be run
- Meaningful screenshots 

All of this seems to be trivial but I can assure you that even your self-documenting code 5 years down the road might not be understood by your future teammate as your
team went through a turnover on multiple sides and now you lost the reason why things are made that way. If you don't know why things have been done that way, especially if you have no idea who is consuming the function/API you are blind-sided. You may think that by applying the best practices you will get away with it, but most of the time reality will knock on the door and realise that perhaps for that particular thing you were working on, the consumer expected things differently. 

Everyday as a Software Engineer, we are subjected to having to make trade-offs - sometimes heavy ones - and we have to understand the consequences of these and if we can afford to take the gamble. Sometimes it will pay off with dividends and sometimes it will not, but you have to edge this with an overall knowledge which you need to document in order to have your teammate on board with the script. 

When writing a software as a team, everyone should know all the decisions made and why they were made. This helps tremendously at design, development, testing and validation time. Everyone will benefit from it, a team is like a hive and if we are not functioning as a hive we are in survival mode until either we get high turnover which will costs the company a lot of goals missed or we fix this communication breakdown and allow people to grow to senior, staff roles. 

As someone who is aspiring to reach a more senior role, one of my qualities is to be able to reverse-engineer and produce documentation based on that. But if I have to do it this means that before the team was keeping this knowledge for themselves and that's not good for the team nor the business. Documenting allows the business to move forward and be able to pivot, the same way as one would when writing an extensive test suite for software. You might think it is a waste of time but this investment will pay dividends when refactoring or onboarding new people as all the contracts are locked in and we know what can be changed and what cannot. 

Also, I would like to say this to conclude this short article, I can say for sure that some of my best wins at work came from wikis that sometimes were written by another team within the group. That's the power of the hive, once it is registered everyone has access to it on command. So please document everything!