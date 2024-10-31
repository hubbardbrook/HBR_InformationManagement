
# HBR Data Inventory

A spreadsheet listing a complete inventory of HBR data, including datasets anticipated and in draft format, is maintained
on the UNH HubbardBrook sharepoint site. Access to this sheet is shared by request. This is used to build the local data catalog by
merging with content from the LTER PASTA API (joined on dataset_archive_id). It is also used by IM as a local record of what is published, as well
as those datasets anticipated, in draft format, or staged for review. the emlWorkflow is helpful in identifying datasets that could be
updated to include eml elements not available at time of publication (improved funding metadata, general annotations, qudt unit annotations).
The checkbox columns for HBR_v were used to modify our data table listing (proposal supplemental document), by highlighting data used in papers during that funding cycle and dta used in the top10 publications that we highlighted in the proposal.

- DataSetID: just the pkg number
- dataset_archive_id: packageId (knb-lter-hbr.XXX)
- rev: revision number
- nickname: a short name for the dataset	
- status: dropdown choice of anticipated, draft, staged, cataloged, deprecated, embargoed
- emlWorkflow: legacy, EMLAL, ezEML, mmb (minimetabase)
- notes	
- pub_notes	
- dbupdatetime:carryover from mmb	
- update_date_catalog: date of last published revision	
- who2bug: contacts	
- in pasta: binary 0/1 to indicate published	
- signature: do we consider this a core HB longterm dataset
- ltercore: core research areas (DP(disturbance process),IM(inorganic matter),OM(organic matter),PP(primary production),PS(population study)	
- hbr_vcited: data cited in HBR-V publications	
- hbr_vtop10: data cited in HBR-V top 10 publications


