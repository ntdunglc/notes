---
title: "Python CLI Application"
date: 2020-05-19T23:11:02-04:00
draft: false
---

This note summarize recent experience building a few command line (CLI) applications in Python.  

Here are a few assumptions before I describe the approach
- It's not a CLI that process GBs of data, or need to have lowest overhead. 
If that's the case, Rust/Go might be better candidate than Python.
- It's not an interactive CLI like `mysql`. However it can still support simple interactive workflow.
- It's not a CLI that [do one thing and do it well](https://en.wikipedia.org/wiki/Unix_philosophy#Do_One_Thing_and_Do_It_Well). 
It's more like a collection of multiple CLIs.

The core library I used is [Click](https://click.palletsprojects.com/en/7.x/), this is a solid and safe choice. 
Python has other options like [argparse](https://docs.python.org/3/library/argparse.html) in std lib, and [python-fire](https://github.com/google/python-fire). However I found that `argparse` give you too little structure, and `fire` is conveninent for development, 
but when it goes production, you might want to tighten it and have more input validation.  

Using Click, the most important question is how to organize your application, and there's multiple ways to do it, but this is the setup I found not too difficult to start, and still flexible for a very big application.
- Use [click.MultiCommand](https://click.palletsprojects.com/en/7.x/commands/#custom-multi-commands). 
It might be a bit easier to start with [click.Group](https://click.palletsprojects.com/en/7.x/commands/#nested-handling-and-contexts),
but using MultiCommand encourage better folder structure when each command live in its own file, and your team should really have some naming convention for the files and the commands.
- Use [Context](https://click.palletsprojects.com/en/7.x/complex/#contexts) to store application configuration and common arguments that can be shared between commands, for example `--verbose`, `--env`, `--output`. These parameters are define in the root CLI, and are passed down to subcommands.
- Support different output format, including human-readable and machine-readable (JSON/CSV) format. [tabulate](https://pypi.org/project/tabulate/) is a solid choice for human-readable output since it support column alignment and number format out of the box.
- Support stdin input, this depends on what your command does, but if you take JSON/CSV as input, especially for batch commands, it will enable integration with awesome tools like [jq](https://stedolan.github.io/jq/), [xsv](https://github.com/BurntSushi/xsv) and old school unix commands like `awk`, `sed`, `grep`
- Enable logging with option of level in root CLI, either with std [logging](https://docs.python.org/3.8/library/logging.html) or more featured tool like [loguru](https://github.com/Delgan/loguru)
- Testing, because Click [makes it easy enough](https://click.palletsprojects.com/en/7.x/testing/)
- Documentation, it's good to have command and argument description so the command is self-documented. And it's great to have an user guideline how to combine parameters, and how to use your CLI to get different jobs done. [mdBook](https://github.com/rust-lang/mdBook) is an great tool for this since it's just simple markdown, just take a look at [mdBook's documentation](https://rust-lang.github.io/mdBook/)
- Packaging, all python code should be packaged for clear dependency requirement, and one line installation using pip.

Use the below cookiecutter template that implemented this approach as a starting point
```
TODO
```