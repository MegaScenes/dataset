# MegaScenes Dataset
The MegaScenes Dataset is an extensive collection of around 430K scenes and 9M images and epipolar geometries, featuring over 100K structure-from-motion reconstructions from 2M of these images. The images of these scenes are captured under varying conditions, including different times of day, various weather and illumination, and from different devices with distinct camera intrinsics.

# Data Format
The MegaScenes S3 bucket URL is `s3://megascenes/`, and is hosted in the `US-West-2` AWS Region. 

The bucket's directory tree is as follows:
- `s3://megascenes/`
    - `databases/`
        - `main/` 
            - `000/000/` . . . `458/152/`
        - `descriptors/`
            - `000/000/` . . . `458/152/`
    - `images/`
        - `000/000/` . . . `458/152/`
    - `metadata/`
        - `subcat/`
            - `000/000/` . . . `458/148/`
        - `wikidata/`
            - `0/0/0/` . . . `9/9/9/`, `other/`
    - `reconstruct/`
        - `000/000/` . . . `458/150/`
    - `README.md`
 

## `databases/` Directory
This directory houses [COLMAP databases](https://colmap.github.io/database.html) for each scene. COLMAP databases contain tabulated information on images, keypoints, descriptors, matches, and estimated two-view geometries. COLMAP databases use the [SQLite](https://sqlite.org/) format.

The `database/` directory is broken into two subdirectories: 

- `main/` (1.9 TB), which contains `database.db` files
- `descriptors/` (6.8 TB compressed, 8.3 TB uncompressed), which contains `descriptors.db.gz` files

In the two above subdirectories, a scene is represented by its scene-ID number as described in [Scene Folders](#scene-folders).

### Partitioned Databases
For each scene, the COLMAP database is partitioned into two files:
- `database.db`, which is the COLMAP database without the Descriptors table.
- `descriptors.db.gz`, which is the Descriptors table extracted from the COLMAP database as its own SQLite database. It is compressed with [gzip](https://www.gnu.org/software/gzip/).

We separate the Descriptors table since it takes the majority of space in the COLMAP database, and may not contain relevant information for certain applications.

### Example
For a scene with ID `1234`, the database files are as follows:
- `databases/main/001/234/database.db`
- `databases/descriptors/001/234/descriptors.db.gz`

<!--
### Tutorials
We provide a tutorial on how to merge the descriptors into the main database here. TODO
Scripts to query the database  TODO
-->

## `images/` Directory
This directory houses images and image metadata for each scene. A scene is represented by its scene-ID number as described in [Scene Folders](#scene-folders).

The `images/` directory is 3.2 TB.

### JSON Contents
A scene can have any number of subcategories. Each subcategory contains images, a `raw_metadata.json`, a `category.json`, and a `0/category.json`.

Image metadata is represented in `raw_metadata.json`. This json contains a key for each image name, and contains information of various data extracted from Wikimedia Commons, including EXIF data and licensing information.

The scene subcategory name resides in `subcategory_name/category.json`.

A list of image names reside in `subcategory_name/0/category.json`.

### Example
For a scene with ID `1234`, the image files are as follows:
- `images/`
    - `001/234/`
        - `commons/`
            - `subcategory_name_1/`
                - `category.json`
                - `raw_metadata.json`
                - `0/`
                    - `category.json`
                    - `pictures/`
                        - `image1.jpg`
                        - `image2.jpg`
                        - ...
            - `subcategory_name_2/`
                - `category.json`
                - `raw_metadata.json`
                - `0/`
                    - `category.json`
                    - `pictures/`
                        - `image1.jpg`
                        - `image2.jpg`
                        - ...

## `metadata/` Directory
This directory houses metadata for the dataset.

The `metadata/` directory is broken into two subdirectories: 

- `subcat/` (TODO GB), which contains JSON files of the subcategory tree for each scene
- `wikidata/` (TODO GB), which contains JSON files for all Wikidata entries related to a scene or their heirarchical classes.

The `subcat/` subdirectory is organized by scenes, according to a scene-ID number as described in [Scene Folders](#scene-folders).

The `wikidata/` subcategory is organized by Wikidata Q-ID. The first three digits of the Q-ID define the three subfolders that the Wikidata JSON information can be found in. If the Q-ID has less than three digits, then its JSON resides in the `other/` folder. Unlike the scene IDs, this number is NOT zero-padded.

### Example
The subcategory information for a scene with ID `1234` is at `metadata/subcat/001/234/database.db`.

The JSON for a Wikidata item with Q-ID `Q1234` is located at `metadata/wikidata/1/2/3/Q1234.json`.

The JSON for a Wikidata item with Q-ID `Q12` is located at `metadata/wikidata/other/Q12.json`.

<!--
### Tutorials
We provide details on Wikidata TODO
-->

## `reconstruct/` Directory
This directory contains the COLMAP sparse point cloud reconstructions for each scene. The `reconstruct/` directory is organized by scenes, according to a scene-ID number as described in [Scene Folders](#scene-folders).

The `reconstruct/` folder is 429 GB.


### Example

The `reconstruction.tar.gz` file will extact into the scene directory as follows:
- `sparses/`
  - `0/`
    - `images.bin`
    - `cameras.bin`
    - `points3D.bin`
  - `1/`
    - ...
  - ...
 
Note that this is a subset of all released id's

COLMAP sparse reconstructions are represented by numbered subdirectories in `sparse/`.
The COLMAP databases contain information on SIFT keypoints and descriptors for scene images, as well as pairwise epipolar geometries.

## Scene Folders
The dataset partitions scenes using two subfolders. The first subfolder uses the first three digits of the 6-digit zero-padded scene ID. The second subfolder uses the last three digits.

For example, if a scene has an ID of `533`, it is zero-padded to `000533`. This number translates to the directory `000/533/`.

If a scene has an id of `422678`, it translates to the directory `422/678/`.

The data associated with the scene resides in the latter subfolder.
