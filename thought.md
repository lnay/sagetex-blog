# Using SageTex in larger articles in a sane way

I've recently been using
[SageTex](https://doc.sagemath.org/html/en/tutorial/sagetex.html) in my
research writing.
There's been ups and downs, benefits but also multiple drawbacks to my
workflows until I finally settled on how to do things in a *sane* way to
overcome the limitations in the methods demonstrated in just the
[official tutorials](https://doc.sagemath.org/html/en/tutorial/sagetex.html).
I've decided that instead of venting over my experiences, maybe it's better to
channel some more positive energy into these **top tips** for using sagetex in
larger projects.
I'll be starting with short, easily-actionable tips that aren't present in the
official tutorials, and then finishing with a couple larger pieces advice around
overall project structure.


To set the expectations, let me clarify my personal reasons for including
SageMath code into my document building:

### 1) Programmatically generate LaTeX expressions to avoid errors
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

Instead of generating plots programmatically inside the LaTeX document, most
people use separate tools to create diagrams. Idealogically, this impacts
reproducibility of the research. But via SageTex, it's also easier to use the same
expressions to generate LaTeX formulae, as creating a plot.

# TIP 1: SageMath plot sizing

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
The rest of the tips here however, you will not find in the official material.

# TIP 2: Use dmath environment for long expressions
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


# TIP 3: Use latex hackery to overcome Sage limitations

## Problem

A big limitation of using the `\sage` function to generate formulae in a latex
document, is that symbols in SageMath, unlike in
[SymPy](https://www.sympy.org/en/index.html), must be valid Python
identifiers (i.e. can be a variable name).
The LaTeX representations are then derived from the variable name.
Typically, the latex representation is precisely the identifier, such as:
`x` is displayed $x$ (with latex `x`), and `a_t` would be displayed $a_t$
(with latex `a_t`).
Although the backslash `\` is disallowed in Python identifiers, certain latex
command names are recognised and treated appropriately, for example:
`Phi_t` is displayed $\Phi_t$ (with latex `\Phi_t`).
```sage
sage: # var() creates SageMath variables so need to give valid var names
sage: var("x_2 Phi")
(x_2, Phi)
sage: latex(x_2) # latex generally matches variable name
x_{2}
sage: latex(Phi) # exceptions are made for common latex commands
\Phi
```
In particular, the caret symbol `^` is disallowed in Python identifiers,
making it impossible to create symbols which display with superscripts.
But also, the `-` symbol is also disallowed, making it impossible, for
example, to create a SageMath symbol for $\beta_{-}$.
```sage
sage: var("a^t")
ValueError: The name "a^t" is not a valid Python identifier.
sage: var("beta_{-}")
ValueError: The name "beta_{-}" is not a valid Python identifier.
```

On the face of it, this limits the expressiveness of the expressions that can
be generated with SageMath into LaTeX documents. But there are ways around
this.

## Solution

My way of getting round this, at the cost of potential chaotic hackery, is to
make use of commands in (La)Tex such as `\let`, `\def`, and `\renewcommand` to
move the responsibility of displaying the symbols the way you want, onto the
main LaTeX document, instead of SageMath.

Suppose you want to create an expression from SageMath involving $\beta^{n}\_{-}$.
In the SageMath code, you can only define a symbol for `beta` (displayed as
$\beta$).
However, with the following latex code, you can redefine the `\beta` command,
generated by SageMath to instead produce $\beta^{n}\_{-}$:

```latex
\let\originalbeta\beta
\renewcommand\beta{{\originalbeta^{n}_{-}}}

\begin{sagesilent}
var("beta")
\end{sagesilent}

\begin{equation*}
\sage{ (beta + 1)^2 }
\end{equation*}
```
This will render to the following:
$$(\beta^n_- + 1)^2$$

## Limiting the bad stuff

Redefining default values for things is inherently ugly, so here are a couple
of ways of mitigating the harm.

We can tweak the above example to limit the scope of the redefinition for the
`\beta` command with `\begingroup` and `\endgroup`, removing possible
unintended consequences in other parts of the document:

```latex
\begingroup % beginning of scope redefining \beta
\let\originalbeta\beta
\renewcommand\beta{{\originalbeta^{n}_{-}}}
...
\begin{equation*}
\sage{ (beta + 1)^2 }
\end{equation*}
\endgroup % end of scope redefining \beta
```

In this example, $\beta$ is redefined to be a relatively similar $\beta^n_-$.
The variable name in the Sage code is still `beta`, but happens to still fit
quite well.
However, you may want to create a symbol which displays to something very
different to any standard symbol in SageMath, say $\mathrm{ch}_2^\beta(v)$, in
which case the variable name will not match very well.
In such an example, it would make sense to redefine a symbol you are unlikely
to use, and store it to an appropriate variable name in the SageMath code:

```latex
\begingroup % beginning of scope redefining \kappa (second_twisted_chern)
\let\originalkappa\kappa
\renewcommand\kappa{{\mathrm{ch}_2^\beta(v)}}

\begin{sagesilent}
# Create symbol for kappa (to display in the latex differently)
# But store it with a variable name better suited the intended mathematical
# meaning
second_twisted_chern = var("kappa")
\end{sagesilent}

\begin{equation*}
\sage{ (second_twisted_chern + 1)^2 }
\end{equation*}
\endgroup % end of scope redefining \kappa (second_twisted_chern)
```
This would render to:
$$\left(\mathrm{ch}\_2^\beta(v) + 1\right)^2$$

# TIP 4: Don't put everything into sagesilent

## Problem

The path of least resistance when writing Sage Code to be used by SageTex in a
latex document, is to write it into `sagesilent` or `sageblock` environments.
```latex
\begin{sagesilent}
# long calculations:
...

final_expression = ...
illustrative_plot = ...
\end{sagesilent}

\begin{equation}
\sage{final_expression}
\begin{equation}

\sageplot{illustrative_plot}
```

But when iterating on the code to do the correct thing (especially so for
creating plots), the feedback loop is very slow (run latex, run sage on
generated script, ... then run latex again...).
Furthermore, any syntax error will make the second step (running the generated
SageMath script) fail completely, making you repeat the first two steps again.
As the document gets larger, it gets even slower and irritating.
SageMath provides a brilliant Jupyter Notebook experience, especially with the
cells which you can run individually to make small adjustments on the fly when
creating the code.
It's natural to do the initial development in notebook and copy-paste into the
SageTex environments, however you may decide to develop on the code some more
but will now find yourself in a more difficult environent.
My thesis here is that you should never copy-paste the code into SageTex
environments and hence never leave the Jupyter notebook.

There are also a few extra knacks which I have with developing in the SageTex
environments, but these are more likely to be noticed by people with more
general software experience:
- Syntax highlighting is poor/non-existant in some editors
- No language intelligence i.e. no find-references, rename-symbol (this is
  available for SageMath notebooks opened in VSCode)
- No debugger (again available for notebooks in VSCode)

## Solution

As mentioned above, I recommend writing most of the SageMath code in Jupyter
notebooks, and create well-labeled variables for the expressions and plots to
be used in the LaTeX document.

An object, say `final_expression`, can be imported from a Python script
(not a SageMath script), say `notebook.py` as follows:

```latex
\begin{sagesilent}
from notebook import final_expression
\end{sagesilent}

\begin{equation}
\sage{final_expression}
\end{equation}
```

Suppose, we have a SageMath notebook `notebook.ipynb` defining
`final_expression` instead, we need to create a corresponding Python script
`notebook.py`.

The following steps in a unix-style shell would do this (including SageMath
shell on Windows):
- From the notebook, create a SageMath script with
  `jupyter nbconvert --to script characteristic_curves.ipynb`
  (with erroneous `.py` extension)
- Correct `.py` extension to `.sage` with `mv notebook.py notebook.sage`
- Remove the ipython call generated by `%display latex` which would crash
  normal Python/Sage with `sed -e "/get_ipython/d" -i notebook.sage`
- Convert SageMath script to valid Python with `sage --preparse notebook.sage`
  (creating file named `notebook.sage.py`)
- Rename to required filename with `mv notebook.sage.py notebook.py`

There's multiple steps here but this can be scripted.
Furthermore the `make` utility is available in most environments where
SageMath is installed, so we can conveniently add rules to a Makefile which
recreates the required Python script whenever the notebook changes.
Have a look at the example repository that comes with this blog post (TODO)
to see this in action.

## Following tip 3 in conjunction with tip 4

Recall the last example in the "limiting bad stuff" subsection in tip 3.
This involved redefining LaTeX commands to make arbitrary SageMath symbols
display themselves the way you want in the document.

My tip for developing the expressions which will appear in the main document,
is that `\let`, `\def`, and `\newcommand` functions used in markdown cells
will successfully affect the rendering of SageMath expressions.

For example, create one Markdown cell with the following content:
```markdown
%display latex
$\let\originalkappa\kappa
\renewcommand\kappa{{\mathrm{ch}_2^\beta(v)}}$
Redefining $\originalkappa$ to $\kappa$
```

Then, the output from the following code cell will be affected:
```sage
second_twisted_chern = var("kappa")
final_expression = (second_twisted_chern + 1)^2
final_expression # non-assignment line to render latex in notebook
```
This will create an output cell with the following:
$$\left(\mathrm{ch}\_2^\beta(v) + 1\right)^2$$

Then, once the a corresponding Python script `notebook.py` is created, this
same expression can be used in the main LaTeX document:
```latex
\begin{sagesilent}
from notebook import final_expression
\end{sagesilent}

\bgroup % same as \begingroup
\let\originalkappa\kappa
\renewcommand\kappa{{\mathrm{ch}_2^\beta(v)}}
\begin{equation}
\sage{final_expression}
\end{equation}
\egroup % same as \endgroup
```
p.s. don't forget to include `%display latex` to the first cell of the
notebook.

# TIP 5: Use this docker container for Git(Hub/Lab) CI

Using SageTex in your LaTeX document automatically rules you out of using
[OverLeaf](https://www.overleaf.com/).
The next most common method for collaborating on a document
(or even just managing a personal project),
is hosting the project on GitHub or GitLab.
These platforms do not support building LaTeX documents out of the box,
however they can set up to do so, and in particular working with SageTex too.
The key is to provide a docker container which has both SageTex and a LaTeX
distribution installed. Luckily, such a container
[exists](registry.gitlab.com/lukenaylor/latex/sagetex-image:latest).
Then we can just create a functioning Makefile to build the pdf document, and
run `make` from within the container.

The example repository that comes with this blog (TODO) is correctly set up
for this, and is a good starting point to try.
Otherwise, the instructions are slightly different depending on the git
repository hosting service:

## Gitlab

In the root of the repository, add a file called `.gitlab-ci.yml` file
with the following contents:
```yaml
build:
  image: registry.gitlab.com/lukenaylor/latex/sagetex-image:latest
  script:
    - source /root/.bashrc
    - make || echo make failed
  artifacts:
    paths:
      - "*.pdf"
```

## Github (untested)

If not present, add a `.github/workflows/` directory to the repo, and add a
file `build.yml` (name not important) with the following contents:
```yaml
name: Build LaTeX (with SageTex) document to pdf

on:
  push:
    branches: ["main"] # or "master" instead

jobs:
  build:
    container:
      image: registry.gitlab.com/lukenaylor/latex/sagetex-image:latest
    steps:
    - name: Build PDF
      run: make
    - name: Upload PDF artifact
      uses: actions/upload-artifact@v3
      with:
        name: main.pdf
        path: main.pdf # adjust to generated pdf name
```

