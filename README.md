# microscope-mosaic

Microscope mosaic is a script used to generate a mosaic image based off a
2-d matrix of images extracted from vk4 Microscopy data files, though
any matrix of images similarly obtained from a microscope or even a 
matrix of images that are precisely and uniformally generated with the 
intent to stitch them together should work.

## Getting Started

Microscope mosaic is meant to be used as a command line script by running the 
microscope_mosaic.py file with approprate arguments. 

### Prerequisites

In order to execute microscope_moscaic the PIL package need to be
installed

Install `PIL`:

```sh
$ pip install PIL
```

### Install

To install, simply download the source file:

* `microscope_mosaic.py`

### Usage (script)

The module `microscope_mosaic.py` can be used as a script with the following required
args:

```sh
$ python3 microscope_mosiac -i <file path to an image> -o <file path for output> 
```

The above command will use a handful of default arguments, it assumes that the 
matrix of images is 15 rows by 20 columns and will use those values to define
the starting and ending x and y values used in the script. And it defaults cropping
to 63 pixels in both the x and y dimension. However, the user can choose to use their
own values using the optional arguments:

*`-cx` or `--cropx` followed by the number of pixels to crop in the x dimension
*`-cy` or `--cropy` followed by the number of pixels to crop in the y dimension
*`-sx` or `--startx` starting index of x (columns)
*`-sy` or `--starty` starting index of y (rows)
*`-ex` or `--endx` ending index of x 
*`-ey` or `--endy` ending index of y


#### Argument options

The input filename must be a valid vk4 file, including the extension .vk4

Options for the output file type argument:

* `csv` (comma separated values)
* `hcsv` (comma separated values with metadata header)
* `jpeg` (image)
* `png` (image)
* `tiff` (image)
*NOTE: Only tiff type output creates valid image files for light and height
data.*

Options for data layers:

* `RGB` (color peak data) 
* `LRGB` (color + light data)
*NOTE: Any combination of L, R, G, B can be used for output (R, RB, LG, LRGB, etc.)* 
* `H` (height data)
* `L` (light data)
*NOTE: L alone refers strictly to light data, L in combination with RGB is color +
light data*

For the argument 

#### Examples

To extract RGB data from the file 'example.vk4' and output that data to the
file 'example_output' (filename extensions are supplied by the script) in jpeg 
image format:

```sh
$ python3 vk4_driver -iexample.vk4 -tjpeg -lRGB -oexample_output 
```

To extract height data from 'example.vk4' and output that data to a csv 
formatted output file with the default filename:

```sh
$ python3 vk4_driver -iexample.vk4 -tcsv -lH 
```

### Usage (module)

Currently vk4extract.py can be used as a module to extract particular data from
a vk4 file for the user to manipualte or analyze as they see fit. In order to 
extract any data from the vk4 file, you must first call the extract_offsets() 
function which returns a dictionary in which the values are the byte offsets 
to the data corresponding to their keys. 


Primary Offset Key | Data Layer 
:----------------- | :--------- 
'meas_conds' | Measurement conditions and metadata
'color_peak' | RGB color data
'color_light' | RGB + light color data
'light' | Light intensity data
'height' | Height data
'string_data' | Metadata refering to file title and microscope lens

Secondary Offset Key  | Data Layer
:-------------------  | :---------
'clr_peak_thumb' | RGB thumbnail data
'clr_thumb' | RGB + light thumbnail data
'light_thumb' | Light thumbnail data
'height_thumb' | Height thumbnail data
'assembly_info' | Microscope Assembly metadata
'line_measure' | ---
'line_thickness' | ---
'reserved' | ---

*NOTE: the values for the last three keys have been zero in all vk4 files I have worked with, indicating there is no data stored for that information, and I am unsure as to what data they might refer to in the case that the offset table in the vk4 file had non-zero values for those offsets*


Once you have the offsets for the vk4 files it is just a matter of reading the
binary data at those offsets, and the vk4extract module provides functions to
extract layers of data from the primary offsets and convert them from binary,
namely:

Function name | Parameters
:------------ | :---------
`extract_measurement_conditions` | offset dict, open vk4 file object 
`extract_color_data` |  offset dict, color type, open vk4 file object
`extract_img_data`  | offset dict, data type, open vk4 file object
`extract_string_data` | offset dict, open vk4 file object

*NOTE: Color type refers to the strings 'peak' (for RGB data) and 'light' (for RGB + light data). Data type refers to the strings 'height' (for height data) and 'light' (for light intensity data)* 


Each of these functions returns a dictionary containing the data pertaining
to a specific layer of data (RGB, RGB + light, Height, etc.) within the vk4 file.

**Measurement Conditions**
Each key in this dict refers to an associated piece of metadata for the microscope or 
for the file itself. For a list of key names refer to the 'VK4layout.ods' file.

**Color Data**
Each key in this dict typically refers to an associated piece of metadata
concerning the data layer in question, except the key 'data' whose value is a
Numpy array the same length as the image size (length * width) where each index
is a list of three 1 byte integers representing the RGB color channels at each
pixel of the image.

**Image Data**
Each key in this dict typically refers to an associated piece of metadata
concerning the data layer in question, except the key 'palette' whose value is
a Numpy array of length 768 (= 256 * 3) containing 8 bit integers, and 'data'
whose value is a Numpy array the same length as the image size where each index
is the unscaled light or height data for each pixel of the image

**String Data**
Each key in this dict typically refers to either the title of the image, or the
name of the microscope lens

*NOTE: each of the above dictionaries will also have a 'name' key whose value is determined by the data layer in question, e.g. when calling `extract_img_data(offset_dict, 'height', vk4_in_file)` the key 'name' will have the value 'Height'*

### Examples

**Create a png image file from RGB data:**

```python
import numpy as np
from PIL import Image
import vk4extract

with open('example.vk4', 'rb') as in_file:
    offsets = vk4extract.extract_offsets(in_file)
    rgb_dict = vk4extract.extract_color_data(offsets, 'peak', in_file)

rgb_data = rgb_dict['data']
height = rgb_dict['height']
width = rgb_dict['width']

rgb_matrix = np.reshape(rgb_data, (height, width, 3))
image = Image.fromarray(rgb_matrix, 'RGB')

image.save('example_image.png', 'PNG')
```

**Create a csv text file from Light data:**

```python
import csv
import numpy as np
import vk4extract

with open('example.vk4', 'rb') as in_file:
    offsets = vk4extract.extract_offsets(in_file)
    light_dict = vk4extract.extract_img_data(offsets, 'light', in_file)
    
light_data = light_dict['data']
height = light_dict['height']
width = light_dict['width']

light_matrix = np.reshape(light_data, (height, width))

with open('example_light.csv', 'w', newline='') as out_file:
    writer = csv.writer(out_file, delimiter=',')
    for row in light_matrix:
        writer.writerow(row)

```















