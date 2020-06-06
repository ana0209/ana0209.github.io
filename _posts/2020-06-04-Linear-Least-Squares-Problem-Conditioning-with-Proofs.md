---
layout: post
title: "Linear Least Squares Problem Conditioning with Proofs"
date: 2020-06-04
---

### Introduction

This post was motivated by my efforts in learning Numerical Linear Algebra. As I was going through Lecture 18 about conditioning of Linear Least Squares Problem in [Numerical Linear Algebra](https://www.amazon.ca/Numerical-Linear-Algebra-Lloyd-Trefethen/dp/0898713617) by Trefethen and Bau, I decided to look up the references provided in the text in order to deepen my understanding. Once I felt I had a decent understanding of the proofs for the condition numbers and bounds I decided to [write them up]({{ site.baseurl }}/pdfs/Linear_Least_Squares_Problem_Conditioning.pdf) partly for my own sake and partly for others with similar pursuits.

The original text [[1]](#1) provides concise proofs for perturbations of $b$. However, I found that I could not follow the geometric intuition offered for perturbations of $A$. So the proofs in the [write up]({{ site.baseurl }}/pdfs/Linear_Least_Squares_Problem_Conditioning.pdf) do not rely on the geometric intuition.



### Linear Least Squares Problem

Here is the statement of linear least squares problem as it is presented in [[1]](#1):

Given $A \in \mathbb{C} ^{m \times n}$ of full rank, $m \geq n$, $b \in \mathbb{C} ^m$, find $x \in \mathbb{C} ^n$ such that $$\|b-Ax\|_2$$ is minimized.

The solution $x$ and the corresponding point $y=Ax$ that is closest to $b$ in $range(A)$ are given by:

\begin{equation}
        x=A^+b \qquad y=Pb
\end{equation}

where $A^+ \in \mathbb{C}^{n \times m}$ is the pseudoinverse of $A$ and $P=AA^+ \in \mathbb{C}^{m \times m}$ is the orthogonal projector onto $range(A)$.


### Conditioning of Linear Least Squares Problem

Here is the **Theorem 18.1** from [[1]](#1) for which I provide the proof in my [write up]({{ site.baseurl }}/pdfs/Linear_Least_Squares_Problem_Conditioning.pdf):

 *Let $b \in \mathbb{C}^m$ and $A \in \mathbb{C}^{m \times n}$ of **full rank** be fixed. The least squares problem has the following 2-norm relative condition numbers describing the sensitivities of $y$ and $x$ to perturbations in $b$ and $A$:*


   |   | $\large{x}$                              | $\large{y}$                                              |
   |--:|:-------------------------------: |:-----------------------------------------------: |
   |$\large{b}$| $\large{\frac{1}{\cos \theta}}$          | $\large{\frac{\kappa(A)}{\eta \cos \theta}}$             |
   |$\large{A}$| $\large{\frac{\kappa(A)}{\cos \theta}}$  | $\large{\kappa(A)+\frac{\kappa(A)^2 \tan \theta}{\eta}}$ |


*The results in the first row are exact, being attained for certain perturbations $\delta b$, and the results in the second row are upper bounds.*



### The Proofs

The proofs themselves are inside this [write-up]({{ site.baseurl }}/pdfs/Linear_Least_Squares_Problem_Conditioning.pdf). In the write-up I rely heavily on ["On  the  Perturbation  of  Pseudo-Inverses,  Projections  and Linear Least Squares Problems"](https://www.jstor.org/stable/pdf/2030248.pdf?casa_token=joldR17HLX4AAAAA:1KRftBiKtATt3TK3qkAIzN7re2ViU5cOpiBzTkv9Knrr9cGaT4k9vB525P1ANvlEOnMQwmwEbiNpZSSwoDE1Av9na4l_pvunCoWEnIw3jpmdgU2Dcb8jog) by Stewart.

 



### References
<a id="1">[1]</a> 
Trefethen, L., and Bau, D. Numerical Linear Algebra, SIAM, (1997).

<a id="1">[2]</a> 
Stewart, G.W. On the Perturbation of Pseudo-Inverses, Projections and Linear Least Squares Problems, SIAM Review, Vol. 19, No. 4, 634-662 (1977). 


