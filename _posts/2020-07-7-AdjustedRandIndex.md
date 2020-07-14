---
title: ' What is Adjusted Rand Index and How it works! '
date: 2020-04-14
permalink: /posts/2020/04/blog-post-4/
tags:
  - Machine Learning
  - Clustering
  - Adjusted Rand Index
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


- It is very easy to use ARI with Python. Thanks to the Sci-kit learn , which has almost all of the machine learning algorithm. Let's look at the simple example about how to use ARI module from scikit-learn using python. 

### Python 

Perfectly matching labelings have a score of 1 even

python ```
>>> from sklearn.metrics.cluster import adjusted_rand_score
>>> adjusted_rand_score([0, 0, 1, 1], [0, 0, 1, 1])
1.0
>>> adjusted_rand_score([0, 0, 1, 1], [1, 1, 0, 0])
1.0
```

Labelings that assign all classes members to the same clusters are complete be not always pure, hence penalized:

python ```
>>> adjusted_rand_score([0, 0, 1, 2], [0, 0, 1, 1])
0.57...
```

ARI is symmetric, so labelings that have pure clusters with members coming from the same classes but unnecessary splits are penalized:

python ```
>>> adjusted_rand_score([0, 0, 1, 1], [0, 0, 1, 2])
0.57...
```
If classes members are completely split across different clusters, the assignment is totally incomplete, hence the ARI is very low:

python ```
>>> adjusted_rand_score([0, 0, 0, 0], [0, 1, 2, 3])
0.0
```



The above explanation and exale with python is so easy to use that sometimes you may forget how to code.

But for understanding the implementation from OOP coding point of view, i am going to implement this with GoLang. With GoLang code we can easily understand how the ARI should be implemented in details with any OOP language. 

We need to have three class for the implementation. 

 - One for the interface
 - One for the Contingency Table creation
 - One for the calculating ARI coefficients

I always use interface for such kind of things like any distance metrics or validation metrics. As there are multiple metrics available for this kind of task, having interface is helpful to implement multiple metrics. 

### Interface: 

```
package graph

/**
 * An abstract interface to measure the clustering performance
 * M.K Hasan
 * hasan.alive@gmail.com
 */

var ValidationFunctionRegistry = map[string]func() ValidationComputation{}

type ValidationComputation interface {
	Measure(y1, y2 []int) int
	GetName() string
}

type ValidationFunction struct {
	Computation ValidationComputation
}

func (vf *ValidationFunction) Measure(y1, y2 []int) int {
	return vf.Computation.Measure(y1, y2)
}

func (vf *ValidationFunction) GetName() string {
	return vf.Computation.GetName()
}


```

### Contingency Table

```
package validation

import (
	"fmt"
)

/**
 * The contigency table of the two cluster
 * M.k Hasan
 * hasan.alive@gmail.com
 */
type CTableAttribute struct {
	n                                  int /* total number of observation */
	NumberOfCluster1, NumberOfCluster2 int /** Total number of cluster in first and second cluster **/
	/** two dimensional contingency table */
}

/** creating the contingency table **/
func (e *CTableAttribute) ContingencyTable(y1, y2 []int) ([][]int, []int, []int) {

	var table [][]int

	if len(y1) != len(y2) {
		fmt.Errorf("length of the two cluster elements should be equal")
	}

	for i := 0; i < e.NumberOfCluster1; i++ {
		for j := 0; j < e.NumberOfCluster2; j++ {
			nij := 0
			for k := 0; k < e.n; k++ {
				if i == y1[k] && j == y2[k] {
					nij = nij + 1
				}
			}
			table[i][j] = nij
		}

	}
	var a []int //rowsum
	var b []int //columnsum

	for i := 0; i < e.NumberOfCluster1; i++ {
		for j := 0; j < e.NumberOfCluster2; j++ {
			a[i] = a[i] + table[i][j]
			b[i] = b[i] + table[i][j]
		}
	}
	return table, a, b
}

```

### Calculate functions

```
package validation

import (
	"gonum.org/v1/gonum/stat/combin"
	g "ki/graph"
)

func init() {
	g.ValidationFunctionRegistry["ARI"] = func() g.ValidationComputation {
		return &ARI{}
	}
}

type ARI struct {
	CTable CTableAttribute
}

func (e *ARI) Measure(y1, y2 []int) int {
	return e.CalculateRand1(y1, y2)
}

/** Calculate the adjusted Rand Index **/

func (e *ARI) CalculateRand1(y1, y2 []int) int {
	var rand1 int
	count, a, b := e.CTable.ContingencyTable(y1, y2)
	for i := 0; i < e.CTable.NumberOfCluster1; i++ {
		for j := 0; j < e.CTable.NumberOfCluster2; j++ {
			if count[i][j] >= 2 {
				rand1 += combin.Binomial(count[i][j], 2)
			}
		}
	}
	rand2a := 0
	for i := 0; i < e.CTable.NumberOfCluster1; i++ {
		if a[i] >= 2 {
			rand2a += combin.Binomial(a[i], 2)
		}
	}

	rand2b := 0
	for j := 0; j < e.CTable.NumberOfCluster2; j++ {
		if b[j] >= 2 {
			rand2b += combin.Binomial(b[j], 2)
		}
	}

	rand3 := rand2a * rand2b
	rand3 = rand3 / combin.Binomial(e.CTable.n, 2)
	randN := rand1 - rand3

	// D
	rand4 := (rand2a + rand2b) / 2
	randD := rand4 - rand3

	rand := randN / randD
	return rand

}

func (e *ARI) GetName() string {
	return "ARI"
}


```


