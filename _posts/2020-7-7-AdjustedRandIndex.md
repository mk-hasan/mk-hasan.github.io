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

 R = a+d/a+b+c+d

Here, R ranges from 0(no pair classified in the same way under both clusterings) t0 1(similar clustering). The term a and b can be seen as agreements and b,c as disagreements. 

 ### ARI:
 A major problem with the RI is that the expected value of Rand Index of two random cluster or partition does not take a constant value. To solve the problem , Adjusted Rand Index was introduced where the generalized hypergeometric distribution considered as the model of randomenss.

Let U and V are two random partitions with multiple cluster inside. Let nij be the numer of objects that are in both cluster ui and vj. Let ni adn nj be the number of objects or elements in cluster ui and cluster vi respectively. The notations are illustrated in the following table.



This table is called Contingency Table. You have to first create this contingency table out of two paritions. Then this table will help tp calculate the ARI coeffiecients. 

The simple term of an index with a constant expected value is 

which is boudnded by 1, and takes the value 0 when the index equals to its expected value.
With the consideration fo generalized hypergeometric model, it can be shown as:

The term (n/2) is called binomial coefficients and it was previoulst a+d. Finally, the adjusted rand index can be written as:





Now, lets go trought the code and example with much simpler way. 

## How to use?
- Python(use scikit-learn)
- R (package)



But for understanding the implementation from coding point of view, i am going to implement this with GoLang. With GoLang code we can easily understand how the ARI should be implemented in details with any OOP language. 

We need to have three class for the implementation. 

 - One for the interface
 - One for the Contingency Table creation
 - One for the calculating ARI coefficients
