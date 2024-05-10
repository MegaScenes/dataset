# MegaScenes Dataset
The MegaScenes Dataset is an extensive collection of around 430K scenes and 9M images and epipolar geometries, featuring over 100K structure-from-motion reconstructions from 2M of these images. The images of these scenes are captured under varying conditions, including different times of day, various weather and illumination, and from different devices with distinct camera intrinsics.

# Data Format
MegaScenes contains the following entries:
- `scenes.csv` - summary of all scenes in the dataset
- `image_catalog.csv` - summary of all images in the dataset
- `colmap_db_catalog.csv` - summary of all COLMAP databases
- `recon_catalog.csv` - summary of all COLMAP sparse reconstructions
- `images/` - Directory housing per-scene imagery
- `reconstruct/` - Directory housing per-scene COLMAP databases and reconstructions

## `images/` Directory
 The `images/` directory is partitioned into scenes by their scene id number, which can be found in `scenes.csv`. Each scene has its own image tar file and is located in a subdirectory. The scene id is zero-padded to six digits; the first three digits of this zero-padded id define the subdirectory that the scene resides at. For instance, if the `Eiffel_Tower` scene has an id of `12345` (five digits), then it will be zero-padded to `012345` (six digits), and the corresponding scene directory is at `images/0/1/2/0123456/`.
 
Within a scene directory, image metadata will be at `metadata.json`. This json contains a key for each image name, and contains information of various data extracted from Wikimedia Commons, including EXIF data and licensing information.

An `images.tar.gz` contains images categorized into subcategories. Specifically, it extracts into the scene directory with the following format:
- `commons/`
    - `subcategory_name/`
      - `image1.jpg`
      - `image2.jpg`
      - ...
  - ...

## `reconstruct/` Directory
The `reconstruct/` Directory is partitioned by scene id number like in the `images/` directory (see that section for details). All scenes have a `database.tar.gz` file that contains the COLMAP database. A subset of scenes with a sparse reconstruction have a `sparses.tar.gz` file; these scenes can be identified in `recon_catalog.csv` in the base dataset directory.

The `database.tar.gz` file will extract into the scene directory as `dbs/database.db`.
The `reconstruction.tar.gz` file will extact into the scene directory as follows:
- `sparses/`
  - `0/`
    - `images.bin`
    - `cameras.bin`
    - `points3D.bin`
  - `1/`
    - ...
  - ...

COLMAP sparse reconstructions are represented by numbered subdirectories in `sparse/`.
The COLMAP databases contain information on SIFT keypoints and descriptors for scene images, as well as pairwise epipolar geometries.
