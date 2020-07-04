---
layout: post
title: "On Product and Difference of Orthogonal Projectors"
date: 2020-06-03
---

I spent some time studying the conditioning of the linear least squares problem. As part of my efforts I learned a few facts about the products of orthogonal projectors and their difference. This post sums up the facts I learned and gives proofs for them. It was not easy for me to extract the proofs from the literature so once I had them in a satisfactory shape, I decided to [write them up]({{ site.baseurl }}/pdfs/Proofs_for_Stewart_1975.pdf) and post them. Hopefully someone else finds it useful. 

The theorems were taken from ["On  the  Perturbation  of  Pseudo-Inverses,  Projections  and Linear Least Squares Problems"](https://www.jstor.org/stable/pdf/2030248.pdf?casa_token=joldR17HLX4AAAAA:1KRftBiKtATt3TK3qkAIzN7re2ViU5cOpiBzTkv9Knrr9cGaT4k9vB525P1ANvlEOnMQwmwEbiNpZSSwoDE1Av9na4l_pvunCoWEnIw3jpmdgU2Dcb8jog) by Stewart and the lemma was taken from one of its references ["Perturbation Theory for Pseudo-Inverses"](https://link.springer.com/article/10.1007/BF01933494) by Wedin. The proofs were put together from the references in the Stewart's work [[1]](#1) and supported by classical Linear Algebra text books. Full set of references I used can be found at the end of my [write-up]({{ site.baseurl }}/pdfs/Proofs_for_Stewart_1975.pdf).

Here I will just put the main theorems proven in the document. For the proofs [click here]({{ site.baseurl }}/pdfs/Proofs_for_Stewart_1975.pdf):


**Theorem 1.** *(Theorem 2.3, part 1 in [[1]](#1))*
*For any matrix $A$ and $B$ the following statement is true:* 

*Let $P_A$ be orthogonal projector onto the range of $A$ and let $P_B$ be orthogonal projector onto the range of $B$.* 
*If* $rank(A) = rank(B)$: 


*1. The singular values of $P_AP_B^\bot$ and $P_BP_A^\bot$ are the same.*

*2. The nonzero singular values $\sigma_i$ of $P_AP_B^\bot$ correspond to pairs $\pm\sigma_i$ of eigenvalues of $P_B-P_A$, so that:*
$$\begin{eqnarray}
\nonumber \\
\|P_B-P_A\|_2=\|P_AP_B^\bot\|_2=\|P_BP_A^\bot\|_2
\end{eqnarray}$$



**Theorem 2.** *(Theorem 2.3, part 2 in [[1]](#1))*
*For any two matrices $A_{m \times n}$ and $B_{m \times l}$ and two orthogonal projectors $P_A$ and $P_B$ onto the ranges of $A$ and $B$ respectively we have:*
$$\begin{eqnarray}
\nonumber \\
\|P_B-P_A\|_2<1\Rightarrow rank(A)=rank(B)
\end{eqnarray}$$


**Lemma 1.** *(Lemma 7.1 in [[2]](#2))*
*Let $X$ and $Y$ be subspaces of $\mathbb C^n$ and $P_X$, $P_Y$ orthogonal projections onto $X$ and $Y$, respectively. $\sigma_i$ is a singular value of $P_XP_Y$ if and only if $\sigma_i^2$ is an eigenvalue of $P_XP_Y$.* 


### References
<a id="1">[1]</a> 
Stewart, G.W. On the Perturbation of Pseudo-Inverses, Projections and Linear Least Squares Problems, SIAM Review, Vol. 19, No. 4, 634-662 (1977). 

<a id="1">[2]</a>
Wedin, P. Perturbation theory for pseudo-inverses. BIT 13, 217–232 (1973). 