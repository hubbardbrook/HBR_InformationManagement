---
# HBR Data Package Cookbook: a step-by-step guide to building HBR data packages for EDI
### Overview
The purpose of this document is to capture details of the data package development workflow that is currently in use at Hubbard Brook. HBR data is published
in the [Environmental Data Initiative Repository (EDI)](https://edirepository.org). With the availability 
of EDI's ezEML data package builder application (adopted by HBR in 2024), this once long and complicated process has been greatly simplified.
### Database and File access
The working directory for package development is on the HBR-IM desktop (system name = gromit; linux OS; HBR-IM root directory = gromit:/d1/proj/hbrook/HBRIM/data).  
Assets for each data package are in folders named hbr[pkgid] in the ezEML folder. 
### Step-by-Step Data Package Workflow
In 2024, HBR fully adopted the EDI ezEML workflow for data package development. All data packages are developed under the EDI HBR user account. 
The division of effort varies with the nature of the data package. Graduate students are encouraged to collaborate with the IM online within the ezEML environment. 
This serves as a way to directly input metadata without first populating a template, reduces IM time on some data packages, and is an important skill-builder for HBR graduate students. 

EzEML provides the capability to store often-used metadata components in a template. For HBR, there is one master template and additional templates for 
HB projects that are frequent publishers (MELNHE, CRCH). Since the master template is very extensive, it is not cloned as a starting point for a new 
dataset (which might be a common template use), but instead accessed through the import [creator,geographic,keywords,project,funding] buttons where just selected 
items are brought into the current dataset.

The HBR Data Inventory table is hosted on the HubbardBrook sharepoint site (HBR-IM administrator at UNH). This table contains additional information that we 
use on our local data catalog to enhance user experience (flagging of significant core datasets, more robust LTER Core Research Area assignments that may be missing in 
older metadata, and a code to categorize datasets and to sort them in the initial catalog view). Data packages are entered in this table as soon as they are identified 
(in some cases with very long lead times). Upon becoming aware of a dataset, a package id is assigned and the entry initiated with status=anticipated.  As soon as data and/or 
metadata are in-hand, the status is updated to 'draft'.  The table includes packageID, abbreviated title, contact, notes as needed, flagging as long-term core dataset, and EDI submission status. 

The steps are as follows: 

    - Components for data packages are provided to the IM through a sharepoint dropoff.
    - ezEML collaboration is established if desired for the dataset.
    - A data package is initiated in ezEML with the naming convention of hbr[pkgid]-[shortname].
    * The HBR Master Template (stored in EDI) is used to import people, geographic areas, 
    keywords, projects, and funding.
    * Data tables are loaded from csv.
    * Data table attributes are documented either directly on the online forms or through the 
    ezEML table entity templating feature (a great timesaver for complex datatables).
    * ezEML data package is downloaded to IM's computer 
    * The R script to add QUDT unit annotations is run
    * The annotated eml file is uploaded to portal-s.lternet.edu and the URL is shared with creator for review
    * Subsequent edits are made with creator feedback
    * Data package is approved by the creator
    * Data package is uploaded to the live repository

### Non-tabular datasets (images, audio, very large datasets, etc)
    
### Notes on revising older datasets.

When earlier data packages are revised, the starting point is an ezEML fetch of the published data package. 
Assets for earlier data packages developed can be found in either the 'EMLassemblyline' or 'legacy' folders, although those files should rarely 
be necessary once a data package is published in EDI.  

An EDI fetched dataset may have been developed with a non-ezEML workflow. If that is the case, items requiring 
attention will be:

    * Creators – delete and import from the template to be sure to get ORCIDs and institution RORs for each person. 
    Use the 'sort' function on people import to find them easier in the long list.
    * Project – EMLAL datasets may have funding in 'related funding' or a text string in project abstract. Delete these. 
    Import all funding from the template as primary or related. The template will have enhanced information to include grant url, 
    funding agency ROR, etc.
    * Intellectual rights will be correct for all older datasets, but it is best to reset that in ezEML to CC-BY selection.
    * Increment the packageId revision number.
    * Use re-upload datatable if revision includes new or modified data. This all goes well if the table is identical. 
    If there are new columns, upload as a new table and clone metadata from the original, then define any new columns.
    If the dataset was prepared in EMLAL, clear min/max bounds. 
