# FLOSSMonitor Server

## Setting up the server
1. Download the ZIP file with the server source code. You can either download the entire repository, or download a ZIP that only contains the server source from the releases tab.
2. Make sure you have NodeJS and NPM installed. You can download NodeJS [here](https://nodejs.org/).
3. Run the command `npm install` to install the required dependencies.
4. Once the dependencies are installed, run `npm start` to start the server.

## Using the GUI Command Center
to-do

## API Endpoints
All `POST` endpoints take in JSON data. All endpoints, `GET` or `POST`, output in JSON unless specified.

### `POST /api/v1/student/authenticate`
Gets an access token for a student.

Request body:
```json
{
  "email_address": "student@your.school.edu",
  "student_key": "<insert student key here>"
}
```

Response body:
```json
{
  "success": true,
  "access_token": "b88da0f3a7c7edd69d90e7ab2e7fb3"
}
```

> Warning: a student key is like a password - it prevents others from impersonating your students - keep it safe!

### `POST /api/v1/admin/authenticate`
Gets an access token for a system administrator.

Request body:
```json
{
  "email_address": "the.admin@urschoolteachers.com",
  "admin_key": "<insert admin key here>"
}
```

Response body:
```json
{
  "success": true,
  "access_token": "b88da0f3a7c7edd69d90e7ab2e7fb3"
}
```

> Warning: an admin key is more important to protect than a student key - Staff Keys give access to teacher and even administrator accounts!

### `POST /api/v1/admin/add-student`
Adds a student to the system - meant for automation.

Request body:
```json
{
   "access_token": "b88da0f3a7c7edd69d90e7ab2e7fb3",
   "email_address": "the.admin@urschoolteachers.com",
   "student": {
     "email_address": "student2@your.school.edu",
     "first_name": "First",
     "middle_name": null,
     "last_name": "Last",
     "policies": {
       "default": [ "your-policy-id-or-uuid-here" ],
       "atSchool": [ "your-at-school-policy-id" ]
     },
   }
}
```

The `atSchool` policies allows you to choose IP address that mean the student is "at school" and have a different set of restrictions.

### `POST /api/v1/student/filter`
Filters a web URL

Request body:
```json
{
  "access_token": "b88da0f3a7c7edd69d90e7ab2e7fb3",
  "email_address": "student@your.school.edu",
  "website_url": "https://websitethatthestudentwent.to/folder/page-student-was-looking-at.html?scrollToSection=1",
  "type": "visit" /* can be 'visit', 'image', 'media', 'visit-iframe', 'xhr-fetch', 'script', and 'style'
}
```

Response body:
```json
{
  "domain": "websitethatthestudentwent.to",
  "protocol": "https",
  "path": "/folder/page-student-was-looking-at.html",
  "filter": {
    "is_blocked": true,
    "categories": [1,2,3],
    "categories_string": "A category, another category, and yet another category",
    "resource_blocked": "page" /* can be either 'page', 'entire_url', 'domain', 'related_domains', 'search_query', or 'media_content' */
  },
  "content_type": "website", /* can be 'website', 'social_media', 'forum_board', or 'search_engine' */,
  "search_engine": null, /* either 'google', 'bing', 'ecosia', 'duckduckgo', 'frogfind', 'querex', 'startpage', or 'yahoo' - otherwise null */
  "force_safe_search": false,
  "search_tab": null, /* either 'web', 'images', 'media', or 'maps' */
  "force_cc3_images": false,
  "remove_tiktok_from_results": false,
  "remove_yt_from_results": false,
  "remove_vimeo_from_results": false,
  "remove_fb_from_results": false
}
```

### `POST /api/v1/student/config`

Request body:
```json
{
  "email_address": " ... ",
  "access_token": " ... ",
}
```

Response body:
```json
{
  "default_policies": ["a_policy_that_exists_for_no_specific_reason", "another_policy_that_also_just_randomly_exists"],
  "school_policies": ["a_policy_that_only_works_at_school"]
  "live_monitoring": {
    "enabled": true,
    "stream": "current_tab", /* Can be 'current_tab', or in the future, 'all_tabs'
    "stream_protected_tabs": false, /* Protected tabs are chrome://, about:, chrome-untrusted://, edge://, brave://, opera://, and border:// URLs - ones that extensions, addons, and Blugins can't usually access (in Chrome, Brave, Edge, and Opera, you can access these tabs by enabling the `allow-chrome-extensions-on-chrome-urls` flag) */
    "resolution": "sd", /* can be 'sd', 'hd', or 'hd4k'/'hdmax'
  },
  "name": {
    "first": "First",
    "middle": null,
    "last": "Last",
    "full": "First Last"
  },
  "filtering": {
    "enabled": true,
    "domains_blocked_by_gaurdians": [ "adomainyourmomdoesntwantyoutovis.it", "thiswebsitewasblockedbyyour.dad", "ormaybeyouhaveagaurdianinste.ad" ]
  },
  "windows_96_restrictions": {
    "enabled": false,
    "allow_p3_network": true,
    "allow_remote_console": true,
    "allow_p3fs_hosting": true,
    "allow_p3fs_mounting": true,
    "force_remote_console": false, /* require_password must be true */
    "forced_p3_secret": null, /* can be any valid P3 secret */
    "disable_internete_browser": false, /* warning: only works on V3, unless you ported InternetE to V2... */
    "block_w96_socials": false, /* ShareBoard and MsgRoom won't work if enabled */
    "require_password": true,
    "allow_disk_encryption": true,
    "mounted_disks": {
      "rofs": [],
      "writable": [
        {
          "label": "QDisk",
          "prefix": "Q:",
          "url": "https://your.school.edu/qdisk/",
          "format": "qdisk.mirrazz.me", /* QDisk Remote Disk Format, by themirrazz (coming soon) - not to be confused with the disk snapshot tool! */
          "username": "student",
          "password": "hhd9jkODSMc SAmz8i!I# S:# cdiW# big black slushie wolf card-reader stock-photo learning education imprint decay KDFOmx9#Em LWSD!!e"
        },
        {
          "label": "Local IDB",
          "prefix": "I:",
          "url": "idb://windows96.net/",
          "format": "idbfs.sys36.net", /* IndexedDB FileSystem (locally stored) */
          "username": "idb",
          "password": null, /* required if Disk Encryption was used on it */
        }
        }
      ]
    }
  }
}
```
