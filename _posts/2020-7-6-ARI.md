---
layout: post
title: What is Adjusted Rand Index and How it works! 
---


> Adjusted Rand Index(ARI) is one of the widely used metrics for validating clustering performance

>Rand Index(RI) and Adjusted Rand index(ARI) is different.

>ARI is easy to implement and needs ground truth to execute

## *Let's Talk about ARI in details....*

# Waht can we learn from this article?

 - What is ARI?
 - Where to use ARI?
 - How to code ARI?
 - Why to use ARI?
 - Math Behind ARI.

 ## What is ARI?
 Before talking about the ARI, we need to go through the Rand Index(RI) first.

 ### Rand Index:
 Rand Index was motivated by the idea of comparing one classifcation result to an another classification result. The naive theory for  measuring the performance of a classification would be calculating the correctly classified items to all items. For RI, the above mention performance measre theory was extened to the comparing two cluster. Here, instead of counting single elemsnts, RI counts the correctly classied pairs of items. Hence, we can have the definition of Rand Index from the mathematical point of view as:

 

 Here, R ranges from 0(no pair classified in the same way under both clusterings) t0 1(similar clustering). 

 ### ARI:
 The  

