Intro
=====

Recently, folks (mainly me) have been struggling with issues with the M1 chip as many packages and software only 
support the Intel chip or have specified versions for the Intel chip versus M1 chip. This has been causing many import issues
and is further complicated by - again - the processor-dependency of package management software (such as Conda) and IDEs.

This document will go over how to navigate some of these issues by setting up two environments: one compatible with the
Intel chip (x86_64) and one with the Silicon/M1 chip (arm64). We will also be using [venv](https://docs.python.org/3/library/venv.html),
the Python virtual environment from Python itself. I chose to do this because I was running into other compatibility issues between
[Anaconda's](https://www.anaconda.com/) Intel versus Silicon versions.

Rosetta 2
=========

[Rosetta 2](https://support.apple.com/en-us/HT211861) comes directly with every Mac that has an Apple silicon chip. It works as a translator 
between arm64 and x86-64. For our use, we will want to use the Terminal with _and_ without Rosetta (a.k.a. in arm64 mode or in x86_64 mode).
To enable Rosetta, right click the application you intend to open, click _Get Info_ and check the _Open using Rosetta_ box. Now, opening the 
selected application will automatically launch it in Rosetta mode.

Previously, you could duplicate the Terminal app (right click the application -> Duplicate), but this has been disabled by Apple in more
recent updates. This would be handy as you could rename one as _"Terminal Rosetta"_ and have Rosetta enabled for that copy of Terminal.
In our case, we will have to manage with a workaround.

We will add the following to lines to the end of your `~/.zshrc` file. [[source](https://stackoverflow.com/questions/74198234/duplication-of-terminal-in-macos-ventura)]

```
alias arm="env /usr/bin/arch -arm64 /bin/zsh --login" 

alias intel="env /usr/bin/arch -x86_64 /bin/zsh --login"
```

Make sure to load your edited file:

```
source ~/.zshrc
```

What this allows you to do is to directly type either `arm` or `intel` commands into your shell to activate the corresponding 
processors, essentially activating or deactivating Rosetta.

```
serenashi@brb-08769 ~ % arm
serenashi@brb-08769 ~ % uname -p
arm
serenashi@brb-08769 ~ % intel
serenashi@brb-08769 ~ % uname -p
i386
```

The `uname -p` command above prints the processor your terminal window is currently operating with. Note that `i386` is printed 
after `intel` is executed; i386 belongs to the set of x86 architecture and has support for 32-bit instructions. x86-64 is essentially
an extension of the x86 architectures that supports 64-bit instructions in addition. Note that the `arch` command also works to display
the current architecture.

Great! Now that we can switch between the two processors, let's get the virtual environment set up.

Setting up the virtual environment
==================================

To create a venv directly from command line, use the command `venv`:

```
python -m venv /path/to/new/virtual/environment
```

This creates a target directory that will house the created virtual environment (a common name for the environment is `.venv`). 
It will contain a key pointing to the Python installation from which the command was run (i.e., the `python` portion of the command).

Now that the environment is created, to activate it within the shell you are in (`bash`/`zsh`):

```
$ source <venv>/bin/activate
```

For more information on venv, take a look at [the API provided by Python](https://docs.python.org/3/library/venv.html).

For my specific case, I use [Pycharm](https://www.jetbrains.com/pycharm/) as my IDE and therefore directly create a new venv environment 
when defining my _Project Interpreter_. Pycharm also allows me to select the Python version I would like to use.

Installing processor-specific packages
======================================

Now that we know how to create a virtual environment, we can install the packages we want. Make sure that **two distinct venv environments**
are created and that you have the **correct environment and correct processor** activated.

First, let's make sure that our environment is set up as expected. _This step is important!_ I spent a lot of time debugging my environments
here as it turned out all my packages had actually been installed to a separate environment directory even though I was working with the correct
one activated.

```
(venv) serenashi@brb-08769 processing % which python
/Users/serenashi/PycharmProjects/processing/venv/bin/python
```

The `which python` command will return the path to the Python executable that was originally used to create the venv. If the path here
does **not** correspond to the venv directory for this environment, you have done something wrong and should retrace your steps here.
If it's the correct path, we can proceed to installing your packages.

Navigate to the project directory where you have your `requirements.txt` file. We will use `pip` to install all packages:

```
pip install -r requirements.txt
```

If you have done everything correctly, you should see the correct processor-specific packages installed according to the processor you are
running with. I like to verify this with the `numpy` package:

```
(venv) serenashi@brb-08769 processing % arch
arm64
(venv) serenashi@brb-08769 processing % pip install numpy
Collecting numpy
  Obtaining dependency information for numpy from https://files.pythonhosted.org/packages/5c/ff/0e1f31c70495df6a1afbe98fa237f36e6fb7c5443fcb9a53f43170e5814c/numpy-1.26.0-cp310-cp310-macosx_11_0_arm64.whl.metadata
  Using cached numpy-1.26.0-cp310-cp310-macosx_11_0_arm64.whl.metadata (53 kB)
Using cached numpy-1.26.0-cp310-cp310-macosx_11_0_arm64.whl (14.0 MB)
Installing collected packages: numpy
Successfully installed numpy-1.26.0
```

```
(venv) serenashi@brb-08769 processing % arch
i386
(venv) serenashi@brb-08769 processing % pip install numpy
Collecting numpy
  Obtaining dependency information for numpy from https://files.pythonhosted.org/packages/be/f8/034752c5131c46e10364e4db241974f2eb6bb31bbfc4335344c19e17d909/numpy-1.26.0-cp310-cp310-macosx_10_9_x86_64.whl.metadata
  Using cached numpy-1.26.0-cp310-cp310-macosx_10_9_x86_64.whl.metadata (53 kB)
Using cached numpy-1.26.0-cp310-cp310-macosx_10_9_x86_64.whl (20.6 MB)
Installing collected packages: numpy
Successfully installed numpy-1.26.0
```

You can see from the snippits above that according to the architecture you have activated, either the `arm64` or `x86-64` version of `numpy` is 
installed correspondingly.

Switching between arm64 and x86_64
==================================

With the environments setup, switching from one processor to the other is as simple as activating the corresponding environment and setting your 
shell processor (`arm` and `intel` commands). Congrats! You made it to the end!

PyCharm specific information
============================

If you are using PyCharm like me, the IDE is unfortunately not well-integrated with Rosetta at the moment. This means we have to use
some workarounds in order to have Pycharm run with x86. (Note that no changes need to be made to run PyCharm with arm64.)

To have the terminal in PyCharm run with Rosetta by default, go to _PyCharm -> Settings -> Tools -> Terminal -> Application Settings -> Shell 
path_. Replace the path with 
[[source(https://intellij-support.jetbrains.com/hc/en-us/community/posts/360010485779-is-it-possible-to-run-the-terminal-through-rosetta-for-arm-)]:

```
env /usr/bin/arch -x86_64 /bin/zsh --login
```

Note that the `intel` and `arm` commands from the first part will still work in PyCharm's terminal as well.

To set up PyCharm to run scripts with Rosetta, create a new file named `pythonM1` in `<venv>/bin/` and paste the following in the file 
[[source](https://youtrack.jetbrains.com/issue/PY-46290/Allow-running-Python-under-Rosetta-2-in-PyCharm-for-Apple-Silicon)]: 

```
#!/usr/bin/env zsh
mydir=${0:a:h}
/usr/bin/arch -x86_64 $mydir/python "$@"
```

In terminal, modify the permissions of the file to make it an executable:

```
chmod +x pythonM1
```

Then, edit the PyCharm _Python Interpreter_ and set the path of the _Existing Environment_ as `<venv>/bin/pythonM1`, the file created above.
This sets the interpreter to use the x86 architecture with the Python executable that the venv was set up with.

Conclusion
==========

Thanks for following until the end! I couldn't personally find any comprehensive tutorials or guides on how to do this, so I just decided to write 
it up. Hopefully this helped some of you out with setting up your environments to run with arm64-unfriendly code.
