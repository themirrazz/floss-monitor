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
     "student_key": "vXZUjx93k,PWdLSExPEFcGZjm @? di!nx"
   }
}
```

The `atSchool` policies allows you to choose IP address that mean the student is "at school" and have a different set of restrictions.

### `POST /api/v1/student/filter`
Filters and logs the URL of a visited website.
> Fun fact: you can use this same method and request body on `/api/v1/teacher/filter` for use with teachers.
> You can also get whether if a site is blocked based on a certain policy using `/api/v1/policy/filter`, keep reading to know more.

Request body:
```json
{
  "access_token": "b88da0f3a7c7edd69d90e7ab2e7fb3",
  "email_address": "student@your.school.edu",
  "website_url": "https://websitethatthestudentwent.to/folder/page-student-was-looking-at.html?scrollToSection=1",
  "type": "visit", /* can be 'visit', 'fetch', 'media', or 'script' */
  "log_history": true /* should be false if the resource wasn't the top-level page */
}
```

Response body:
```json
{
  "blocked": true,
  "categories": [0, 1, 2],
  "policies": [ "your-at-school-policy-id" ],
  "blocked_entry": "related_domains" /* can be 'related_domains', 'domain', 'resource', or 'search_query'
}
```

### `POST /api/v1/policy/filter`
Gets whether a URL should be blocked based on a specific policy.

Request body:
```json
{
  "policies": [ 'your-at-school-policy-id', 'your-policy-id-or-uuid-here', 'your-staff-policy-id' ],
  "website_url": "https://awebsiteto.test/path/to/exampleresource.html",
  "type": "visit"
}
```

Response body:
```json
{
  "blocked": true,
  "categories": [3, 4],
  "policies": [ 'your-at-school-policy-id', 'your-staff-policy-id' ],
  "blocked_entry": "resource"
}
```

### `POST /api/v1/student/rt*`/`POST /api/v1/teacher/rt*`/`POST /api/v1/admin/rt*`
Realtime WebSocket/WebRTC real-time API.

#### `POST /api/v1/student/rtfeed`
Get real-time PNG feed of the website they're viewing - and using the appropriate TabCapture instead of Html2Canvas allows it to perfectly capture every single element of the tab as-is - that includes iframes, too.

Request body:
```json
{
  "email_address": "student@your.school.edu",
  "access_token": "b88da0f3a7c7edd69d90e7ab2e7fb3",
  "feed": [ "..." ], /* this really contains an array, converted from a Uint8Array containing image data */
  "feed_type": "png", /* either 'png' or 'jpeg' - PNG images are easier for the server */
  "tab_title": "A Tab That I'm Looking At",
  "tab_domain": "atabthatimlooking.at"
}
```

#### `POST /api/v1/student/rtmessage`
Socket.IO channel that recieves messages from a teacher.

Send on connected (`hello`):
```json
{
  "email_address": "student@your.school.edu",
  "access_token": "b88da0f3a7c7edd69d90e7ab2e7fb3"
}
```

Recieve when message (`message`):
```json
{
  "teacher": "Mrs. Teachsalot",
  "type": "how_are_you", /* Can be 'alert', 'how_are_you', 'yes_no', 'selection', or 'reply'
  "message": "How are you doing on your math assignment?",
  "channel_id": "a888b7d3c9a9f8eb98af761f9aff60b7c8d73fe",
  "message_id": "c99ad0ef90ab8d83fe",
  "options": [ "..." ] /* only works if type is `selection`
}
```

Reply to message (`reply`):
```json
{
  "channel_id": "a888b7d3c9a9f8eb98af761f9aff60b7c8d73fe",
  "message_id": "c99ad0ef90ab8d83fe"
  "value": 0
}
```

##### Types of messages
* `alert` - Only an informational dialog, doesn't give the ability to reply.
* `how_are_you` - Lets you choose from three buttons: Good (0), OK (1), and Not Good (2)
* `yes_no` - Lets you choose either "Yes" or "No" - reply is sent in a boolean value
* `selection` - Shows a drop-down menu populated with the options from `options`, replies with index of option
* `reply` - Common for easier responses, allows the student to type in a text box to reply

#### `POST /api/v1/teacher/rtmessage`
Socket.IO channel that send messages to a student.

Send on connected (`hello`):
```json
{
  "email_address": "mrsteachsalot@urschoolteachers.com",
  "access_token": "b88da0f3a7c7edd69d90e7ab2e7fb3"
}
```

On recieves reply (`replied`):
```json
{
  "message_id": "c99ad0ef90ab8d83fe",
  "channel_id": "a888b7d3c9a9f8eb98af761f9aff60b7c8d73fe",
  "reply_data": 0
}
```

Send message (`send`):
```json
{
  "channel_id": "a888b7d3c9a9f8eb98af761f9aff60b7c8d73fe",
  "message": "How are you doing on your math assignment?",
  "type": "how_are_you"
}
```
