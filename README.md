# Islandora Mirador

## Archived
This repository is archived as of September 2020.

## Introduction

This module implements [Mirador](https://github.com/IIIF/mirador) open source [IIIF](http://iiif.io/) image viewer for Islandora Book and Large Image objects and provides functionality to generate IIIF Manifests and ingest them into our [IIIF API](https://github.com/utlib/utl_iiif_api) server.

This module is based off [Islandora Mirador Bookreader](https://github.com/utlib/islandora_mirador_bookreader) and [Islandora SharedCanvas Manifest Generator](https://github.com/utlib/islandora_sc_manifest). Initial development of both modules was supported by the The Andrew W. Mellon Foundation for the [French Renaissance Paleography website](https://paleography.library.utoronto.ca/).


## Prerequisite

* Drupal 7.x with Islandora 7.x
* Drupal module [Libraries API](https://www.drupal.org/project/libraries)
* Compiled Mirador javascript library
  * [Stable 2.x release version](https://github.com/ProjectMirador/mirador/tree/release-2.x)
You need to compile it following the README file. The Master branch contains many experimental features that are not stable, but fun for testing.
* UTL IIIF [Presentation API](https://iiif.library.utoronto.ca/presentation/v2/collections) server
    * Github https://github.com/utlib/utl_iiif_api
* IIIF [Image API](http://iiif.io/api/image/2.0/) server
  * [Loris](https://github.com/loris-imageserver/loris) works well.


## Install
NOTE: Uninstall *islandora_mirador_bookreader* and *sc_manifest_generator* modules (if installed).
1. Download compiled Mirador javascript library into Drupal's libraries directory, usually in `sites/all/libraries`. Verify the file permission is web servable.
2. Clone and enable this module.
3. Go to admin/islandora/solution_pack_config/book and choose **Mirador BookReader** under Book Viewers and Page Viewers.
4. Go to /admin/islandora/solution_pack_config/large_image and choose **Mirador BookReader** under Viewers


## Generate IIIF Manifest
In order for the viewer to automatically display the current page's book object, you can generate and ingest a IIIF Manifest for the object and tell the viewer to use this when loading the page.

This produces [IIIF Manifest](http://iiif.io/api/presentation/2.1) for the following Islandora Solution Packs:
 * [Islandora Book Solution Pack](https://github.com/Islandora/islandora_solution_pack_book) as a IIIF Manifest
 * [Islandora Large Image Solution Pack](https://wiki.duraspace.org/display/ISLANDORA715/Large+Image+Solution+Pack) as a IIIF Manifest
 * [Islandora Compound Solution Pack](https://wiki.duraspace.org/display/ISLANDORA715/Compound+Solution+Pack)  as a IIIF Manifest for the Compound object's first Large Image child

### Usage
There are two ways to generate IIIF Manifests: Via Drush or Via one of the above mentioned Solution Pack. Both ways would first delete the existing IIIF Manifest and then create the newly generated **IIIF Manifest** into the server.

### Drush
On the command line, run the following drush command:
```
drush iiifgen PID_OF_OBJECT
```
where PID_OF_OBJECT is the PID of the object to generate IIIF Manifest.
To generate the IIIF for a Collection and all of its children, run the drush command with the `recursive` flag:
```
drush iiifgen PID_OF_OBJECT --recursive
```
For testing and debugging purposes, run the drush command with the `test` and `show` flag:
```
drush iiifgen PID_OF_OBJECT --test --show
```
where `test` will prevent the datastream being ingested and `show` will output the generated JSON in the console.

To perform the operation in production environment, run the drush command with `production` flag:
```
drush iiifgen PID_OF_OBJECT --production --auth='theAuthenticationToken'
```
All possible flags are as follows:
```
--recursive => Whether to recursively generate IIIF Manifest for this object including its children: true|false. Defaults to false.
--test => Whether to run the command in Test mode: true|false. Defaults to false.
--show => Whether to print the resulting IIIF JSON object: true|false. Defaults to false.
--production => Whether to perform in production environment. Defaults to using development.
--auth=<token> => JWT authentication token. Used to ingest the generated JSON into IIIF server.
--delete => Whether to delete the given PID from IIIF server.
```
Example output when generating a Collection Manifest will look like:
```
drush iiifgen paleography:historicalmaps --test --recursive

<<<<<<<-- Running in TEST mode. This will not ingest to Fedora. -->>>>>>>>

Processing IIIF Collection for Object: paleography:historicalmaps
Collection has 6 children

Processing Sub Manifest: paleography:423 for Collection: paleography:historicalmaps
Processing Sub Manifest: paleography:387 for Collection: paleography:historicalmaps
..............

<<<<<<<-- Running in Recursive mode. -->>>>>>>>

Processing Manifest paleography:423
Processing Canvas (Page) for Object: paleography:423

Processing Manifest paleography:423
Processing Canvas (Page) for Object: paleography:423

..............

Successfully generated the IIIF JSON for: paleography:historicalmaps                                                   [status]
Successfully generated the IIIF JSON for: paleography:423                                                              [status]
Successfully generated the IIIF JSON for: paleography:423                                                              [status]
..............
```

### UI
Each Comaptible Solution Pack object has a new **IIIF Generation** option under **Manage** through the admin UI for each object. The option exists as a sub-menu item.

Clicking **Generate IIIF Manifest** will add/update a IIIF Manifest in the server.



## Customizations for the French Renaissance Paleography website

### Fixed thumbnail not showing up on first load
The thumbnails do not appear unless the view changes. In $.ThumbnailsView's Handelbars the src attribute does not receive the value when first loading.
To fix this, thumbUrl was placed into the src value.

### Load current manuscript first
When Mirador is initialized with an array of manifests, it can display any one of them automatically by setting the loadedManifest option in the windowObjects array. However, Mirador does not prioritize loading the loadedManifest, causing it to wait until the other manifests before it in the array have been loaded. If the current manifest is in low position in a large array, the wait could be long.
The fix was to manually rearrange the loadedManifest's position to be at top before initializing Mirador.

### Removing annotation overlay from all windows
In official release the annotation overlay "speech bubble" can be removed by setting annotationLayer to false in the windowObjects array. However, the setting could not be applied to subsequent windows. Instead, the overlay link is hidden from view via CSS.

## Current maintainers

* [University of Toronto Libraries:](http://its.library.utoronto.ca/)
	* [Kelli Babcock](http://its.library.utoronto.ca/staff/kelli-babcock)
	* [Bilal Khalid](https://github.com/bilalkhalid)
	* [Sunny Lee](https://github.com/sunnywd)
	* [Jana Rajakumar](https://github.com/jana-uoft)
	* [Monica Ung](https://github.com/monicau)

## Copyright and License

Copyright 2020 University of Toronto Libraries

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

