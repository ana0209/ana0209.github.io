---
layout: post
title: "Solutions for Chapter 18 in NLA by Trefethen and Bau"
date: 2020-06-25
---
Disclaimer: All of this is best effort. Feel free to let me know if you spot any mistakes.


### Exercise 18.1.

I do not provide solutions for parts (a) and (b). I have calculated answers (c)-(e) by writing this [Octave script](https://gist.github.com/ana0209/e05eb0907ac27bdb72507420279b1b34).

(c) These numbers are obtained by using the definitions (18.3)-(18.5) in [[1]](#1): 
* $\kappa(A)=42429.23542$, 

* $\theta=0.68470$, 

* $\mu=1.0$

(d) These condition numbers are obtained by using formulas from Theorem 18.1.:
* $\kappa_{b \rightarrow y}=1.2910$

* $\kappa_{b \rightarrow x}=54775.17704$

* $\kappa_{A \rightarrow y}=54775.17707$

* $\kappa_{A \rightarrow x}=1469883252.86400$

(e) The perturbation $\delta b$ that relative error equal to $\kappa_{b \rightarrow y}$ is any perturbation in the range of $A$. IWe can get such perturbation by multiplying any non-zero vector $$v$$ by a projector into the range of $A$: $\delta b=AA^+v$. I used $\delta b=AA^+[1 \; 1 \; 1]\'$ and got relative error $err_{b\rightarrow y}=1.2910$.  You can find the code for this as for any subsequent error calculation in this [script](https://gist.github.com/ana0209/e05eb0907ac27bdb72507420279b1b34).
 
The perturbation $\delta b$ that achieves relative error of x equal to the condition number $\kappa_{b \rightarrow x}$ is one where $$\|A^+\delta b\|_2=\|A^+\|_2\|\delta b\|_2$$. We now construct that kind of $\delta b$. 

We know that $$\|A^+\|_2=1/\sigma_n$$ where $$\sigma_n$$ is the n-th and smallest singular value of $A$. We also know that $$A^+=V\Sigma^+U^*$$ (see [Wikipedia entry on Moore-Penrose pseudoinverse](https://en.wikipedia.org/wiki/Moore–Penrose_inverse#Singular_value_decomposition_(SVD))). $\Sigma^+$ is a diagonal matrix with i-ith diagonal entries being $1/\sigma_i$ for non-zero singular values $\sigma_i$ of $A$ and zeros elsewhere. In our particular case where $n=2$ and $m=3$. If we choose $\delta b=U[0 \; 1 \; 0]'$ we get:


$$\begin{eqnarray}\|A^+\delta b\|_2 &=& \|V \Sigma^+U^*U\begin{bmatrix}0 \\ 1 \\ 0\end{bmatrix}\|_2 \nonumber \\ 
&=&\|\Sigma^+ \begin{bmatrix}0 \\ 1 \\ 0\end{bmatrix}\|_2 \nonumber \\ &=& \frac{1}{\sigma_2}\|\begin{bmatrix}0 \\ 1 \\ 0\end{bmatrix}\|_2 \nonumber \\ &=& \|A^+\|_2\|U \begin{bmatrix}0 \\ 1 \\ 0\end{bmatrix}\|_2 \nonumber \\ &=& \|A^+\|_2\|\delta b\|_2 
\end{eqnarray}$$

Above we use the fact that $$\| \cdot \|_2$$ norm is unitarily invariant (the norm of the vector does not change when it is multiplied by a unitary matrix). The key here was to set a non-zero value in the n-th entry of the vector $U$ is multiplied by. That way we select the $$1/\sigma_n=1/\sigma_2$ value from $\Sigma^+$$.

When using the above described perturbation this error was obtained: $$err_{b \rightarrow x}=54775.17704$$.

While the previous two errors achieved the condition number exactly, the following two will do so only approximately.

$$y$$ only depends on the $range(A)$. From [[1]](#1), *Tilting the range section* we see that the biggest tilt of the $range(A)$ for some fixed value of $$\|\delta A\|_2$$ is achieved when $$\delta A=\delta p \cdot v_n$$ where $v_n$ is the right singular vector of $A$ associated with its smallest singular value $\sigma_n$ wich in the case of this problem is $\sigma_2$. $\delta p$ is a vector orthogonal to the range of $A$. I calculated $\delta p$ as:

$$\begin{eqnarray}
\delta p = (I-AA^+)\begin{bmatrix}0 \\ 1 \\ 0\end{bmatrix}/10^6
\end{eqnarray}$$

And the final perturbation of $$A$$ is $$\delta A=\delta p v_2$$ where $v_2$ is obtained as the second column of $V$ from $A=U \Sigma V^*$.

The error obtained using this perturbation was $err_{A \rightarrow y}=34639.57287$. The difference between $err_{A \rightarrow y}$ and $\kappa_{A \rightarrow y}$ can be explained by the value of $$y=[2 \; 2.0001 \; 2.0001]'$$. Let us look at the left singular vectors of $A$ in $U$:

$$\begin{eqnarray}U=\begin{bmatrix}-5.77e-01 & 8.17e-01 & -8.76e-17 \\ -5.77e-01 & -4.08e-01 & -7.07e-01 \\ -5.77e-01 & -4.08e-01 & 7.07e-01 \end{bmatrix}\end{eqnarray}$$

We see that $y$ is approximately aligned with the first left singular vector. This means it is roughly perpendicular to the plane in which the tilt for the described perturbation is made. As explained in [[1]](#1) this reduces the error by a factor of $\sin \theta$. Indeed we get $$err_{A \rightarrow y} \cdot \sin \theta = 54771.06913$$ which is very close to $$\kappa_{A \rightarrow y}$$.

You might ask why not tilt the range in the plane that contains $y$. The answer is that due to big difference between $\sigma_1$ and $\sigma_2$, the angle of the tilt of the range while keeping the same $$\|\delta A\|_2$$ is much smaller resulting in an error that is significantly less than the above cacluclated $$err_{A \rightarrow y}$$ and $$\kappa_{A \rightarrow y}$$.

The final error $$err_{A \rightarrow x}$$ has two terms, one that depends on changes of $A$ that do not alter its range and the other term that is affected by tilting of the range of $A$. The first term is $\kappa(A)$ and the second $$\kappa(A)^2\tan\theta/\mu$$. In our example $\mu=1.0$ so the overall error is much more affect by tilting the range than the changes in the mapping of points within in the range, since that term contains the quadratic factor $\kappa(A)^2$. Therefore I picked the same perturbation as for $$err_{A \rightarrow y}$$ since that is the perturbation that causes the largest tilt of the range of $$A$$ for a given $$\|\delta A\|_2$$. The obtained relative error is $$err_{A \rightarrow x}=1469620366.28139=0.9982 \cdot \kappa_{A \rightarrow x}$$.

### Exercise 18.2.

If many variables are used, some of them may be highly correlated. For example, parents' level of education and parents' annual income are likely heavily correlated on a large sample of people. This would result in having a very low smallest singular value of $$A$$ and a very high $$\kappa(A)$$ leading to high condition number $\kappa_{A \rightarrow x}$. This means that a small change in $$A$$ can cause a big change in the resulting $$x$$. But the nature of this problem is capturing trends. So a small change in the input data such as somebody having 1 year of graduate education vs. 2 years with the same annual income or 2 point difference in IQ should not substantially affect the importance the solution assigns to different factors.

### Exercise 18.3.

(a) This solution was based on [[2]](#2). Let us simplify the problem by assuming that at every point on the water surface we have ripples angled in all direction. Let $$\alpha$$ be the maximum angle at which the ripples curve. 

The incoming light ray falls on a surface at some angle $\beta$ to the surface normal and reflects along a line that forms the same angle with the normal but on the other side of it in the plane made by the incoming ray and the normal. See figure below:

![figure-1](https://raw.githubusercontent.com/ana0209/ana0209.github.io/master/images/figure-1.png)

This means that a light ray from the house that falls in the middle of the lake on a piece of water with no ripples will reflect so it falls in the observers eyes, since both the light source and the observer are at the same height. However, if the reflecting surface is at some angle, then the distance between the light source and the and the reflecting surface needs to be adjusted as is depicted below:


We see from the figure xx that if the reflecting surface is angled with the sharper angle turned towards the light source that it needs to be closer to the light source than the observer in order to reach the observer at the right height. The situation is reversed when the more oblique angle of the reflecting surface is turned to the light source: then the surface should be closer to the observer than the light source. Also, as visible from the figure above xx, for the same narrow angle $\alpha$, the two situations are symmetrical.

We are going to follow the assumption that the perceived dimension of the object is proportional to the angle the endpoints of the object subtend at the eye (See xxx). According to xx, the perceived dimension of the object is $$n\cdot\gamma$$ where $$\gamma$$ is the mentioned subtended angle. 

Here we will calculate the ratio of width and length of the vertical streak as perceived by the observer and not as the ratio of width and height as they are at the water surface at the points where the light we see is reflected.

What limits the length of the vertical streak is the maximum angle $$\alpha$$ at which the water ripples xxx. We need to calculate the subtended angle $$gamma$$ in figure xx for the maximum angle $$\alpha$$.

From figure xx we can calculate $$\gamma$$ in the following way:


$$\begin{eqnarray}\label{gamma-1}
\alpha_1=\frac{\pi}{2}-\beta_1-\alpha \nonumber \\
\alpha_2=\frac{\pi}{2}+\alpha-\beta_1 \nonumber \\
\alpha_2-\alpha_1=2\alpha 
\end{eqnarray}$$


$$\begin{eqnarray}\label{gamma-2}
\alpha_1+\gamma+\pi-\alpha_2=\pi \nonumber \\
\gamma=\alpha_2-\alpha_1 \nonumber \\
\gamma=2\alpha
\end{eqnarray}$$


From \ref{gamma-1, gamma-2} we see that the angle subtended at the eye that encompasses the length of the vertical streak is equal to $$2\alpha$$.

Width of the streak varies at different points. We will use the point in the middle for our calculations. The actual width on the lake in the middle is the biggest width of the streak*. However, according to [[2]](#2) that is not the point of the maximum perceived length. However, that is the point they use to caculate the width and so will I, because it renders itself to easy calculation of the subtended angle. 


In the middle of the lake, even at a point away from the plane perpendicular to the lake that contains the observer and the light, the light ray has an equidistant path from the source to the point and from the point to the observer. The reflecting surface is inclined in order to account for the needed change in the direction of the travel. The plane is rotated over the axis passing through points $$T_1$$ and $$T_2$$ in figure xx. In the figure yy we show the inclination of the reflecting surface which is symmetric on the two sides and an observer (or light source) projected onto a plane passing through the middle of the lake and being perpendicular on the axis of rotation $$T_1T_2$$. The two sides of the depicted triangle $$V_1O'$$ and $$V_2O'$$ are positioned on the normal to the reflecting plane at the point of the light reflection. From the image we see that the angle $$\angle V_1O'P=\alpha$$. Therefore the physical width of the streak on the lake is $$2 h\tan \alpha$$. 

The angle subtended at the eye for the width of the lake $$\angle P_1OP_2=\delta$$. We know that the length $$\overline{MO}=\sqrt{h^2+l^2}m$$ with $$h=50m$$ and $$2l=1000m$$. We can write $$\tan \delta=h \tan \alpha/\sqrt{h^2+l^2}$$. We know that for small angles $$\beta$$ that $$\beta \approx tan \beta$$. In our case with $$\frac{h}{\sqrt(h^2+l^2)}\approx 0.01$$ for at least any $$\alpha \leq 45^\circ$$ $$\tan (\delta/2)$$ is small. [[2]](#2) makes the assumption that in normal circumstances of not excessive wind that $$\alpha=15^\circ$$. Taking that into account we can write: 

$$\begin{eqnarray}\delta \approx \frac{2h\tan\alpha}{\sqrt{h^2+l^2}}
\end{eqnarray}$$

And finally, the ratio of the perceived width and length is $$r=\frac{\delta}{\gamma}=h\tan\alpha/(\alpha\sqrt{h^2+l^2})=0.1018$$. For small $$\alpha$$ we have $$\tan\alpha \approx \alpha$$ so we can write $$r\approx\frac{h}{\sqrt(h^2+l^2)}=\sin \omega=0.0995$$. We see that for $$\alpha=15^\circ$$ the two calculations are quite close.

(b) This problem is related to the way the $$\delta y$$ behaves for various tilts of $$range(A)$$. The biggest error and the biggest tilt for a fixed $$\|\delta A\|_2$$ is achieved when the range is tilted around axis of rotation that is perpendicular to the segment connecting the origin to point $$y$$. The smallest error is achieved when the tilt is around axis of rotation that connects the origin and the point $$y$$. Similarly, in this problem we get different dimensions for width and the length of the vertical streak with the maximum length in the plane perpendicular to the lake connecting the light source and the observer when the reflecting plane is rotated around an axis that is perpendicular to the line passing through points $$T_1$$ and $$T_2$$ in the figure xx. As we tilt the reflecting plane around different axis, the length of the segment where light is visible shortens. We can see that particularly when we observe the width of the light streak in the middle of the lake as we did in part (a) of the problem. The approximate ratio of the width and the length has a similar looking value as is the case for changes in $$\delta y$$: $$\sin\omega$$ with $$\omega$$ in this case being the angle at the middle of the lake between the line connecting that point to the observer and the horizontal line on the lake passing through that point.



### Exercise 18.4.

The condition number of $y$ becomes $0$ with respect to perturbations of $A_{mxn}$ when $m=n$ because an assumption was made that $A$ has full rank. Since $m=n$ that means that $A$ is non-singular. We know that for small enough perturbations $E$ of a non-singular $A$, $B=A+E$ is non-singular (for the proof see Theorem 3 in this [write-up]({{ site.baseurl }}/pdfs/Linear_Least_Squares_Problem_Conditioning.pdf)). Condition number is defined in the limit for small perturbations, so we may assume that the perturbed $B$ is non-singular and full rank, much like $A$. Since both $A$ and be $B$ are square matrices of full rank and same dimensions, their ranges are the same. We know that $y$ is a projection into the range of $A$. With perturbations it becomes a projection of $b$ into the range of $B$. Since the two ranges are the same, the projection remains the same across different perturbations. Therefore, condition number for $y$ with respect to perturbations in $A$ becomes $0$.

### References
<a id="1">[1]</a> 
Trefethen, L., and Bau, D. Numerical Linear Algebra, SIAM, (1997).

<a id="2">[2]</a> 
Minnaert, M. The Nature of Light and Color in the Open Air, Dover Publications Inc., (1954)
