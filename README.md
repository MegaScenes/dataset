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
        - `000/000/` . . . `458/152/`
    - `reconstruct/`
        - `000/000/` . . . `458/150/`
    - `README.md`
 
## `databases/` Directory
This directory houses [COLMAP databases](https://colmap.github.io/database.html) for each scene. COLMAP databases contain information on images, keypoints, descriptors, matches, and estimated two-view geometries. The databases use the [SQLite](https://sqlite.org/) format.

We extract Descriptors table of each COLMAP database into its own SQLite database then gzip it, due to how it takes the majority of space in a COLMAP database. Consequently, the `database/` directory is broken into two subdirectories: 

- `main/` (1.9 TB), which contains the COLMAP database without the Descriptors table
- `descriptors/` (6.8 TB compressed, 8.3 TB uncompressed), which contains all gzipped Descriptors tables

Each subdirectory is partitioned into scenes by scene id number like in the `images/` directory (see that section for details).

We provide a tutorial on how to merge the descriptors into the main database here. TODO

Scripts to query the database  TODO

## `images/` Directory
This directory houses images and image metadata for each scene grouped by subcategory.

 The `images/` directory is partitioned into scenes by their scene id number, which can be found in `scenes.csv`. Each scene has its own image tar file and is located in a subdirectory. The scene id is zero-padded to six digits; the first three digits and last three digits of this zero-padded id define the subdirectory that the scene resides at. For instance, if the `Eiffel_Tower` scene has an id of `12345` (five digits), then it will be zero-padded to `012345` (six digits), and the corresponding scene directory is at `images/012/456/`.
 
Within a scene directory, image metadata will be at `metadata.json`. This json contains a key for each image name, and contains information of various data extracted from Wikimedia Commons, including EXIF data and licensing information.

An `images.tar.gz` contains images categorized into subcategories. Specifically, it extracts into the scene directory with the following format:
- `commons/`
    - `subcategory_name/`
      - `image1.jpg`
      - `image2.jpg`
      - ...
  - ...

## `metadata/` Directory
This directory houses general metadata for the dataset.

- `scenes.csv` - summary of all scenes in the dataset
- `image_catalog.csv` - summary of all images in the dataset
- `colmap_db_catalog.csv` - summary of all COLMAP databases
- `recon_catalog.csv` - summary of all COLMAP sparse reconstructions

## `reconstruct/` Directory
The `reconstruct/` Directory is partitioned by scene id number like in the `images/` directory (see that section for details). A catalog of scenes can be identified in `metadata/recon_catalog.csv`.
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
