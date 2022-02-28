# Ultimate CLI

This repository will serve the purpose of documenting my research of the "Ultimate CLI" problem, i.e. how to create superb CLI experiences with very little thinking and code, for various programs, utilities and frameworks that I invariably end up building.

Requirements:

1. Minimum code. Every line counts! While creating your new framework you don't want to be thinking about HOW to give it a nice CLI and googling for solutions, but just dropping a few lines and that is it.
2. Maximum flexibility to do whatever you need in terms of the arguments etc. in the CLI, so that the framework does not fall apart if you go beyond the examples they provide.
3. Simple things should be simple. Bash is simple. Everyone remembers how to do the basic stuff in bash. If basic stuff is hard to do in framework X, people fall back to bash.
4. Good integration with bash, if we can, since bash is great for grep etc.
5. (optional) Enable code reuse with tuttest.

## Typer + click-repl (current solution)

[Typer](https://github.com/tiangolo/typer) is a somewhat involved CLI generator based on click, but it just works.
I suspect something simpler could be built based on click but that would require a non-negligible amount of work.

It does not support interactive shells, but I found a way to workaround that using [click-repl](https://github.com/click-contrib/click-repl).
See also [here](https://github.com/tiangolo/typer/issues/185).

### Reference code

#### Command

```python
import typer
app = typer.Typer(help="A test app", add_completion=False)

@app.command()
def test(test_arg: str):
    "Test command"
    print(f"Test, the argument is {test_arg}.")
```

Then:

```python
if __name__ == "__main__":
    app()
```

#### Initialization + global arguments 

```python
@app.callback(hidden=True)
def some_init_function(global_arg: str="global_arg"):
    print("Initializing stuff, global_arg = {global_arg}")
```

#### Interactive shell (+ command with alias)

```python
import click
from click_repl import repl

@app.command("i")
def interactive():
    "Run in interactive mode"
    click.echo("Running in interactive mode. This supports tab completion.")
    click.echo("Use ':help' for help info, or ':quit' to quit.")
    repl(click.get_current_context())
```

### Disadvantages

* when you don't provide the subcommand, the default help does not list the subcommands (is this fixable? it must be!)
* the decorated function names are not available as commands, i.e. aliases supersede the main subcommand function name (perhaps this is fixable too; a workaround is to register the function twice)
* you sometimes will need to use Typer.argument instead of normal Python objects for completion to work in interactive shell
* interactive shell shows the full help (with program name etc.) every time you make a mistake (fixable?)
* help/quitting is weird in the interactive shell (fixable? but would need to dig into click-repl)

Once some more fixes are found, we will probably need a new package to hide the complexity.

### Example usage (tablist)

For some example usage, see a rudimentary utility I created, [tablist](https://github.com/mgielda/tablist/blob/master/tablist.py).
It actually does something in under 100 lines of code.

## plac

Plac is as awesome as it is unpopular and arcane.

See:

* [batch scripts](https://plac.readthedocs.io/en/latest/#plac-batch-scripts)
* [runner](https://plac.readthedocs.io/en/latest/#the-plac-runner)
* [automated tests](https://plac.readthedocs.io/en/latest/#plac-easy-tests)
* [server](https://plac.readthedocs.io/en/latest/#the-plac-server)

Tried to use it several times, but always fall into the same pitfall:

* convoluted documentation (it could be improved by separating it into multiple chapters etc. but it's a non-trivial amount of work)
* so-so support for [subcommands](https://plac.readthedocs.io/en/latest/#implementing-subcommands) -- you need to either self-import a module or create a separate class; this is too complex, and subcommands are my main requirement
* some arbitrary limitations just because (which may hit you for advanced use cases, but you can work around them with `argparse` which is used underneath; this however sacrifices the simplicity)
* arcane internals (which is understandable, since it has so many cool features)

It is a great inspiration, but for now remains well, inspiration.

## argh

[Argh](https://argh.readthedocs.io/en/latest/tutorial.html) is nice. I used to use argh. I think where it failed me was stuff like general options not tied to any subcommand etc. Perhaps of course it was me not knowing how to use argh well enough.
Also no interactive shells and fancy stuff to speak of ;-)
Anyway it remains a very decent choice.

## fire

While [fire](https://github.com/google/python-fire) is a cool project, it lacks some capabilities for the use cases that I typically hit (at least last I checked) and does not offer any of interactive shell and other advanced features.

## ipython / Jupyter

With its seamless bash integration, magics, vast ecosystem etc, Jupyter seems like an excellent project to base upon here.
However, it seems nobody ever seriously thought of using Jupyter for CLI-driven task automation (or I am just looking in the wrong places).

And `ipython` seems to crash for me whenever used with anything remotely similar to plac, Typer or argparse for that matter (although workarounds exist).

So perhaps there is promise here, but failed to find it so far, will continue looking.

## xonsh and similar solutions

I don't know -- I am not really looking for a new shell, exactly, more a simple task automation solution.
May revisit in the future.

Xonsh seems to run e.g. `tablist` properly, **including** the interactive shell, but to get the xonsh magic, you will need to explicitly use the `.xsh` extension. So this is promising!

Question is, if super-seamless bash integration is really the main requirement to warrant including xonsh as a dependency.

## bpython

[bpython](https://github.com/bpython/bpython) seems like a cool idea at first glance.
With the syntax highlighting and code completion, argument suggestions etc. it looks really promising, but there seems to be no way to use bpython ON a program to get a nice CLI, with only the commands the program itself offers.

If someone knows of a utility like that, it would indeed be pretty cool.
Also, the considerations about `xonsh` apply.

## JavaScript solutions

While there are many JS task automation solutions (perhaps too many), generally I feel somewhat reluctant to a) use JavaScript (but I can be convinced) and b) pull in Node as a dependency everywhere.

For bash integration, there is the excellent [zx](https://github.com/google/zx) at least.
