# Using SageTex in larger articles in a sane way

I've recently been using
[SageTex](https://doc.sagemath.org/html/en/tutorial/sagetex.html) in my
research writing.
There's been ups and downs, benefits but also multiple drawbacks to my
workflows until I finally settled on how to do things in a *sane* way to
overcome the limitations in the methods demonstrated in the
[official tutorials](https://doc.sagemath.org/html/en/tutorial/sagetex.html).
I've decided that instead of venting over my experiences, maybe it's better to
channel some more positive energy into these **top tips** for using sagetex in
larger projects.
I'll be starting with short, easily-actionable tips that aren't present in the
official tutorials, and then finishing with a couple larger pieces advice around
overall project structure.


To set the expectations, let me clarify my personal reasons for including
SageMath code into my document building:

### 1) Programmatically generate latex expressions to avoid errors
(add tiny example)

At the point when I decided to start using SageTex, I was frequently making
mistakes in the expressions I was deriving. Particularly when specializing
expressions to different contexts, where there are different coefficients
involved which were easily mixed up. Sometimes the expressions were just long.
Having a CAS do these calculations added an extra degree of certainty to my
work. Furthermore, including the code inside the project (as opposed to doing
it separately) just keeps the calculation steps *on record* easily checked in
case I worry I had done something incorrectly.

### 2) Include plot creation into the document build
(add tiny example)

Instead of generating plots programmatically inside the latex document, most
people use separate tools to create diagrams. Idealogically, this impacts
reproducibility of the research. But via SageTex, it's also easier to use the same
expressions to generate latex formulae, as creating a plot.

# SageMath plot sizing

Although, not present in the official tutorials, the
[official example](https://github.com/sagemath/sagetex/blob/master/example.tex)
shows some invaluable options for the `\sageplot` function to adjust sizing
among other things.
The most common usages I have are:
```latex
\sageplot[width=\textwidth]{...}
```
```latex
\sageplot[width=\linewidth]{...}
```
If you have trouble sizing plots correctly, especially in subfigures, this
might be your solution.


Another takeaway here is to have a look at the
[official example](https://github.com/sagemath/sagetex/blob/master/example.tex)
to come across more advanced usage of SageTex than the minor snippets in the
tutorial page.
The rest of the tips here however, are completely different to what you may
find in the official material.

# Use dmath environment for long expressions
When generating long latex expressions into equation environments, you may run
into the equations extending into the margins, or even off the page.

```latex
% demonstrate coefficients of particularly long binomial expansion

\begin{sagesilent}
    var("x y") # declare symbols for 'x' and 'y'
\end{sagesilent}

\begin{equation}
    \sage{expand((x+y)^10)}
\end{equation}
```
(mini example)

The easy fix for this is to use the `dmath` environment from the `breqn`
package. This environent behaves very similarly to the `equation` environment,
however it breaks the equation over multiple lines appropriately.

```latex
% in preamble:
\usepackage{breqn}


% ... in document body
\begin{sagesilent}
var("x y") # declare symbols for 'x' and 'y'
\end{sagesilent}

% demonstrate coefficients of particularly long binomial expansion
\begin{dmath}
    \sage{expand((x+y)^10)}
\end{dmath}
```

(show render)


# Use latex hackery to overcome sage limitations

A big limitation of using the `\sage` function to generate formulae in a latex
document, is that symbols in SageMath, unlike in
[SymPy](https://www.sympy.org/en/index.html), must be valid Python
identifiers.
The latex representations are then a product of the Python identifiers.
Typically, the latex representation is precisely the identifier, such as:
`x` is displayed $$x$$ (with latex `x`), and `a_t` would be displayed $$a_t$$
(with latex `a_t`).
Although the backslash `\` is disallowed in Python identifiers, certain latex
command names are recognised and treated appropriately, for example:
`Phi_t` is displayed $$\Phi_t$$ (with latex `\Phi_t`).

- store in more semantic name
- use \def \let and \renewcommand to make symbols appear as intended
- \bgroup \egroup to scope this hackery

# Don't put everything into sagesilent

problems:
- slow feedback loop
- one error breaks the rest
- no language intelligence + highlighting

solution:
- external lib allows import
- jupyter notebook allows quicker feedback loop
- Makefile can be used to develop in sagemath notebook, but reference to python
  - don't explicity print (to keep import cleaner)

# Use my docker container for git(hub/lab) CI

