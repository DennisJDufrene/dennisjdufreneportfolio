!!! info
    
    This is a sample API reference document targeting developers.


# Get User Details

The **Get User** transaction is used to retrieve user details from the system.

## Request

Format the request as follows:

    GET https://fake.system.com/users/user={user_email}

User email is required and must be formatted as a standard email address: `{string}@{string}.{string}`. For example:

    dennis.dufrene@yahoo.com

### Request Sample

    GET https://fake.system.com/users/user=dennis.dufrene@yahoo.com

## Response

The system returns the following fields:

<table>
<caption>Supported Fields</caption>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Field</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>user_email</p></td>
<td><p>The user’s email address.</p></td>
</tr>
<tr class="even">
<td><p>user_name</p></td>
<td><p>The user’s first and last name.</p></td>
</tr>
<tr class="odd">
<td><p>user_phone</p></td>
<td><p>The user’s contact phone number (cell or home).</p></td>
</tr>
<tr class="even">
<td><p>user_active</p></td>
<td><p>Yes or no field indicating if the user has been active in the last 30 days.</p></td>
</tr>
</tbody>
</table>

### Response Sample

    {
       "user_email" : "dennis.dufrene@yahoo.com",
       "user_name" : "Dennis Dufrene",
       "user_phone" : "5555555555",
       "user_active" : "Yes",
       },

## Error Handling

The system returns the following errors:

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Error</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code>200: User Not Found</code></p></td>
<td><p>The email address included in the request is not available in the system.</p></td>
</tr>
<tr class="even">
<td><p><code>231: Bad Request</code></p></td>
<td><p>The email address is blank, null, or incorrectly formatted.</p></td>
</tr>
<tr class="odd">
<td><p><code>260: Number of Requests Exceeded</code></p></td>
<td><p>API calls exceeded the limit of 15 in 5 second(s).</p></td>
</tr>
<tr class="even">
<td><p><code>299: Server Error</code></p></td>
<td><p>Internal server error resulting from failed API call. Review stacktrace for more details.</p></td>
</tr>
</tbody>
</table>
