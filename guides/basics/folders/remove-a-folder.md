---
rank: 3

related_endpoints:
  - post_folders_id
related_guides:
  - folders/find-a-folder-id
  - folders/get-folder-items
requires:
  - sdk/install
  - sdk/initialize
---

# Remove a folder

To create a folder in Box you will need to provide our API with a `name` for the new folder, as well as the `id` of the `parent` folder that you would like to create the new folder within.

<Samples id='post_folders_id'></Samples>

<Message type='notice'>
  # Name restrictions
  
  There are some restrictions to the file name. Names containing non-printable ASCII characters, forward
  and backward slashes (`/`, `\`), as well as names with trailing spaces are prohibited.

  Additionally, the special names `.` and `..` are reserved names and therefore also prohibited.
</Message>