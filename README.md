# Mad Graph 5 tutorial

## Introduction
In Mad Graph 5 the general syntax for parameter setting is PARAM=1 (PARAM <= 1) and PARAM==1 (PARAM = 1).

## Doing EFT physics

To use EFT operators in top-physics import e.g. the [dim6 top model](https://feynrules.irmp.ucl.ac.be/wiki/dim6top) within your mg5 session. You may need
to download the model first.

```
mg5> import model dim6top_LO_UFO
```

The available parameters are:
* **FCNC**: configuration of FCNC interactions (=0 excludes FCNC interactions).
* **DIM6^2**: coupling order of the dim6 EFT operators at the squared level (=1 e.g.: cross-section only
contains linear Wilson-coefficients, i.e. the SM-EFT interference terms).
* **DIM6**: coupling order at amplitude (diagram) level (= max. number of EFT vertices).
* **QED**: max. allowed QED coupling order (default 0)

To generate a process at parton level (e.g. g g -> t t~) use:

```
mg5> generate g g > t t~
```

To display all generated diagrams use:

```
mg5> display diagrams
```

which will open a pdf file with the Feynman diagrams with the EFT operator vertices highlighted by a blob.
To then safe the output use:

```
mg5> output [name_of_process_output]
```

Then a run can be launched, e.g. a testrun:

```
mg5> launch --name=testrun
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
mg5> set ebeam1 6500
mg5> set ebeam2 6500
```
same is achieved via
```
mg5> set lep 13000
```

Also to set the running coupling to the value at a certain renormalization energy use

```
mg5> set fixed_scale 91.1876
```

to e.g. set it to the Z boson mass.

After executing, the variables and the generated cross-section are displayed in your browser. To set
EFT-coefficients more permanently for a certain model, you can create a `restrict_default.dat` in the
model's directory. To generate a generic one in dim6 you can use a script within the model directory:

```
~> python2 write_param_card.dat $$ mv param_card.dat restrict_default.dat
```

The file can then be modified and will set the default parameters every time the model is used. Any other
card of the type `restrict_[name].dat` (with `[name]` not containing a `-`) can be loaded by appending a `-[name]` when loading the model:

```
mg5> import model dim6top_LO_UFO-[name]
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

The way to exactly reproduce a generated process is shown in the file `proc_card_mg5.dat`.
To safe all results for a process you can call the method `print_results`.

```
mg5> print_results --path=./results.txt --format=short
```

From there you can e.g. read in the cross-sections for further analyses or plots. The file
contains the run name, the tag, the cross section, the uncertainty, the number of
generated events.

To remove all runs in a process you can start a session with `bin/madevent` within the
directory and execute

```
mg5> remove all banner -f
```

## Mad Analysis

[MadAnalysis 5](https://www.sciencedirect.com/science/article/pii/S0010465512002950) is a
single efficient framework for phenomenological analyses. It can read in several MC
simulator outputs and analyse the data. The general format for storing (parton-level)
events and their properties is the `Les Houches Event` (lhe) format. It's a single
file following an XML like structure.

MC-based analyses at parton-level generally use the following steps:
1. parsing lhe event files of signal and background processes
2. implementing various selection cuts on the objects
3. creating histograms representing kinematic quantities

These histograms can e.g. be used for optimization studies (only on parton-level, if no
detector-simulation involved). Showering and all the other processes are done e.g. by
PYTHIA / HERWIG etc and can be applied to lhe files.
Running a hadron-level analysis is much easier and faster after the particles are
clustered to jets (e.g. using FASTJET). Detector-simulations like DELPHES or PGS4 create
the reconstructed objects. There are fast detector-simulations that don't run a full
detector-simulation, but yield reasonable results to (not) motivate certain full
simulations.

MadAnalysis5 also contains an expert mode that can be used to implement your own
analysis directly within the `C++` kernel. Data formats that MA5 accepts are among
others `LHE`, `STDHEP`, `HEPMC` and `LHCO`. It's compatible with almost all data-levels
and able to create histograms, display the results in latex or html, run selections and,
if contained in the files, display the whole interaction history of a particle.

### Sample analysis within MadAnalysis5

Analyses within MA5 always follow the same scheme:
1. **Sample declaration:** gather (multiple) signal and background datasets in an object called `dataset`
2. **Particle content of events:** associate PDG-id to more intuitive terms as `e+ e-` or define terms like `e (=e+ e-)`.
3. **Definition of the analysis:** define histograms to be made and selection cuts (called the selection). The ordering of the cuts applied, of course, affects intermediate events.
4. **Running the analysis:** MA5 generates C++ code for the analysis which is called a job which is executed by MA5.
5. **Display of results:** MA5 generates a report with several results and the histograms

### First steps with MadAnalysis5 (standalone)

For further information look among others [here](https://arxiv.org/pdf/1206.1599.pdf) and [here](http://madanalysis.irmp.ucl.ac.be/).
Depending on the location of the MA5 installation the particle content changes. E.g. if
installed inside a MadGraph installation it automatically uses MG5's particle content.
Generally, the content can be displayed within an MA5 session via:

```
ma5> display_particles
ma5> display_multiparticles
```

To define own particles, e.g. `mu` as both charged muons use:

```
ma5> define mu = mu+ mu-
```

To load MC samples to a MA5 session, e.g. all samples within the directory `samples`:

```
ma5> import samples/*.lhe.gz
```

This creates a unique event sample denoted by `defaultset`, containing **all** events.
To then e.g. apply cuts, use:

```
ma5> reject MET > 100
```

This will cut out all events with missing Et > 100 GeV. Another example, where
events containing mu+ or mu- with Pt < 20 GeV are cut out.


```
ma5> reject (mu) PT < 20
```

To tell MA5 to generate plots you can then use

```
ma5> plot M(mu) 20 0 100
```

Or to generate a preview of that plot use

```
ma5> preview <hist>
```

This example tells MA5 to plot the mu+ mu- invariant mass in 20 bins ranging
from 0 to 100 GeV. To finish the selection and execute it, you can create a job.
It will created in the directory you specify:

```
ma5> submit <dirname>
```

which will contain a series of `C++` source and header files.

### Using MadAnalysis5 within MadGraph5

Using MadAnalysis5 within MadGraph5 is really easy. You just need to make sure, MadGraph
has access to MadAnalysis, by e.g. installing it directly in MadGraph. To do so, open a
MadGraph session and do

```
mg5> install MadAnalysis5
```

Then you can generate a process, output it and launch it, which will now allow you to
also run MadAnalysis on the output. It will create a default number of plots which can be
amended within the MG5 session or by editing the file `madanalysis5_parton_card.dat`. The
output pdf is stored in the run directory of the process' corresponding `Events`
directory.

Furthermore, standalone `python` and `C++` files for every histogram are created. You
can find them in the process' directory under `HTML/<runname>/tag_<*>/Output/Histos/MadAnalysis5job_<*>/`.

To make sure, the spin correlations (important for the decays) are treated the right way, it is
best, to specify the decay chain like this:

```
mg5> p p > t t~ , (t > w+ b, ...)
```


### Useful references (cumulated)

* https://github.com/kenmimasu/RootAnalyser
* https://arxiv.org/pdf/1802.07237.pdf
* https://arxiv.org/pdf/1405.0301.pdf
* https://www.sciencedirect.com/science/article/pii/S0010465512002950
* http://madanalysis.irmp.ucl.ac.be/
* https://arxiv.org/pdf/1206.1599.pdf
