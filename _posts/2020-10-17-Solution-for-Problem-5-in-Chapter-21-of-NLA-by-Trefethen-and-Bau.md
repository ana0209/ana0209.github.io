---
layout: post
title: "Solution for Problem 5 in Chapter 21 of NLA by Trefethen and Bau"
date: 2020-10-17
---

**Disclaimer**: All of this is best effort. Do let me know if you spot any mistakes.

I initially answered this on [math.stackexchange.com](https://math.stackexchange.com/questions/3439214/asking-for-a-solution-on-problem-21-5-in-numerical-linear-algebra-by-lloyd-n-tr/3813645#3813645), but then decided to add it to my blog.

(a) The problem asks to describe a strategy of symmetric pivoting that preserves the hermitian structure of the matrix while still leading to a lower unit triangular matrix with entries $\|l_{ij}\| \leq 1$.

It is possible to obtain a decomposition $PAP^T=LDL^T$ where $D$ is a tridiagonal matrix and $L$ is a lower triangular matrix with a unit diagonal and all off-diagonal entries having an absolute value $\leq 1$. I found the idea and the algorithm to do this in [this paper](https://link.springer.com/article/10.1007/BF01931804).

By using symmetric pivoting, one can only interchange the position of diagonal entries and it is not possible to put an off-diagnonal entry on the diagonal. Aasen [proposed](https://link.springer.com/article/10.1007/BF01931804) to use symmetric pivoting to pivot the largest off diagonal element in the column to the location just below the diagonal entry. So, at step $i$ we put the largest off diagonal element of column $i$ in the position $(i+1,i)$ and then eliminate all the subsequent entries $(i+2:m,i)$ in the column using that element as the pivot. 

Let’s say we have: 
$$\begin{eqnarray} A=\begin{pmatrix}1 & 2  & 3 & 4 \\ 2 & 3 & 5 & 6 \\ 3 & 5 & 4 & 9 \\ 4 & 6 & 9 & 7\end{pmatrix}\end{eqnarray}$$ 

The largest off diagonal element of matrix $A$ is $A(4,1)=4$. In the first step $A(4,1)$ is pivoted to position $(2,1)$ by using a permutation matrix: $$\begin{eqnarray}P_1AP_1=\begin{pmatrix}1 & 0 & 0 & 0 \\ 0 & 0 & 0 & 1\\ 0 & 0 & 1 & 0 \\ 0 & 1 & 0 & 0\end{pmatrix} \begin{pmatrix}1 & 2  & 3 & 4 \\ 2 & 3 & 5 & 6 \\ 3 & 5 & 4 & 9 \\ 4 & 6 & 9 & 7\end{pmatrix}\begin{pmatrix}1 & 0 & 0 & 0 \\ 0 & 0 & 0 & 1\\ 0 & 0 & 1 & 0 \\ 0 & 1 & 0 & 0\end{pmatrix} =\begin{pmatrix}1 & 4 & 3 & 2\\4 & 7 & 9 & 6\\3 & 9 & 4 & 5\\2 & 6 & 5 & 3\end{pmatrix}\end{eqnarray}$$

Now we can construct matrix $L_1$: $$\begin{eqnarray}L_1=\begin{pmatrix}1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & -3/4 & 1 & 0 \\ 0 & -1/2 & 0 & 1\end{pmatrix}\end{eqnarray}$$
that can be used to set entries $(3,1)$ and $(4,1)$ of $P_1AP_1$ to $0$:
$$\begin{eqnarray} L_1P_1AP_1L_1^T=\begin{pmatrix}1 & 4 & 0 & 0\\4 & 7 & 3.75 & 2.5\\0 & 3.75 & -5.5625 & -1.375 \\ 0 & 2.5 & -1.375 & -1.25\end{pmatrix} \end{eqnarray}$$

This step is then repeated for the subsequent columns for a total of $m-2$ times for an $m \times m$ matrix. 

The main thing to notice here is that each $L_i$ has non-zero subdiagonal entries in column $(i+1)$ only and that each $P_i$ pivots the largest subdiagonal element in column $i$ to position $(i+1,i)$ instead of to the diagonal position $(i,i)$.

Below is the pseudocode for the above described algorithm that applies the permutation and the symmetric elimination on the full matrix at each step. For full implementation in Octave see [here](https://gist.github.com/ana0209/e51971fdecc27c1f0588f91a6de37345).

```
input: A, mxm matrix
output: L, D, P

L=I %identity matrix
D=tril(A) %lower triangular part of A
for i=1:m-2
    [col_max, r] = max(abs(D(i+1:m,i))); 
   
    % if the larget element is 0
    if col_max == 0
        continue
    endif
 
    r=r+i
    Pi=permute(I,i+1,r)
    L=Pi*L*Pi

    %symmetric swap on a triangular matrix
    D=symmetricRowAndColumnSwap(D,i+1,r) 
    L(i+2:m,i+1)=D(i+2:m,i)/D(i+1,i)

    % eliminating the i-th column
    for j=m:-1:i+2
        D(j,i)=0
        D(j,i+1:j)=D(j,i+1:j)-D(i+1:j,i+1)'*Li(j,i+1)
    endfor

    % changes due to elimination of the i-th row
    for j=i+2:m
        D(j:m,j)=D(j:m,j)-Li(j,i+1)*D(j:m,i+1)
    endfor

    P=Pi*P
endfor

D=D+tril(D,-1)’

```


It takes $\approx 2/3m^3$ flops to execute the algorithm as it is done in the pseudocode above. However, Aasen implements the update differently, by updating a column at a time at each step. This requires $\approx m^3/3$ flops. Apart from the original paper I also used [this presentation](ftp://ftp.numerical.rl.ac.uk/pub/talks/jkr.strathclyde.24VI09.pdf) to help me implement column by column update and understand the complexity of the algorithm. 

The implementation of the faster, column by column by column update is [here](https://gist.github.com/ana0209/e655c111c694aeaf2ec315b79be0a78a).

This version takes advantage of the fact that at each step $i$ we have all the information we need to update the column $i+1$ in the triangular matrix. Below you can find the explanation of this faster method. For the sake of clarity,  I skip the permutations in the explanation and focus on the column update.

After performing the algorithm for $i$ steps we know matrices $L_i$ that do elimination at each step. 

We can write:
$$\begin{eqnarray}A=L_1^{-1} \cdots L_i^{-1} D^{(i)} (L_i^{-1})^T \cdots (L_{1}^{-1})^T\end{eqnarray}$$
$$\begin{eqnarray}L^{(i)}_{m \times m}=L_1^{-1} \cdots L_i^{-1} =\begin{pmatrix} L_{11} & 0  \\ L_{21} & L_{22}^{(i)} \end{pmatrix}\end{eqnarray}$$

with $L_{22}^{(i)}= \begin{pmatrix} l_{i+1} & 0 \end{pmatrix}_{(m-i) \times (m-i)}$

$L^{(i)}$ is the decomposition matrix after step $i$. $l_{i+1}$ is a column vector of dimension $(m-i) \times 1$. $L_{11; (i \times i)}$ is a lower unit triangular matrix and $L_{21; (m-i) \times i}$ is a rectangular matrix with potentially all non zero entries (except for the off-diagonal entries in the first column).

We can factor A and $D^{(i)}$ in a similar manner:
$$\begin{eqnarray}A=\begin{pmatrix} A_{11(i \times i)} & A_{21(i \times m-i)}^T \\ A_{21(m-i \times i)} & A_{22(m-i \times m-i)}  \end{pmatrix}\end{eqnarray}$$

$$\begin{eqnarray}D^{(i)}=\begin{pmatrix} D_{11(i \times i)} & D_{21(i \times m-i)}^T \\ D_{21(m-i \times i)} & D_{22(m-i \times m-i)}^{(i)} \end{pmatrix}\end{eqnarray}$$
where $D_{11}$ is a tridiagonal block of the final matrix, $D_{21}$ is also a block of the final matrix $D$ and it has only one non-zero value at the intersection of its first row and last column and $D_{22}^{(i)}$ is the part that still needs to be reduced further after step $i$.

We have:
$$\begin{eqnarray}\begin{pmatrix}A_{11} & A_{21}^T \\ A_{21} & A_{22}\end{pmatrix}=\begin{pmatrix} L_{11} & 0 \\L_{21} & L_{22}^{(i)} \end{pmatrix} \cdot \begin{pmatrix} D_{11} & D_{21}^T \\ D_{21} & D_{22}^{(i)} \end{pmatrix} \cdot \begin{pmatrix} L_{11}^T & L_{21}^T \\ 0 & L_{22}^{(i)T} \end{pmatrix}\end{eqnarray}$$ 

From there we get:
$$\begin{eqnarray}A_{22}=L_{21}D_{11}L_{21}^T + L_{22}^{(i)}D_{21}L_{21}^T + L_{21}D_{21}^TL_{22}^{(i)T} + L_{22}^{(i)}D_{22}^{(i)}L_{22}^{(i)T}\end{eqnarray}$$

and
$$\begin{eqnarray}L_{22}^{(i)}D_{22}^{(i)}L_{22}^{(i)T} = A_{22}-L_{21}D_{11}L_{21}^T - L_{22}^{(i)}D_{21}L_{21}^T - L_{21}D_{21}^TL_{22}^{(i)T}\end{eqnarray}$$

From the equation above we can get the first column of $L_{22}^{(i)}D_{22}^{(i)}L_{22}^{(i)T}$. What we really want is the first column of $D_{22}^{(i)}$. Luckily we have the information we need to calculate that column. $L_{22}^{(i)}$ has only one column with non-zero off-diagonal entries, its first column. This happens to be the non-zero part of the $(i+1)$-st column of $L_i^{-1}$, the inverse of the reduction matrix used to create zeros in the $i$-th column of the reduced matrix at step $i$. We can use this to get the first column of $D_{22}^{(i)}$ which is all we need to construct $L_{i+1}$ and continue the above described column by column process.

For exact operations, here is the code snippet that does the column update at step $i$:

```
L(i+2:m,i+1)=D(i+2:m,i)/D(i+1,i);
D(i+2:m,i)=0;

v=zeros(i,1);
if i > 1
	v(1)=D(1,2)*L(i+1,2);
	for j=2:i-1
		v(j)=D(j,j-1:j)*L(i+1,j-1:j)'+D(j+1,j)*L(i+1,j+1);
	endfor
	v(i)=D(i,i-1:i)*L(i+1,i-1:i)';
	endif

D(i+1:m,i+1)=D(i+1:m,i+1)-L(i+1:m,1:i)*v-L(i+1:m,i+1)*D(i+1,i)*L(i+1,i)-L(i+1:m,i)*D(i+1,i);
D(i+2:m,i+1)=D(i+2:m,i+1)-L(i+2:m,i+1)*D(i+1,i+1);
```

(b) The form the factorization takes is $PAP^T=LDL^T$ where $P$ is a permutation matrix and L is a lower unit triangular matrix with its first column being all $0$s apart from the 1 on the diagonal. Also, all the subdiagonal entries of $L$ have absolute value less than 1. D is a tridiagonal symmetric matrix.

(c) The first implementation of the algorithm requires $\approx 2/3m^3$ flops and the second half of that. 

From the pseudocode for the first version of the algorithm we see that each entry update requires one multiplication and one addition. There are $\approx (m-i-1)\cdot(m-i)/2$ entries to be updated when taking advantage of the symmetry which comes to $\approx (m-i)^2/2 \cdot 2=(m-i)^2$ operations for column clearing and the same amount for row clearing. 
The update happens $m-2$ times so we have:
$$\begin{eqnarray}\sum_{i=1}^{m-2}2(m-i)^2 \approx \sum_{k=1}^{m}2k^2\approx 2m^3/3\end{eqnarray}$$  

The column based update from [Aasen](https://link.springer.com/article/10.1007/BF01931804) requires $\approx m^3/3$ flops. The code snippet above calculates a vector $v$ first which is the first column of $D_{11}L_{21}^T$ so $v=D_{11}L_{21}(1,:)^T$. Since each row of $D_{11}$ has at most $3$ non-zero values, calculating $v$ takes $\approx 5i$ operations (at most 3 multiplications and 2 additions for each entry of $v$).

We only need the first column of $L_{21}T_{11}L_{21}^T$ which is equal to $L_{21}v$. Calculating this column takes $\approx 2i(m-i)$ flops.

We also need the first columns of $L_{22}^{(i)}D_{21}L_{21}^T$ and $L_{21}D_{21}^TL_{22}^{(i)T}$. Since $D_{21}$ only has one non-zero entry calculating each of these columns takes $\approx 2(m-i)$ flops.

To this we need to add the subtractions of the $3$ elements from the first column of $A_{22}$ which comes to $\approx{3(m-i)}$ flops and the operations required to get the first column of $D_{22}$ from $L_{22}^{(i)}D_{22}^{(i)}$. This takes $\approx 2(m-i)$ operations.

There are $m-1$ steps and all the sums of the other factors except for $L_{21}v$ are of order $m^2$. So the number of flops is dominated by the sum for the term $2i(m-i)$:
$$\begin{eqnarray}\sum_{i=1}^{m-1}2i(m-i)=2m\sum_{i=1}^{m-1}i - 2\sum_{i=1}^{m-1}i^2 = m^3/3-m/3 \approx m^3/3\end{eqnarray}$$

From this we see that the number of flops for the column by column elimination is around $m^3/3$.

