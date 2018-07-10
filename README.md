# vk4_driver

vk4_driver is a tool used to extract data from Keyence profilometry vk4 format
data files and provide useful output formats for analysis (csv, tiff, jpeg, 
png).

## Getting Started

The vk4_driver is meant to be used as command line script by running the 
vk4_driver.py file with approprate arguments. However, it can also be 
used as a module providing a flexible set of tools to process data from
vk4 files 

### Prerequisites

In order to execute the vk4_driver the NumPy and PIL packages need to be
installed

First, install `numpy`:

```sh
$ pip install numpy
```

Next, install `PIL`:

```sh
$ pip install numpy
```

### Install

To install, download the 4 source files:

* `VK4container.py`
* `vk4extract.py`
* `vk4out.py`
* `vk4_driver.py`

### Usage

First, the module `vk4_driver` can be used as a script with the following args:

```sh
$ python3 vk4_driver -i<vk4 filename> -t<type of output> -l<layers of data> 
    -o<optional output filename>
```






