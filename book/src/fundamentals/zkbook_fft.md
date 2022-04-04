### Multiplying polynomials with the Fast Fourier Transform (FFT)

The algorithm that allows us to multiply polynomials in $O(n \log n)$ is called the **Cooley-Tukey fast Fourier transform**, or **FFT** for short.
It was discovered by [Gauss](https://en.wikipedia.org/wiki/Carl_Friedrich_Gauss) 160 years earlier, but then separately rediscovered and publicized by Cooley-Tukey.

The heart of Cooley-Tukey FFT is actually about converting between coefficient and evaluation representations, rather than the multiplication itself.
Given polynomials $p$ and $q$ in dense coefficient representation, it works like this.
1.  Convert $p$ and $q$ from coefficient to evaluation form in $O(n\log n)$ using Cooley-Tukey FFT
2.  Compute $r = p*q$ in evaluation form by multiplying their points pairwise in $O(n)$
3.  Convert $r$ back to to coefficient form in $O(n\log n)$ using the inverse Cooley-Tukey FFT

The key observation is that we can choose any $n$ distinct evaluation points to represent any degree $n - 1$ polynomial.
The Cooley-Tukey FFT works by selecting points that yield an efficient FFT algorithm.  These points are fixed and work for any polynomials of a given degree.

Specifically, say we have $\omega \in F$ such that $\omega^{n} = 1$, and $\omega^r \neq 1$ for any $0 < r < n$.

Put another way, all the values $1, \omega, \omega^2, \omega^3, \dots, \omega^{n-1}$ are distinct and $\omega^n = 1$.

Put yet another way, the group generated by $\omega$ inside $F^\times$ (written $\langle \omega \rangle$) has size $n$.

We call such an $\omega$ a primitive $n$-th root of unity.

Suppose we have an $\omega$ which is a primitive $2^k$th root of unity and let $A_k = \{ 1, \omega, \dots, \omega^{2^k - 1} \}$.

The FFT algorithm will let us compute $\mathsf{interp}_{A_k}$ for this set.

Actually, it is easier to see how it will let us compute the $\mathsf{eval}_{A_k}$ algorithm efficiently.

We will
describe an algorithm $\mathsf{FFT}(k, \omega, f)$ that takes as input

- $k \in \N$
- $\omega \in F$ a primitive $2^k$th root of unity
- $f \in F[x]_{< 2^k}$ in dense coefficients form (i.e., as a vector of coefficients of length $n$).

and outputs the vector of evaluations
$$
[f(1), f(\omega), f(\omega^2) \dots, f(\omega^{2^k - 1})]
$$
and does it in time $O(k 2^k)$ (which is to say, $n \log n$ if $n = 2^k$).

Notice that naively, computing each evaluation $f(\omega^i)$ using the coefficients of $f$ would require time $O(n)$, and so computing all $n$ of them would require time $O(n^2)$.

The algorithm $\mathsf{FFT}(k, \omega, f)$ can be defined recursively as follows.

If $k = 0$, then $\omega$ is a primitive $1$st root of unity, and $f$ is a polynomial of degree $0$.
That means $\omega = 1$ and also $f$ is a constant $c \in F$.
So, we can immediately output the array of evaluations $[c] = [f(1)]$.

If $k > 0$, then we will split $f$ into two polynomials, recursively call $\mathsf{FFT}$ on them, and reconstruct
the result from the recursive calls.

To that end, define $f_0$ to be the polynomial whose coefficients are all the even-index coefficients of $f$
and $f_1$ the polynomial whose coefficients are all the odd-index coefficients of $f$.
In terms of the array representation, this just means splitting out every other entry into two arrays.
So that can be done in time $O(n)$.

Write $f = \sum_{i < 2^k} c_i x^i$, so that $f_0 = \sum_{i < 2^{k - 1}} c_{2i} x^i$ and
$f_1 = \sum_{i < 2^{k-1}} c_{2i + 1} x^i$. Then

$$
\begin{aligned}
f(x)
&= \sum_{i < 2^k} c_i x^i \\
&= \sum_{i < 2^{k-1}} c_{2i} x^{2i} + \sum_{i < 2^{k-1}} c_{2i + 1} x^{2i + 1} \\
&= \sum_{i < 2^{k-1}} c_{2i} (x^2)^i+ \sum_{i < 2^{k-1}} c_{2i + 1} x \cdot (x^2)^i  \\
&= \sum_{i < 2^{k-1}} c_{2i} (x^2)^i+ x \sum_{i < 2^{k-1}} c_{2i + 1} (x^2)^i  \\
&= f_0(x^2) + x f_1(x^2)
\end{aligned}
$$

Now, notice that if $\omega$ is a $2^k$th root of unity, then $\omega^2$ is a $2^{k - 1}$th
root of unity. Thus we can recurse with $\mathsf{FFT}(k - 1, \omega^2, f_0)$ and similarly
for $f_1$. Let

$$
\begin{aligned}
[e_{0, 0}, \dots, e_{0, 2^{k-1} - 1}] &= \mathsf{FFT}(k-1, \omega^2, f_0) \\
[e_{1, 0}, \dots, e_{1, 2^{k-1} - 1}] &= \mathsf{FFT}(k-1, \omega^2, f_1)
\end{aligned}
$$

By assumption $e_{i, j} = f_i((\omega^2)^j)$. So, for any $j$ we have

$$
\begin{aligned}
f(\omega^j)
&= f_0((\omega^2)^j) + \omega^j f_1((\omega^2)^j)
\end{aligned}
$$

Now, since $j$ may be larger than $2^{k-1} - 1$, we need to reduce it mod $2^{k-1}$, relying on the fact that
if $\tau$ is an $n$th root of unity then $\tau^j = \tau^{j \mod n}$ since $\tau^n = 1$. Thus,
$(\omega^2)^j = (\omega^2)^{j \mod 2^{k-1}}$ and so we have

$$
\begin{aligned}
f(\omega^j)
&= f_0((\omega^2)^{j \mod 2^{k-1}} ) + \omega^j f_1((\omega^2)^{j \mod 2^{k-1}}) \\
&= e_{0, j \mod 2^{k-1}} + \omega^j e_{1, j \mod 2^{k-1}}
\end{aligned}
$$

We can compute the array $W = [ 1, \omega, \dots, \omega^{2^k - 1}]$ in time $O(n)$ (since each
entry is the previous entry times $\omega$). Then we can compute each entry of the output in $O(1)$ as

$$
\begin{aligned}
f(\omega^j)
&= e_{0, j \mod 2^{k-1}} + W[j] \cdot e_{1, j \mod 2^{k-1}}
\end{aligned}
$$

There are $n$ such entries, so this takes time $O(n)$.

This concludes the recursive definition of the algorithm $\mathsf{FFT}(k, \omega, f)$.

> **Algorithm: computing $\mathsf{eval}_{A_k}$**
>  * $\mathsf{Input~} f = [c_0, \ldots, c_{2^k - 1}]$ the coefficients of polynomial $f(x) = \sum_{i < 2^k} c_i x^i$
>  * $\mathsf{Compute~} W \gets \left[1, \omega, \omega^2, ..., \omega^{2^k - 1}\right]$
>  * $\mathsf{FFT}(k, \omega, f) \rightarrow \left[f(1), f(\omega), f(\omega^2) \dots, f(\omega^{2^k - 1})\right]$
>    * $\mathtt{if~} k == 0$
>      * $\mathtt{return~} f$
>    * $\mathtt{else}$
>      * $\mathsf{Compute~} f_0 = [c_0, c_2, ..., c_{2^k - 2}]$ the even coefficients of $f,$ corresponding to $f_0(x) = \sum_{i < 2^{k - 1}} c_{2i} x^i$
>      * $\mathsf{Compute~} f_1 = [c_1, c_3, ..., c_{2^k - 1}]$ the odd coefficients of $f,$ corresponding to $f_1(x) = \sum_{i < 2^{k - 1}} c_{2i + 1} x^i$
>      * $e_0 \gets \mathsf{FFT}(k - 1, \omega^2, f_0)$
>      * $e_1 \gets \mathsf{FFT}(k - 1, \omega^2, f_1)$
>      * $\mathtt{for~} j \in [0, 2^k - 1]$
>        * $F_j \gets e_{0, j \mod 2^{k - 1}} + W[j] \cdot e_{1, j \mod 2^{k - 1}}$
>      * $\mathtt{return~} F$

Now let's analyze the time complexity. Let $T(n)$ be the complexity on an instance of size $n$ (that is, for $n = 2^k$).

Looking back at what we have done, we have done

- $O(n)$ for computing $f_0$ and $f_1$
- two recursive calls, each of size $n / 2$
- $O(n)$ for computing the powers of $\omega$
- $O(n)$ for combining the results of the recursive calls

In total, this is $O(n) + 2 T(n / 2)$. Solving this recurrence yields
$T(n) = O(n) \cdot log n = O(n \log n)$. Basically, there are $\log n$ recursions before we hit the base case, and each step
takes time $O(n)$. $\square$

Now, in practice there are ways to describe this algorithm non-recursively that have better concrete performance, but that's
out of scope for this document. Read the code if you are interested.

#### Using the FFT algorithm to compute $\mathsf{interp}_{A_k}$

So far we have a fast way to compute $\mathsf{eval}_{A_k}(f)$ all at once, where
$A_k$ is the set of powers of a $2^k$th root of unity $\omega$. For convenience let $n = 2^k$.

Now we want to go the other way and compute a polynomial given an array of evaluations.
Specifically, $n$ evaluations $\left[f(x_0), f(x_1), ..., f(x_{n - 1})\right]$ uniquely
define a degree $n - 1$ polynomial.  This can be written as a system of $n$ equations

$$
\begin{aligned}
f(x_0) &= c_0 + c_1x_0 + \ldots + c_{n - 1}x_0^{n - 1} \\
f(x_1) &= c_0 + c_1x_1 + \ldots + c_{n - 1}x_1^{n - 1} \\
\vdots\\
f(x_{n - 1}) &= c_0 + c_1x_{n - 1} + \ldots + c_{n - 1}x_{n - 1}^{n - 1}, \\
\end{aligned}
$$
which can be rewritten as a matrix vector product.
$$
\begin{bmatrix}
    f(x_0) \\
    f(x_1) \\
    \vdots \\
    f(x_{n - 1})
\end{bmatrix}
=
\begin{bmatrix}
    1 & x_0 & \cdots & x_0^{n - 1} \\
    1 & x_1 & \cdots & x_1^{n - 1} \\
    \vdots & \vdots & \ddots & \vdots\\
    1 & x_{n - 1} & \cdots & x_{n - 1}^{n - 1} \\
\end{bmatrix}
\times
\begin{bmatrix}
    c_{0} \\
    c_{1} \\
    \vdots \\
    c_{n - 1}
\end{bmatrix}
$$
This $n \times n$ matrix is a Vandermonde matrix and it just so happens that square Vandermonde matrices are invertible, iff the $x_i$ are unique.  Since we purposely selected our $x_i$ to be the powers of $\omega$, a primitive $n$-th root of unity, by definition $x_i = \omega^i$ are unique.

Therefore, to compute the polynomial given the corresponding array of evaluations (i.e. interpolation) we can solve for the polynomial's coefficients using the inverse of the matrix.
$$
\begin{bmatrix}
    c_{0} \\
    c_{1} \\
    \vdots \\
    c_{n - 1}
\end{bmatrix}
=
\begin{bmatrix}
    1 & 1 & \cdots & 1^{n - 1} \\
    1 & \omega & \cdots & \omega^{n - 1} \\
    \vdots & \vdots & \ddots & \vdots\\
    1 & \omega^{n - 1} & \cdots & \omega^{(n - 1)(n - 1)} \\
\end{bmatrix}^{-1}
\times
\begin{bmatrix}
    f(1) \\
    f(\omega) \\
    \vdots \\
    f(\omega^{n - 1})
\end{bmatrix}
$$
All we need now is the inverse of this matrix, which is slightly complicated to compute.  I'm going to skip it for now, but if you have the details please make a pull request.

Substituting in the inverse matrix we obtain the equation for interpolation.
$$
\begin{bmatrix}
    c_{0} \\
    c_{1} \\
    \vdots \\
    c_{n - 1}
\end{bmatrix}
=
\frac{1}{n}
\begin{bmatrix}
    1 & 1 & \cdots & 1^{n - 1} \\
    1 & \omega^{-1} & \cdots & \omega^{-(n - 1)} \\
    \vdots & \vdots & \ddots & \vdots\\
    1 & \omega^{-(n - 1)} & \cdots & \omega^{-(n - 1)(n - 1)} \\
\end{bmatrix}
\times
\begin{bmatrix}
    f(1) \\
    f(\omega) \\
    \vdots \\
    f(\omega^{n - 1})
\end{bmatrix}
$$
Observe that this equation is nearly identical to the original equation for evaluation, except with the following substitution.
$$
\omega^i \Rightarrow \frac{1}{n}\omega^{-1i}
$$
Consequently and perhaps surprisingly, we can reuse the FFT algorithm $\mathsf{eval}_{A_k}$  in order to compute the inverse-- $\mathsf{interp}_{A_k}$.

So, suppose we have an array $[a_0, \dots, a_{n-1}]$ of field elements (which you can think of as a function $A_k \to F$) and we want to compute the coefficients of a polynomial $f$ with $f(\omega^i) = a_i$.

To this end, define a polynomial $g$ by $g = \sum_{j < n} a_j x^j$. That is, the polynomial whose coefficients are the evaluations in our array that we're hoping to interpolate.

Now, let $[e_0, \dots, e_{n-1}] = \mathsf{FFT}(k, \omega^{-1}, g)$.

That is, we're going to feed $g$ into the FFT algorithm defined above with $\omega^{-1}$ as the $2^k$th root of unity. It is not hard to check that if $\omega$ is an n-th root of unity, so is $\omega^{-1}$. Remember: the resulting values are the evaluations of $g$ on the powers of $\omega^{-1}$, so $e_i = g(\omega^{-i}) = \sum_{j < n} a_j \omega^{-ij}$.

Now, let $h = \sum_{i < n} e_i x^i$. That is, re-interpret the values $e_i$ returned by the FFT as the coefficients of a polynomial. I claim that $h$ is almost the polynomial we are looking for. Let's calculate what values $h$ takes on at the powers of $\omega$.

$$
\begin{aligned}
h(\omega^s)
&= \sum_{i < n } e_i \omega^{si} \\
&= \sum_{i < n} \omega^{si} \sum_{j < n} a_j \omega^{-ij} \\
&= \sum_{i < n} \sum_{j < n} a_j \omega^{si-ij} \\
&= \sum_{j < n} a_j \sum_{i < n} \omega^{i(s-j)} \\
\end{aligned}
$$

Now, let's examine the quantity $c_j := \sum_{i < n} \omega^{i(s-j)}$. We claim that if $j = s$, then $c_j = n$, and if $j \neq s$, then $c_j = 0$. The first claim is clear since

$$
c_s = \sum_{i<n} \omega^{i(s-s)} = \sum_{i < n} \omega^0 = \sum_{i<n} 1 = n
$$

For the second claim, we will prove that $\omega^{s-j} c_j = c_j$. This implies that $(1 - \omega^{s-j}) c_j = 0$. So either $1 - \omega^{s-j} = 0$ or $c_j = 0$. The former cannot be the case since it implies $\omega^{s-j} =1$ which in turn implies $s = j$ which is impossible since we are in the case $j \neq s$. Thus we have $c_j = 0$ as desired.

So let's show that $c_j$ is invariant under multiplication by $\omega^{s-j}$. Basically, it will come down to the fact that $\omega^n = \omega^0$.

$$
\begin{aligned}
\omega^{s-j} c_j
&= \omega^{s-j} \sum_{i<n} \omega^{i(s-j)} \\
&= \sum_{i < n} \omega^{i(s-j) + (s-j)} \\
&= \sum_{i < n} (\omega^{i+1})^{s-j} \\
&= (\omega^{0+1})^{s-j} + (\omega^{1+1})^{s-j} + \dots + (\omega^{(n-1)+1})^{s-j} \\
&= (\omega^{1})^{s-j} + (\omega^{2})^{s-j} + \dots + (\omega^{n})^{s-j} \\
&= (\omega^{1})^{s-j} + (\omega^{2})^{s-j} + \dots + (\omega^{0})^{s-j} \\
&= (\omega^{0})^{s-j} + (\omega^{1})^{s-j} + \dots + (\omega^{n-1})^{s-j} \\
&= \sum_{i < n} (\omega^i)^{s-j} \\
&= c_j
\end{aligned}
$$

So now we know that

$$
\begin{aligned}
h(\omega^s)
&= \sum_{j < n} a_j c_j \\
&= a_s \cdot n + \sum_{j \neq s} a_j \cdot 0 \\
&= a_s \cdot n
\end{aligned}
$$

So if we define $f = h / n$, then $f(\omega^s) = a_s$ for every $s$ as desired. Thus we have our interpolation algorithm, sometimes called an inverse FFT or IFFT:

> **Algorithm: computing $\mathsf{interp}_{A_k}$**
>
> 0. Input: $[a_0, \dots, a_{n-1}]$ the points we want to interpolate and $\omega$ a $n$th root of unity.
>
> 1. Interpret the input array as the coefficients of a polynomial $g = \sum_{i < n} a_i x^n$.
>
> 2. Let $[e_0, \dots, e_n] = \mathsf{FFT}(k, \omega^{-1}, g)$.
>
> 3. Output the polynomial $\sum_{i < n}(e_i / n) x^i$. I.e., in terms of the dense-coefficients form, output the vector $[e_0 / n, \dots, e_{n - 1}/n]$.

Note that this algorithm also takes time $O(n \log n)$

## Takeaways

- Polynomials can be represented as a list of coefficients or a list of evaluations on a set $A$

- If the set $A$ is the set of powers of a root of unity, there are time $O(n \log n)$ algorithms for converting back and forth between those two representations

- In evaluations form, polynomials can be added and multiplied in time $O(n)$

  - TODO: caveat about hitting degree

### Exercises

- Implement types `DensePolynomial<F: FfftField>` and `Evaluations<F: FftField>` that wrap a `Vec<F>` and implement the FFT algorithms described above for converting between them

- Familiarize yourself with the types and functions provided by `ark_poly`