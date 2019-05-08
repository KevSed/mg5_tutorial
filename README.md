# Mad Graph 5 tutorial

## Introduction
In Mad Graph 5 the general syntax for parameter setting is PARAM=1 (PARAM <= 1) and PARAM==1 (PARAM = 1).

## Doing EFT physics

To use EFT operators in top-physics import e.g. the dim6 top model within your mg5 session. You may need
to download the model first.

```
> import model dim6top_LO_UFO
```

The available parameters are:
..* FCNC: configuration of FCNC interactions (=0 excludes FCNC interactions).
..* DIM6^2: coupling order of the dim6 EFT operators at the squared level (=1 e.g.: cross-section only
contains linear Wilson-coefficients, i.e. the SM-EFT interference terms).
..* DIM6: coupling order at amplitude (diagram) level (= max. number of EFT vertices).
..* QED: max. allowed QED coupling order (default 0)

To generate a process at parton level (e.g. g g -> t t~) use:

```
> generate g g > t t~
```

To display all generated diagrams use:

```
> display diagrams
```

which will open a pdf file with the Feynman diagrams with the EFT operator vertices highlighted by a blob.
To then safe the output use:

```
> output [name_of_process_output]
```

Then a run can be launched, e.g. a testrun:

```
> launch --name=testrun
```

This gives you the opportunity to switch specific programs on or off, like the showering, detector
simulation, etc. Also you can modify the param_card and the run_card. The param card contains all
used SM parameters and the configuration for the EFT coefficients (which ones to use and what value to give
them). Among others are the coupling constants, masses and Yukawa couplings.
The run card contains specifics for the generated events, like the number of events, the beam energies, the
collider type (pp, pp~, no pdf at all, etc). You can also set cuts on certain variables here. To do all this,
you can either edit the files (by chossing 1,2, according to the displayed info) or you can use the `set`
command to set certain parameters. The autocompletion `tab` shows you the available parameters. This example
sets a symmetric beam energy of 13TeV:

```
> set ebeam1 6500
> set ebeam2 6500
```

After executing, the variables and the generated cross-section are displayed in your browser. To set
EFT-coefficients more permanently for a certain model, you can create a `restrict_default.dat` in the
model's directory. To generate a generic one in dim6 you can use a script within the model directory:

```
~> python2 write_param_card.dat $$ mv param_card.dat restrict_default.dat
```

The file can then be modified and will set the default parameters every time the model is used. Any other
card of the type `restrict_[name].dat` can be loaded by appending a `-[name]` when loading the model:

```
> import model dim6top_LO_UFO-[name]
```

When you exit an mg5 session, your parameters and configurations from within the session are lost. Of course
you can save them in the run and param cards. But if you want to redo a generation you can restart generated
processes. To easily "re-enter" the session, you can execute `bin/madevent` within the desired process
directory.
Also, to automate Mad Graph processes, you can use scripts of the following fashion.
1. generate a file `whatever.mg5`
2. fill in the mg5 commands as if you were in a session
3. then you can call the commands e.g. if you want to run a process:

```
~> bin/madevent whatever.mg5
```
