# Benchmarking PPSeq and Elephant ASSET

## Install the project

1. [Download and install Julia](https://julialang.org/downloads/).

```bash
wget https://julialang-s3.julialang.org/bin/linux/x64/1.5/julia-1.5.3-linux-x86_64.tar.gz
tar zxvf julia-1.5.3-linux-x86_64.tar.gz
```

Add the path to `julia-1.5.3/bin/` in the `PATH` environment variable by adding the following line in the `.bashrc` file: 

```bash
export PATH=$PATH:/path/to/julia-1.5.3/bin/
```

where `/path/to/julia-1.5.3/bin/` is the path to Julia bin directory on your laptop where you've downloaded it.

To apply the changes, run `source ~/.bashrc` command in a terminal.

2. Clone this repository

```bash
git clone https://github.com/lindermanlab/PPSeq.jl.git
cd PPSeq
```

and run the `julia` command in a terminal. Inside the terminal, hit the `]` key to activate the package mode of Julia. Then run `activate .` (similar to `conda activate env`) and `instantiate` commands.

```
(@v1.5) pkg> activate .
 Activating environment at `~/PycharmProjects/other/PPSeq.jl/Project.toml`

(PPSeq) pkg> instantiate
   Updating registry at `~/.julia/registries/General`
  Installed Rmath_jll ──────────────────── v0.2.2+1
  Installed SortingAlgorithms ──────────── v0.3.1
  Installed PyPlot ─────────────────────── v2.9.0
  Installed StatsFuns ──────────────────── v0.9.6
  Installed SpecialFunctions ───────────── v1.3.0
  Installed DataStructures ─────────────── v0.18.9
  Installed StatsBase ──────────────────── v0.33.3
  Installed Distributions ──────────────── v0.24.15
  Installed Conda ──────────────────────── v1.5.0
  Installed Compat ─────────────────────── v3.25.0
  Installed Reexport ───────────────────── v1.0.0
  Installed JSON ───────────────────────── v0.21.1
  Installed PDMats ─────────────────────── v0.11.0
  Installed FixedPointNumbers ──────────── v0.8.4
  Installed OpenSpecFun_jll ────────────── v0.5.3+4
  Installed ColorTypes ─────────────────── v0.10.10
  Installed Colors ─────────────────────── v0.12.6
  Installed PyCall ─────────────────────── v1.92.2
  Installed Parsers ────────────────────── v1.0.16
  Installed OrderedCollections ─────────── v1.4.0
  Installed VersionParsing ─────────────── v1.2.0
  Installed FillArrays ─────────────────── v0.11.5
  Installed QuadGK ─────────────────────── v2.4.1
  Installed LaTeXStrings ───────────────── v1.2.1
  Installed Artifacts ──────────────────── v1.3.0
  Installed JLLWrappers ────────────────── v1.2.0
  Installed Rmath ──────────────────────── v0.6.1
  Installed DataAPI ────────────────────── v1.6.0
  Installed Missings ───────────────────── v0.4.5
  Installed CompilerSupportLibraries_jll ─ v0.3.4+0
  Installed MacroTools ─────────────────── v0.5.6
  Installed ChainRulesCore ─────────────── v0.9.29
Downloading artifact: Rmath
Downloading artifact: OpenSpecFun
Downloading artifact: CompilerSupportLibraries

```

To be able to execute Julia notebooks, run

```
(PPSeq) pkg> add IJulia
```

Then you can open the notebooks in the "demo" folder as you do with Python notebooks: `jupyter notebook demo/songbird.ipynb`.

Finally, install PyCall in Julia:

```
(PPSeq) pkg> add PyCall
```


3. Install PyJulia.

```
pip install julia
```

4. Install Elephant.

```
pip install elephant[extras]
```

If you want to benchmark PPSeq with Elephant ASSET, it's highly recommended installing Elephant with optimized OpenCL support as described [here](https://elephant.readthedocs.io/en/latest/install.html).

## Data

We'll use [ASSET tutorial data](https://gin.g-node.org/INM-6/elephant-data/src/master/dataset-2). You can download it as a Nix file with the following command:

```
curl https://web.gin.g-node.org/INM-6/elephant-data/raw/master/dataset-2/asset_showcase_500.nix --output asset_showcase_500.nix --location
```

## PPSeq Input format

PPSeq requires the input to be represented as a vector of `Spike`s, and `Spike` is a structure, defined in `structs.jl`, that

```julia
"""
Holds neuron index and timestamp for each spike.
"""
struct Spike
    neuron::Int64
    timestamp::Float64
end
```

Therefore, we need to convert a python list of `neo.SpikeTrain`s to a julia vector of `Spike`s.

You can load a Nix file in Python, save it to a file format similar to `data/songbird_spikes.txt`, and load it back in Julia.

```python
def dump_spiketrains(spiketrains, units='s'):
    # spiketrains is a list of neo.SpikeTrain
    spikes = [st.times.rescale(units).magnitude for st in spiketrains]
    spike_ids = [[i] * len(st) for i, st in enumerate(spiketrains, start=1)]
    spikes = np.c_[np.concatenate(spike_ids), np.concatenate(spikes)]
    np.savetxt("asset_spikes.txt", spikes, fmt=["%d", "%.6f"], delimiter='\t')
```

But, of course, such approach is not recommended because it involves disk I/O operations.

A better approach is in-memory Python <-> Julia communication (to be done).

## Caveats

Julia, unlike the majority of the languages, is 1-based instead of 0-based. This means that arrays start at index 1.
