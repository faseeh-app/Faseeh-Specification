---
Type: Folder
---
# `library/{uuid}` Directory Overview

> [!Note] Description
> Represents the dedicated container folder for a **single imported media item**. The folder name `{media_object_uuid}` is the unique ID corresponding to the item's record in the `MediaObject` table in the database. It aggregates all files related to this specific item.

#### Key Notes

> [!Summary]+ Content:
> - **[[📄 source.ext]] (Optional):** The primary source file, if copied/managed internally.
> - **[[📄 Document.json]] (Optional):** The structured [[Content Adapter#^205d65|FaseehContentDocument]]
> - **[[📂assets]] (Optional):** Subfolder for binary assets extracted during adaptation.
> - **[[📂associated]] (Optional):** Subfolder for supplementary files like subtitles. 
> ---
> - **Managed By:** [[Storage Service]] (Main Process). Created during the import process after the `MediaObject` database record is created and its ID is generated. Deleted when the corresponding `MediaObject` is deleted.
> ---
> - **Relationship:** Directly corresponds to one `MediaObject` record in the database via its UUID name.

> [!Tip] Locality:
> This folder provides excellent data locality – everything needed to represent and display a single imported item (beyond the core metadata in the DB) is located within this single directory.