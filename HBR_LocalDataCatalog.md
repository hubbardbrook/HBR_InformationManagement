# Hubbard Brook Local Data Catalog A step-by-step guide

Updated 10/18/2024

mary.martin@unh.edu

## Overview

The purpose of this document is to capture details of the workflow to
build a data catalog table that is used on the [Hubbard Brook Data Catalog](https://hubbardbrook.org/data-catalog) displayed
on our website.
This workflow has been reconfigured from earlier versions to now read
only from publicly available sources - EDI and a public sharepoint file
with enhanced data package details. The latter improve the user
experience for data searchers by categorizing HBR data and adding robust
LTER core area tags.

## Database and File access

Access to dataset details is provided by the LTER PASTA API via the
EDIutils R package. The enhanced table resides on the HubbardBrook
sharepoint site (HBR IM admin, UNH). The sharepoint data inventory file is
maintained to track status of each dataset and to provide additional information that
is not included in the formal metadata or may be lacking in very old datasets but is useful
in our data catalog (LTER Core Research Area, HBR significant data status)

## Step-by-Step Catalog Workflow

-   Run the code in Appendix B (dataCat.R)
-   log in to the wordpress site
-   open the data catalog table
-   upload wptablefeed.csv to replace existing version

## APPENDIX A -- Code to build the local HBR data catalog:

## FetchSarepointDataInventory.R

```
{
\##########################################

\# FetchSharepointDataInventory.R

\##########################################

library(httr)

library(readxl)

\# Function to read Excel file from SharePoint URL

read_sharepoint_excel \<- function(url) {

\# Download the file

response \<- GET(url, config(ssl_verifypeer = FALSE))

\# Check if the download was successful

if (status_code(response) != 200) {

stop(\"Failed to download the file. Status code: \",
status_code(response))

}

\# Create a temporary file

temp_file \<- tempfile(fileext = \".xlsx\")

\# Write the content to the temporary file

writeBin(content(response, \"raw\"), temp_file)

\# Read the Excel file

df \<- read_excel(temp_file)

\# Remove the temporary file

unlink(temp_file)

return(df)

}

\# URL of your SharePoint Excel file

url \<-
\"https://universitysystemnh.sharepoint.com/:x:/t/HubbardBrook/EXFAJdG37VxOsrGI5jMhxmcBc1vTpUnZw1WPtiP3Q5Mc4A?download=1\"

\# Read the Excel file

df \<- data.frame(read_sharepoint_excel(url))

\# subset to just cataloged datasets, save as df \'ps\' (for package
state)

ps=df\[which(df\$status==\"cataloged\"),\]

}
```
## dataCat.R 

```{
\#############################################

\# dataCat.R

\#

\# mary.martin@unh.edu 20241016

\# Purpose: build a datatable for the HB website local data catalog

\# This is a revised script to generate the data catalog file on the
wordpress site

\# this revision now runs on all publicly available data - no pw
protected databases for local info

\# Inputs are - sharepoint xlsx file that tracks data status locally and
provides additional info to enhance user

\# experience in searching for data

\# Requirements: The following script is sourced and should be located
in the same folder as this main script: FetchSharepointDataInventory.R

\# Usage: Run this script, then move the wptablefeed.csv file to the
website and updated the table to refresh to this new version

\# Note: once this runs a few times and I gain a little confidence, I
will consider adding a command to ftp directly to wordpress to refresh
the site

\#############################################

library(EDIutils)

library(tidyverse)

library(httr)

library(\"stringr\")

library(xml2)

\# set the working directory to the location of the script

setwd(dirname(rstudioapi::getActiveDocumentContext()\$path))

\# source the script that gets local package info from sharepoint
spreadsheet

\# returns dataframe ps(package state)

source(\"FetchSharepointDataInventory.R\")

\# Fetch the basic eml info directly from EDI for each package

\# consider getting abstract and making that display as a hoverover on
the website table

res\<-search_data_packages(

query =
\"q=\*&fq=-scope:ecotrends&fq=scope:knb-lter-hbr&fq=-scope:lter-landsat\*&fl=id,packageid,doi,title,pubdate,begindate,enddate\")

names(res)=c(\"id\",\"PackageId\",\"doi\",\"Title\",\"pubdate\",\"begindate\",\"enddate\")

res\$Title=gsub(\"\[\\r\\n\]\", \" \", res\$Title)

res\$Title=gsub(\"\\\\s+\", \" \", res\$Title)

\# extract begin and end YEAR

res\$startYear=as.POSIXlt(as.Date(res\$begindate))\$year + 1900

res\$endYear=as.POSIXlt(as.Date(res\$enddate))\$year + 1900

\# if dataset has start/end dates, create column that shows them with
dash separator

res\$yearrange=\"NA\"

res\$yearrange = paste0(res\$startYear,\" - \",res\$endYear)

index= grep(\"NA\",res\$yearrange)

res\$yearrange\[index\] = \" \"

#create the edilink

res\$edilink=paste0(\"https://portal.edirepository.org/nis/mapbrowse?packageid=\",res\$PackageId)

\# Fetch the pesky keywords and authors as xml so you can insert a
separator

k\<-search_data_packages(

query =
\"q=\*&fq=-scope:ecotrends&fq=scope:knb-lter-hbr&fq=-scope:lter-landsat\*&fl=id,packageid,keyword,author\",

as = \"xml\"

)

kw \<- data.frame(

PackageId=character(),

Originators=character(),

Keywords=character(),

stringsAsFactors=FALSE)

count=1

for (doc in xml_find_all(k, \".//document\")) {

print(doc)

\# Get the keywords from the current doc

keyword_elements \<- xml_find_all(doc, \".//keyword\")

keyword_strings \<- xml_text(keyword_elements)

\# This doesn\'t get the authors where HBWaTER and USFS are institution
authors.

\# see solution below to get those from cite.edirepository.org

kw\[count,3\] \<- paste(keyword_strings, collapse = \";\")

\# Get the authors from the current doc

author_elements \<- xml_find_all(doc, \".//author\")

author_strings \<- xml_text(author_elements)

kw\[count,2\] \<- paste(author_strings, collapse = \"; \")

\# get the packageid

pid \<- xml_find_all(doc, \".//packageid\")

pidstring \<- xml_text(pid)

kw\[count,1\] \<- pidstring

count=count+1

}

\# tidy up keywords where some strings have newlines or consec white
spaces

kw\$Keywords=gsub(\"\[\\r\\n\]\", \" \", kw\$Keywords)

kw\$Keywords=gsub(\"\\\\s+\", \" \", kw\$Keywords)

\# merge the tidier keywords with the main table

resj=merge(res,kw, by = \"PackageId\")

\# get the dataset citations so that you have a nicer listing of authors

\# you could probably do that where I do the keywords now from xml, but
it doesn\'t

\# populate the records where author is an institution (USFS and
HBWaTER)

for(i in 1:dim(resj)\[1\]){

print(i)

CMD=paste0(\'GET
(\"https://cite.edirepository.org/cite/\',resj\[i,1\],\'\")\')

\# sleep can be removed if you are whitelisted to make rapid EDI queries

Sys.sleep(0.5)

print(CMD)

c=content(eval(parse( text = CMD )))

print(c\$authors)

resj\[i,\"Originators\"\]=c\$authors

}

\# FetchSharepointDataInventory returns ps dataframe (aka package state
in the originalmetabase database)

\# add data category based on sort order codes in ps (local package
state table)

\# apply category name to sort order values

ps\$category=0

index=which(substr(ps\$pub_notes,1,1)==1)

ps\$category\[index\]=\"Hydrometeorology\"

index=which(substr(ps\$pub_notes,1,1)==2)

ps\$category\[index\]=\"Water Chemistry\"

index=which(substr(ps\$pub_notes,1,1)==3)

ps\$category\[index\]=\"Soils\"

index=which(substr(ps\$pub_notes,1,1)==4)

ps\$category\[index\]=\"Vegetation\"

index=which(substr(ps\$pub_notes,1,1)==5)

ps\$category\[index\]=\"Heterotrophs\"

index=which(substr(ps\$pub_notes,1,1)==8)

ps\$category\[index\]=\"Documentation\"

index=which(substr(ps\$pub_notes,1,1)==9)

ps\$category\[index\]=\"Spatial Datasets\"

\# subset out the columns that are to be used in the datatable

pscat=ps\[,c(\"dataset_archive_id\",\"category\",\"ltercore\",\"pub_notes\")\]

\# merge the EDI query dataframe with ps

m=merge(resj,pscat, by.x=\"id\",by.y=\"dataset_archive_id\")

\##### Write out wordpress data table \#################

\# pull out the columns needed for website table

wptablefeed=m\[,c(\"PackageId\",\"Title\",\"Originators\",\"yearrange\",\"ltercore\",\"edilink\",\"category\",\"pub_notes\",\"Keywords\")\]

\# sort packages based on pub_notes

wptablefeed.order=wptablefeed\[order(wptablefeed\$pub_notes),\]

\# write out the table

write.table(wptablefeed.order,\"wptablefeed.csv\",row.names=FALSE,sep=\",\",na=\"
\")
}'''
