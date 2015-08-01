# phy format

**This is an early draft.**

* Root directory: `/home/user/spikesorting/20160101/`
* All paths below are relative to this directory.


## Raw data

### API

Looking at the traces in the terminal:

```
phy traces 20160101_1.ns5
phy traces 20160101_2.dat --n-channels=128 --dtype=int16 --sample-rate=20000
```

Looking at the traces in IPython:

```python
import phy.io
import phy.plot as plt

traces = phy.io.read_traces(['20160101_1.ns5', '20160101_2.dat'])
plt.plot_traces(traces)
plt.show()
```


## File structure

```
/20160101_1.ns5
/20160101_2.dat
/metadata.json
[/params.prm]

/main/0/params.json  # refers to basename of dat files
/main/0/clusters.json  # metadata about JSON
/main/0/spike_times.npy
/main/0/spike_clusters.npy
/main/0/mean_waveforms.npy
/main/0/raw_0.ns5  # symlinks to ../20160101_1.ns5...
/main/0/raw_1.dat

/test/0/...

```


## Data arrays

* An array is either:
    * a JSON array,
    * or an object with a `path` attribute.

* The `path` refers to a file
* The `path` is relative to the phy directory.
* Every array is defined by one file
* The file extension defines the format for the array
* Supported formats: txt, npy, ns5, Neo formats <http://neo.readthedocs.org/en/latest/io.html>...
* New tar format: first archived file is format.ini with comment with link to description, array description, then one or several flat binary files
    * array description in ini: dtype, shape, labels


## API

* TODO: how the user specifies the parameters/probe (PRM? PRB)? Where these files should be stored? Should the parameters be imported in the JSON? => duplication problem. Any traces/spike detection/clustering parameter should be stored in a single file.

```

_read_array(path)
_write_array(path, array)


exp = Experiment(metadata_path=None, metadata_dict=None, base_dir=None)

exp.base_dir  # the path (absolute or relative) to the root.
              # all relative paths are made relative to the base dir

exp.metadata  # the JSON dictionary
exp.read_metadata(alias_or_path)  # return a dict
exp.write_metadata(alias_or_path, metadata)  # update the current metadata
                                             # dict and save it

exp._resolve_alias(alias)  # return a relative path
exp.read_array(alias_or_path)  # return an array
exp.write_array(alias_or_path, array)

Examples
exp.read_array('0/spike_times')
exp.read_array('0/spike_times')

```
