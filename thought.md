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

# Use latex hackery to overcome sage limitations

- symbols in sagemath must be valid python identifiers
- store in more semantic name
- use \def \let and \renewcommand to make symbols appear as intended
- \bgroup \egroup to scope this hackery

# Use my docker container for git(hub/lab) CI

# Sageplot hidden option
- use \sageplot[width=\textwidth] for example

# Use dmath environment for long expressions
- adds newlines in appropriate places
