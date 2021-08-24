---
title: "Python style guide"
date: 2021-05-09T20:00:00-04:00
draft: true
---
Style guide is for the readers of your code ~

Some quick notes for python development. I haven't use wemake-python to be able to evaluate.
- Google stack:
    - [pytype](https://github.com/google/pytype): a tool used by google to check and infer types for python code. After using it at google, I would say using type for python and have a quick feedback cycle when you edit code and get type check error add more value than the cost. Type is also a good documentation when reading other developers' code.
        pytype vs mypy: the biggest difference is that pytype does whole-file type inference for unannotated code, so you can just run pytype over your project without having to do anything. (as a nice consequence of this we can also generate type stubs for you, and optionally even merge them into your source code as annotations).
    - [Google style guide](https://google.github.io/styleguide/pyguide.html): for things that are not detected by linter, like docstring, naming. It goes with [pylint](https://www.pylint.org/) as linter and [yapf](https://github.com/google/yapf/) as code formatter
- wemake-python: not compatible with `black`
    - [wemake-python-styleguide](https://github.com/wemake-services/wemake-python-styleguide): linting tool. 
    - It suggests using code formaters in <https://wemake-python-stylegui.de/en/latest/pages/usage/integrations/auto-formatters.html>, for eg [autopep8](https://github.com/hhatto/autopep8), isort
    - [flakehell](https://wemake-python-stylegui.de/en/latest/pages/usage/integrations/flakehell.html) to add style guide to a legacy project. Only new code get lint errors.
- [black](https://github.com/psf/black): it's opinioned, so to use it with linters, we need to configure the linters
    - <https://black.readthedocs.io/en/stable/compatible_configs.html>
- will take a look at https://pyre-check.org/docs/types-in-python

Git hooks: 
    - [Git hook worflow](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)
    - [pre-commit-hooks](https://github.com/pre-commit/pre-commit-hooks)
        - useful hooks: check-added-large-files, check-docstring-first, check-executables-have-shebangs, check-json, check-merge-conflict, check-shebang-scripts-are-executable, check-yaml, debug-statements, detect-aws-credentials, double-quote-string-fixer, detect-private-key, end-of-file-fixer, file-contents-sorter, mixed-line-ending, no-commit-to-branch (disable commit con master directly), pretty-format-json, requirements-txt-fixer, trailing-whitespace
    - bigger list of hooks: https://pre-commit.com/hooks.html
    - docstring: https://github.com/FalconSocial/pre-commit-mirrors-pep257 https://github.com/myint/docformatter
    - [gitlint](https://github.com/jorisroovers/gitlint)
    - [talisman](https://github.com/thoughtworks/talisman): A tool to detect and prevent secrets from getting checked in
    - [bandit](https://github.com/PyCQA/bandit): find common security issues in Python code
    - [markdownlint](https://github.com/igorshubovych/markdownlint-cli)
    - https://github.com/MarcoGorelli/absolufy-imports
    - https://github.com/asottile/pyupgrade
    - [pre-commit](https://github.com/pre-commit/pre-commit) python package. Run on changed files during commit.
    - pre-receive
    - linting should be done in CI.

pytest
    - [directory structure](https://docs.pytest.org/en/reorganize-docs/new-docs/user/directory_structure.html): prefer unit tests to be in the same folder as code, each code file has corresponding test file with _test suffix, for eg example.py and example_test.py, so test file will be next to code file. Integration tests can be in dedicated tests folder.
    - [customize](https://docs.pytest.org/en/reorganize-docs/customize.html) `python_files` to allow suffix only