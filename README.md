# phy format

**This is an early draft.**


* Root directory: `/home/user/spikesorting/20160101/`
* All paths below are relative to this directory.

## Raw data

### Files

```
20160101_1.ns5
20160101_2.dat
```

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


## Experiment files

* The subdirectory `20160101.phy/`, named **phy directory**, represents a single dataset.
* `20160101.phy/20160101.metadata` is the **main metadata file**.
* All `.metadata` files in subdirectories are recursively merged in a single JSON object.
    * TODO: how to deal with conflicts? I suggest to raise an error (strict behavior).
    * TODO: how to deal with collections containing multiple objects with the same key?
* Every object has a unique id which **must be automatically generated by pymongo**.

Some metadata objects (by default, all are in the main metadata file):

```
{"experiment": {
    "_id": 098274365020,
    "subject_name": "ns20141202",
    "datetime": "2016-01-01T11:01:37Z"  # mongodb uses ISO 8601
    }
}

{"traces": {
    "experiment_id": 098274365020,
    "files": ["traces/20160101_1.ns5",
              "traces/20160101_2.dat"]
    }
}

{"spikes": {
    "_id": 102784512155,  # there can be multiple spikes objects = multiple runs of SpikeDetekt
    "experiment_id": 098274365020
    "spike_times": {"path: "spike_times/"}, # an array or an object with "path"
    "spike_widths: [0.34, 0.57, 0.34, ...],
    }
}

{"clusters": {
    "_id": 987513216832,  # there can be multiple clusters objects = multiple clusterings
    "experiment_id": 098274365020
    "spike_clusters": {"path": "spike_clusters/"},
    "cluster_groups": [0, 2, 1, 0, 3, 1, 0, ...]
    }
}

# TODO: a JSON object **cannot** contain two objects with the same key
* but mongoDB can: it's the concept of collection
* in which files should we store these objects?
{"clusters": {
    "_id": 129898408501,  # there can be multiple clusters objects = multiple clusterings
    "experiment_id": 098274365020
    "spike_clusters": {"path": "spike_clusters_klustakwik/"},
    }
}
```


## Data arrays

* An array is either:
    * a JSON array,
    * or an object with a `path` attribute.

* The `path` refers to an **array subdirectory** containing a set of files.
* The `path` is relative to the phy directory.
* Every array is defined by one or several files in the array subdirectory. The files share the same basename.
* (Optional) The `.data` or `.dat` file is a flat binary array with no header.
* (Optional) The `.ns5` (or any other supported file) is a binary array that can be read by the `read_ns5()` function (or similar).
* (Optional) The `.metadata` file contains some metadata about the array. TODO: do we need this?
* (Optional) The `.format` file contains some information required by the `read_data()` function.

Examples:

```
20160101.phy/traces/20160101_1.n5  # symlink to ./20160101_1.n5

20160101.phy/traces/20160101_2.dat  # symlink to ./20160101_2.dat
20160101.phy/traces/20160101_2.format
{
    "file_format": "dat",
    "byte_offset": 0,
    "data_type": "int16",
}
```

## API

* TODO: how the user specifies the parameters/probe (PRM? PRB)? Where these files should be stored? Should the parameters be imported in the JSON? => duplication problem. Any traces/spike detection/clustering parameter should be stored in a single file.
