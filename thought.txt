Provide a minimal example/template

# Don't put everyting into sagesilent

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
