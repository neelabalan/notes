## MathJax MD Referenceo

- [MathJax MD Referenceo](#mathjax-md-referenceo)
  - [Matrices](#matrices)
  - [Fractions](#fractions)
  - [Piecewise](#piecewise)
  - [Arrays](#arrays)
  - [Centering equations on the sign](#centering-equations-on-the-sign)
  - [Sizing](#sizing)
  - [Vectors](#vectors)
  - [Arrows](#arrows)
  - [Limits, sums, products \& integrals](#limits-sums-products--integrals)
- [Reference links](#reference-links)


### Matrices

 
 
$$
   \begin{pmatrix}
     1 & 2 \\
     3 & 4 \\
   \end{pmatrix}
$$


$$
   \begin{bmatrix}
     1 & 2 \\
     3 & 4 \\
   \end{bmatrix}
$$


$$
   \begin{Bmatrix}
     1 & 2 \\
     3 & 4 \\
   \end{Bmatrix}
$$


$$
   \begin{vmatrix}
     1 & 2 \\
     3 & 4 \\
   \end{vmatrix}
$$

$$
   \begin{pmatrix}
     1      & a_1    & \cdots & a_n    \\
     1      & b_1    & \cdots & b_n    \\
     \vdots & \vdots & \ddots & \vdots \\
     1      & z_1    & \cdots & z_n    \\
   \end{pmatrix}
$$


$$ 
\left[
    \begin{array}{cc|c}
    1 & 2 & 3 \\
    4 & 5 & 6 \\
    \end{array}
\right] 
$$

### Fractions

$$ y = \frac{x+1}{x+2} $$


$$
x = \cfrac{1}{\sqrt{2}+
      \cfrac{1}{\sqrt{2}+
        \cfrac{1}{\sqrt{2}+\dotsb}
      }
    }
$$

### Piecewise

$$  f(n) =
\begin{cases}
  \dfrac{n+1}{2} & \text{if n is even} \\ \\
  \dfrac{n}{2}   & \text{if n is odd}
\end{cases}
\qquad \forall n \in \mathbb N
$$


### Arrays

$$
\begin{array}{c|lcr}
  n & \text{Left} & \text{Center} & \text{Right} \\
  \hline
  1   & 1      & 1     & 1     \\
  2   & 10     & 100   & 10    \\
  3   & 100    & 1000  & 100
\end{array}
$$

### Centering equations on the sign

$$
\begin{align}
  f(x)   &= \frac{x+1}{x-2} \\
  f'(x)  &= \frac{(x-2) - (x+1)}{(x-2)^2} \\
         &= - \frac{3}{x^2 - 4x + 4}
\end{align}
$$

$$
\left \{
  \begin{align}
     2x	+	y	−	2z  &=	3   \\
     x	−	y	−	z    &=	0   \\
     x	+	y	+	3z   &=	12
  \end{align}
\right.
$$

### Sizing

$$ ( \sqrt{ \frac{x}{y} } ) $$

$$ \left( \sqrt{ \frac{x}{y} } \right) $$

$$ \Biggl( \biggl( \Bigl( \bigl( ( x^2 ) \bigr) \Bigr) \biggr) \Biggr) $$


### Vectors

$$ \vec{u} + \vec{v} = \vec{w} $$

$$ \vec{AB} + \vec{BC} = \vec{AC} $$

$$ \overrightarrow{AB} + \overrightarrow{BC} = \overrightarrow{AC} $$

### Arrows

$$ T:\mathbb R \to \mathbb R,\; x \mapsto x+1 $$

$$ 2x + 3 = 0 \implies x=- {3 \over 2} $$

$$ \iff  \qquad  \impliedby $$

$$ X \overset{a}{\underset{b}{\to}} Y $$

### Limits, sums, products & integrals

$$ \lim \limits_{ x \to 2^+ } { \dfrac{x+1}{x-2} } $$

$$ \sum_{n=1}^\infty \frac{1}{n^2} $$

$$ \prod_{n=1}^\infty \frac{1}{n^2} $$

$$ \int_o^\infty e^x dx $$

$$ \iint_D  \qquad  \iiint_V  \qquad  \oint_C $$

<!-- $$ {\huge\unicode{x222F}}_S   \qquad   {\huge\unicode{x2230}}_V $$ -->

## Reference links

- [sqlpac.com](https://www.sqlpac.com/en/documents/mathjax-tex-practical-handbook-quick-reference.html)
- [mathematics meta](https://math.meta.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference)