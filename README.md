# MegaScenes Dataset v1.0

[**Paper**](https://megascenes.github.io/MegaScenes_paper_v1.pdf) | [**Arxiv**](https://arxiv.org/abs/2406.11819) | [**Project Page**](https://megascenes.github.io) <br>

The MegaScenes Dataset is an extensive collection of around 430K scenes and 9M images and epipolar geometries, featuring over 100K structure-from-motion reconstructions from 2M of these images. The images of these scenes are captured under varying conditions, including different times of day, various weather and illumination, and from different devices with distinct camera intrinsics.

To view reconstructions in the browser, see our [**Web Viewer**](https://megascenes.github.io/web-viewer/)!

# Data Access
The MegaScenes Dataset is hosted on [Amazon S3](https://aws.amazon.com/s3/) thanks to the [AWS Open Data Sponsorship Program](https://aws.amazon.com/opendata/).

Specifically, MegaScenes uses the AWS S3 bucket URL `s3://megascenes/` in the `US-West-2` AWS Region.

## Dataset Access via Command-line Interface (recommended)

Users can access the dataset using [s5cmd](https://github.com/peak/s5cmd) (recommended) or [AWS CLI](https://aws.amazon.com/cli/). These programs are locally installed command-line interfaces that can access publicly hosted datasets on AWS. For the rest of this section, we will share some s5cmd commands.

### Downloading MegaScenes
Copy a file or a directory locally: `s5cmd --no-sign-request cp <source_bucket_url> <local_dest>`

Alternatively, `sync` can be used instead of `cp`. `sync` additionally checks for differences between AWS and the locally downloaded dataset.

#### Example 1: Downloading a specific directory locally
If the source URL is a directory, then it must have a wildcard (`*`).

This command recursively downloads the contents of the `images` folder from AWS into the local folder `MegaScenes/images/`:
```
s5cmd --no-sign-request cp s3://megascenes/images/* ./MegaScenes/images/
```

#### Example 2: Downloading a single file locally
This command downloads a specific `database.db` file from AWS into its respective local folder:
```
s5cmd --no-sign-request cp s3://megascenes/databases/main/000/000/database.db ./MegaScenes/databases/main/000/000/database.db
```

### Listing MegaScenes directory contents
List directory contents: `s5cmd --no-sign-request ls <bucket_url>`

#### Example
Input
```
s5cmd --no-sign-request ls s3://megascenes/databases/
```

Output
```
                                  DIR  descriptors/
                                  DIR  main/
                                  DIR  databases/
```

### Other Notes
The `--no-sign-request` flag is for the user to access the AWS bucket without the need to create and supply AWS credentials.

## Dataset Access via HTTP
Singular files can be downloaded over HTTP (via `wget` or `curl`) using the base URL `https://megascenes.s3.us-west-2.amazonaws.com/`.

For instance, [this URL](https://megascenes.s3.us-west-2.amazonaws.com/metadata/subcat/000/007/subcats.json) is a direct download for the subcategory information for scene-ID `7`.


# Dataset Layout
The bucket's directory tree is as follows:
- `s3://megascenes/` or `https://megascenes.s3.us-west-2.amazonaws.com/`
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

A scene is represented by its zero-padded six-digit scene-ID number as described in [Scene Folders](#scene-folders) in applicable subdirectories. A directory that links scene name to scene-ID can be found at: `s3://megascenes/metadata/categories.json`. For details on subfolder contents, see the respective sections below.
 

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

### 

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

The `metadata/` directory has the following contents: 

- `subcat/` (386 MB), which is a directory that contains JSON files of subcategory information for scenes with at least one subcategory
- `wikidata/` (4.5 GB), which is a directory contains JSON files for all Wikidata entries related to a scene or their heirarchical classes
- `categories.json` (19.2 MB), which is dictionary that maps a Wikimedia Commons category name to a scene-ID.

### Subcategory Information
Subcategory information resides in the `metadata/subcat/` directory. This directory is organized by scene-ID number as described in [Scene Folders](#scene-folders).

A scene is present in `metadata/subcat/` only if it has at least one category besides the main category. Such a scene will have a `subcats.json` to represent the subcategory data.

A `subcats.json` file is a dictionary that contains the following fields:
- `main_category`: a string of the name of the Wikimedia Commons top-level category.
- `graph`: a dictionary mapping a Wikimedia Commons category to a list of its direct subcategories. A category will be a key in `graph` if it has been visited. An empty list means that the category has no subcategories. 
- `frontier`: a list of subcategories present in `graph` that have not been expanded to have its own key in `graph`.

#### Example
The category [Arco degli Argentari](https://commons.wikimedia.org/wiki/Category:Arco_degli_Argentari) has a scene-ID of `7`. The subcategory information for this scene is at `s3://megascenes/metadata/subcat/000/007/subcats.json`, and has the following contents:
```
{
    "main_category": "Arco_degli_Argentari",
    "graph": {
        "Arco_degli_Argentari": [
            "Arco_degli_Argentari_in_art",
            "Historical_images_of_the_Arco_degli_Argentari"
        ],
        "Arco_degli_Argentari_in_art": [],
        "Historical_images_of_the_Arco_degli_Argentari": [
            "Arco_degli_Argentari_in_art"
        ]
    },
    "frontier": []
}
```
Here, the graph shows that the main category [Arco degli Argentari](https://commons.wikimedia.org/wiki/Category:Arco_degli_Argentari) has two subcategories: [Arco degli Argentari in art](https://commons.wikimedia.org/wiki/Category:Arco_degli_Argentari_in_art) and [Historical images of the Arco degli Argentari](https://commons.wikimedia.org/wiki/Category:Historical_images_of_the_Arco_degli_Argentari). The category [Arco degli Argentari in art](https://commons.wikimedia.org/wiki/Category:Arco_degli_Argentari_in_art) has no subcategories, hence the empty list. In contrast, the category [Historical images of the Arco degli Argentari](https://commons.wikimedia.org/wiki/Category:Historical_images_of_the_Arco_degli_Argentari) has the subcategory [Arco degli Argentari in art](https://commons.wikimedia.org/wiki/Category:Arco_degli_Argentari_in_art).

The frontier list is empty, meaning that this subcategory graph is expanded in its entirety. 


### Wikidata Entries
The `wikidata/` subcategory is organized by Wikidata Q-ID. The first three digits of the Q-ID define the three subfolders that the Wikidata JSON information can be found in. If the Q-ID has less than three digits, then its JSON resides in the `other/` folder. Unlike the scene IDs, this number is NOT zero-padded.


#### Examples

The JSON for a Wikidata item with Q-ID `Q1234` is located at `metadata/wikidata/1/2/3/Q1234.json`.

The JSON for a Wikidata item with Q-ID `Q12` is located at `metadata/wikidata/other/Q12.json`.

#### Resources
For JSON documentation, see this page on [Wikibase JSON](https://doc.wikimedia.org/Wikibase/master/php/docs_topics_json.html).

For additional tools to parse this JSON, see this Wikidata page on [Data access](https://www.wikidata.org/wiki/Wikidata:Data_access).

<!--
### Tutorials
We provide details on Wikidata TODO
-->

## `reconstruct/` Directory
This directory contains the COLMAP sparse point cloud reconstructions for each scene. The `reconstruct/` directory is organized by scenes, according to a scene-ID number as described in [Scene Folders](#scene-folders). Each reconstruction consists of an `images.bin`, `cameras.bin`, and `points3D.bin` as described [here](https://colmap.github.io/format.html). A scene may have zero or more reconstructions; the `reconstruct/` folder only contains scenes with one or more.

The `reconstruct/` folder is 429 GB.

### Example
Suppose a scene with ID `1234` has three reconstructions. In this scene's `sparses/` folder, there will be three folders numbered from `0` to `2`.

Specifically, the format is as follows:
- `reconstruct/`
    - `001/234/`
        - `sparses/`
            - `0/`
                - `images.bin`
                - `cameras.bin`
                - `points3D.bin`
            - `1/`
                - `images.bin`
                - `cameras.bin`
                - `points3D.bin`
            - `2/`
                - `images.bin`
                - `cameras.bin`
                - `points3D.bin`

## Scene Folders
The dataset uses a system of two subfolders to divide scenes; each scene has a scene-ID number. The first subfolder uses the first three digits of the 6-digit zero-padded scene ID. The second subfolder uses the last three digits. The data associated with the scene resides in the latter subfolder.

For example:
- If a scene has an ID of `533`, it is zero-padded to `000533`. This number translates to the directory `000/533/`.
- If a scene has an id of `422678`, it translates to the directory `422/678/`.

Each scene is based off of a category from Wikimedia Commons. For instance, the scene "Arc_de_Triomphe_de_l'Étoile" uses images from [Category:Arc de Triomphe de l'Étoile](https://commons.wikimedia.org/wiki/Category:Arc_de_Triomphe_de_l%27%C3%89toile) and its subcategories. MegaScens use underscores instead of spaces for scene names, but they are interchangable when used in Wikimedia Commons URLs.

The file `s3://megascenes/metadata/categories.json` [(HTTP Link)](https://megascenes.s3.us-west-2.amazonaws.com/metadata/categories.json) links the category name to the scene-ID.

# Contributions, Issues, and Suggestions
We are continually looking for ways to improve the dataset. If you find any incorrect reconstructions, please create an GitHub issue [here](https://github.com/MegaScenes/dataset/issues) or discussion post [here](https://github.com/MegaScenes/dataset/wiki).

# License
This dataset is licensed under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/). The photos in the `images/` folder have their own licenses.
