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

* `-cx` or `--cropx` followed by the number of pixels to crop in the x dimension
* `-cy` or `--cropy` followed by the number of pixels to crop in the y dimension
* `-sx` or `--startx` starting index of x (columns)
* `-sy` or `--starty` starting index of y (rows)
* `-ex` or `--endx` ending index of x 
* `-ey` or `--endy` ending index of y

e.g.

```sh
$ python3 microscope_mosiac -i ./images/FY09_DE09_270_Y1_X1.png -o mosaic.png -cx 70 -cy 90 -sx 1 -sy 1 -ex 10 -ey 12 
```




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
    light_dict = vk4extract.extract_img_data(offsets, 'https://tecadmin.net/install-python-3-7-on-ubuntu-linuxmint/light', in_file)
    
light_data = light_dict['data']
height = light_dict['height']
width = light_dict['width']

light_matrix = np.reshape(light_data, (height, width))

with open('example_light.csv', 'w', newline='') as out_file:
    writer = csv.writer(out_file, delimiter=',')
    for row in light_matrix:
        writer.writerow(row)

```















