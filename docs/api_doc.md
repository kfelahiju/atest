Overview
=======
## Conventions

* The HTTP(or HTTPS) protocol will be used for communication between a client side and the server.
* Uses `UTF-8` character encoding.
* Request headers should include a `Content-Type` of `application/json; charset=utf-8`
* All the responses will be in `JSON` format.
* All datetime values are [`ISO-8601`](https://en.wikipedia.org/wiki/ISO_8601) format and `UTC` timezone.
* All responses from API will be `200 OK` even a server error is occurred. Client side should check `http_status_code` and `error code` to know exactly what happened.
## Authentication

All authenticated endpoints require an API access token, client side must provide the token in header of requests:
```
X-Access-Token: <X-User-Access-Token>
```

You can retrieve token by using the [Login](#endpoints-authentication-login) endpoint.

## Errors Handling

The DSRT API uses the following HTTP status codes for errors:

Status | Meaning
---------- | -------
400 | Bad Request
401 | Unauthorized
403 | Forbidden
404 | Not Found
405 | Method Not Allowed
406 | Not Acceptable
500 | Internal Server Error
503 | Service Unavailable

Codes in the `4xx` range indicate an error that failed given the information provided (e.g., parameters is not pass validation, access token expired, ...), and codes in the `5xx` range indicate an error from server.
However, not all errors map cleanly onto HTTP response codes, so there is some our own error codes to indicate more exacty what happened:

Code | HTTP status | Meaning
--------- | ------- |------
`BAD_PARAMETERS` | 400 | Some parameters are not passes validations. It usually cause from user inputs.
`BAD_REQUEST` | 400 | The request form is illegal or not in expected format from server. It usually a technical problem with client side.
`ACCESS_TOKEN_INVALID` | 401 | User access token is not valid or expired. Client side should re-authenticate to get new access token.
`UNAUTHORIZED_ERROR` | 401 | Server failed to find user with given credentials.
`ACCESS_DENINED` | 403 | The request is not allowed due to user do not have permission to access a resource or restricted to do something
`RESOURCE_NOT_FOUND` | 404 | The request asked for a resource which not exists or has been deleted from server.
`ENDPOINT_NOT_EXISTS` | 404 | The request path does not exists, or cannot handler by any endpoint. It might incase HTTP method is invalid.
`SERVER_ERROR` | 500 | An unhandled error from server occurred. AKA technical problem.
`SERVER_MAINTENANCE` | 503 | Server maintenance now in progress, it might for deployment.

## Pagination
All APIs has pagination accept following parameters:
Field | Type | Description
--------- | ------- | -----------
page | integer | Must be greater than `0`. Default is `1`.
per | integer | Must be greater than `0`. Default is `20`. Maximum is `100`.

JSON response has these information for pagination:
Field | Type | Description
--------- | ------- | -----------
page | integer | The current page number as passed from client side or default
per | integer | The page size as passed from client side or default
total | integer | The number of records matched query paramters

Endpoints
=======

## Authentication

### Login

* Path: `/api/v2/user/authentication/login`
* Method: `POST`
* Authenticate: `no`
* Parameters:

Field | Type | Required | Description
--------- | ------- | ------- | -----------
user[email] | string | yes |
user[password] | string | yes |

* Request example:

```json
{
    "user": {
        "email": "john.doe@example.com",
        "password": "secret"
    }
}
```

* **Success response (200 OK)**:

```json
{
    "message": "Login successfully",
    "user": {
        "id": 76,
        "first_name": "Hida",
        "last_name": "Isao",
        "full_name": "Hida Isao",
        "email": "isaohida@gmail.com",
        "avatar": null,
        "organisation": null,
        "default_password": true
    },
    "access_token": "xxx"
}
```

* Failure response (401 Unauthorized):

```json
{
    "message": "Email or password is invalid",
    "http_status_code": 401,
    "error": {
        "code": "UNAUTHORIZED"
    }
}
```

### Social Login

* Path: `api/v2/user/authentication/social_login`
* Method: `POST`
* Authenticate: `no`
* Parameters:

Field | Type | Required | Description
--------- | ------- | ------- | -----------
user[email] | string | yes |
user[first_name] | string | yes |
user[last_name] | string | yes |
user[provider_token] | String | yes
user[provider] | String | yes

* Request example:

```json
{
    "user": {
        "email": "isaohida@gmail.com",
        "first_name": "Isao",
        "last_name":"H",
        "provider_token": "xxx",
        "provider":"google"
    }
}
```

* **Success response (200 OK)**:

```json
{
    "http_status_code": 200,
    "message": "Login successfully",
    "user": {
        "id": 76,
        "first_name": "Isao",
        "last_name": "H",
        "full_name": "Isao H",
        "email": "isaohida@gmail.com",
        "avatar": null,
        "organisation": "Orga",
        "default_password": true
    },
    "access_token": "xxx"
}
```

* Failure response (401 Unauthorized):

```json
{
    "message": "Failed to login. Please check for form errors",
    "http_status_code": 400,
    "error": {
        "code": "BAD_PARAMETERS",
        "errors": [
            {
                "field": "email",
                "message": "Email is invalid"
            }
        ]
    }
}
```

### Register

* Path: `/api/v2/user/registration/register`
* Method: `POST`
* Authenticate: `no`
* Parameters:

Field | Type | Required | Description
--------- | ------- | ------- | -----------
user[email] | string | yes | Max length `255`
user[password] | string | yes | Length `6..128`
user[first_name] | string | yes | Length `1..25`
user[last_name] | string | no | Length `1..25`
user[password] | string
user[password_confirmation]

* Request example:

```json
{
    "user": {
        "email": "john.doe@example.com",
        "first_name": "John",
        "last_name": "Smith",
        "password": "secret",
        "password_confirmation": "secret",
    },
}
```

> Notes: The `access_token` is only returns if user's email is confirmed

* **Success response (200 OK)**:

```json
{
    "http_status_code": 200,
    "message": "Thanks for register with us. You now can login to the DSRT",
    "user": {
        "id": 85,
        "first_name": "test",
        "last_name": "test2",
        "full_name": "test test2",
        "email": "isaohida@gmail.com",
        "avatar": null,
        "organisation": null,
        "default_password": false
    },
    "access_token": "xxx"
}
```

* Failure response (400 Bad Request):

```json
{
    "message": "Failed to register. Please check for form errors",
    "http_status_code": 400,
    "error": {
        "code": "BAD_PARAMETERS",
        "errors": [
            {
                "field": "first_name",
                "message": "First name can not be blank"
            },
        ]
    }
}
```

## Profile
### Get profile details

* Path: `/api/v1/user/profile/me`
* Method: `GET`
* Authenticate: `yes`

* **Success response (200 OK)**:

```json
{
    "http_status_code": 200,
    "user": {
        "id": 76,
        "first_name": "Hida",
        "last_name": "Isao",
        "full_name": "Hida Isao",
        "email": "isaohida@gmail.com",
        "avatar": null,
        "organisation": null,
        "default_password": true
    },
    "access_token": "xxx"
}
```

### Update profile

* Path: `/api/v2/user/profile/update`
* Method: `POST`
* Authenticate: `yes`
* Parameters:

Field | Type | Required | Description
--------- | ------- | ------- | -----------
user[first_name] | string | yes | Length `1..25`
user[last_name] | string | no | Length `1..25`
user[organisation] | string | yes | Max length `255`

* Request example:

```json
{
    "user": {
        "first_name": "John",
        "last_name": "Smith",        
        "organisation": "org"
    }
}
```

* **Success response (200 OK)**:

```json
{
    "http_status_code": 200,
    "message": "Profile update successfully",
    "user": {
        "id": 76,
        "first_name": "Isao",
        "last_name": "H",
        "full_name": "Isao H",
        "email": "isaohida@gmail.com",
        "avatar": null,
        "organisation": "Orga",
        "default_password": true
    }
}
```

* Failure response (400 Bad Request):

```json
{
    "message": "Failed to update profile. Please check for form errors",
    "http_status_code": 400,
    "error": {
        "code": "BAD_PARAMETERS",
        "errors": [
            {
                "field": "first_name",
                "message": "First name can not be blank"
            },
        ]
    }
}
```

## Password
### Update password

* Path: `/api/v2/user/profile/update_password`
* Method: `POST`
* Authenticate: `yes`
* Parameters:

Field | Type | Required | Description
--------- | ------- | ------- | -----------
user[password] | string | yes | Length `6..128`
user[new_password] | string | yes | Length `6..128`

* Request example:
```json
{
    "user": {
        "password": "current_password",
        "new_password":"new_password"
    }
}
```

* **Success response (200 OK)**:

```json
{
    "message": "Password updated successfully",
    "http_status_code": 200
}
```

* Failure response (400 Bad Request):

```json
{
    "message": "Failed to update password",
    "http_status_code": 400,
    "error": {
        "code": "BAD_PARAMETERS",
        "errors": [
            {
                "field": "password",
                "message": "Password is too short (minimum is 6 characters)"
            }
        ]
    }
}
```

### Forgot password
> Use to make request to reset password if user has forgot
> System will send a reset password instructions email if email exists

* Path: `/api/v2/user/password/forgot`
* Method: `POST`
* Authenticate: `no`
* Parameters:

Field | Type | Required | Description
--------- | ------- | ------- | -----------
password | string | yes | The current password of user

* Request example:
```json
{
    "user": {
      "email": "some.email@example.com"
    }
}
```

* **Success response (200 OK)**:

```json
{
    "message": "Please check your email for reset password instructions"",
    "http_status_code": 200
}
```

## Report
### Reports
> Get user reports

* Path: `/api/v2/reports`
* Method: `GET`
* Authenticate: `yes`
* Parameters:


* **Success response(200 OK)**:
```json
{
    "http_status_code": 200,
    "reports": [
        {
            "id": 7,
            "image": null,
            "latitude": "-33.3216",
            "longitude": "149.085",
            "created_at": "19/Jul/2018 02:37PM",
            "updated_at": "19/Jul/2018 02:37PM",
            "surface_water": null,
            "rivers_creeks": null,
            "native_pasture": null,
            "improved_pasture": null,
            "crops": null,
            "supplementary_cattle": null,
            "supplementary_sheep": null,
            "livestock_number_cattle": null,
            "livestock_number_sheep": null,
            "livestock_number_other": null,
            "livestock_condition_cattle": null,
            "livestock_condition_sheep": null,
            "livestock_condition_other": null,
            "surface_water_cmt": null,
            "rivers_creeks_cmt": null,
            "native_pasture_cmt": null,
            "improved_pasture_cmt": null,
            "crops_cmt": null,
            "supplementary_cattle_cmt": null,
            "livestock_number_cattle_cmt": null,
            "livestock_number_sheep_cmt": null,
            "livestock_number_other_cmt": null,
            "livestock_condition_cattle_cmt": null,
            "livestock_condition_sheep_cmt": null,
            "livestock_condition_other_cmt": null,
            "supplementary_sheep_cmt": null,
            "rainfall": "<50mm",
            "rainfall_cmt": "Ttttt",
            "based_on_a_rain_gauge": "",
            "based_on_a_rain_gauge_cmt": "",
            "land_use": "",
            "land_use_cmt": "",
            "pasture_mass": "",
            "pasture_mass_cmt": "",
            "percentage_cover": "",
            "percentage_cover_cmt": "",
            "percentage_green": "",
            "percentage_green_cmt": "",
            "have_you_been_trained_in_pasture_assessment": "",
            "have_you_been_trained_in_pasture_assessment_cmt": "",
            "farm_and_plantation_trees_or_shrubs": "",
            "farm_and_plantation_trees_or_shrubs_cmt": "",
            "livestock_type": "",
            "livestock_type_cmt": "",
            "condition_score": "",
            "condition_score_cmt": "",
            "herd_flock_management": "",
            "herd_flock_management_cmt": "",
            "survival_drought_or_hand_feeding": "",
            "survival_drought_or_hand_feeding_cmt": "",
            "feeding_duration": "",
            "feeding_duration_cmt": "",
            "stock_water": "",
            "stock_water_cmt": "",
            "crop_type": "",
            "crop_type_cmt": "",
            "crop_stage": "",
            "crop_stage_cmt": "",
            "crop_program": null,
            "crop_program_cmt": null,
            "current_yield_estimates": "",
            "current_yield_estimates_cmt": "",
            "has_your_crop_failed": "",
            "has_your_crop_failed_cmt": "",
            "stored_soil_water": "",
            "stored_soil_water_cmt": "",
            "based_on_soil_moisture_probe": "",
            "based_on_soil_moisture_probe_cmt": "",
            "farm_water": "",
            "farm_water_cmt": "",
            "natural_water": "",
            "natural_water_cmt": "",
            "manage_drought_condition": "",
            "manage_drought_condition_cmt": "",
            "drought_severity": "",
            "drought_severity_cmt": "",
            "drought_duration": "",
            "drought_duration_cmt": "",
            "drought_stage": "",
            "drought_stage_cmt": "",
            "stock_breeding": "",
            "stock_breeding_cmt": "",
            "last_grazing": "",
            "last_grazing_cmt": "",
            "crop_damage": "",
            "crop_damage_cmt": "",
            "dry_sown": "",
            "dry_sown_cmt": "",
            "carting_water": "",
            "carting_water_cmt": ""
        },
    ]
}
```

### Following
> Get account holders which following by current user
> Also returns Notice of passing, Funeral details .. incase current user has permission

* Path: `/api/v1/user/friendship/following`
* Method: `GET`
* Authenticate: `yes`
* Parameters:

Field | Type | Required | Description
--------- | ------- | ------- | -----------
level | string | `yes` | The filter for frienship level. Allowed values: `trusted_friend`, `lifelink`
page | integer | no | The page number
per | integer | no | The page size
sort | string | no | The sort field. Sortable fields: `account_holder_first_name`, `account_holder_last_name` , `account_holder_full_name`, `status`, `created_at`
order | string | no | The sort direction

* Request example:

```json
{
    "level": "trusted_friend",
    "page": 1,
    "per": 10,
    "sort": "account_holder_full_name",
    "order": "asc"
}
```

* **Success response(200 OK)**:
```json
{
    "http_status_code": 200,
    "users": [
        {
            "id": 1,
            "first_name": "Someone1",
            "middle_name": null,
            "last_name": null,
            "email": "someone1@example.com",
            "level": "trusted_friend",
            "status": "pending",
            "invite_token": null,
            "avatar": {
                "origin": "http://example.com/path/to/image.png",
                "thumb": "http://example.com/path/to/image.png"
            },
            "death_notice": null,
            "invite_sent_at": null,
            "account_holder": {
                "id": 1,
                "first_name": "Someone1",
                "middle_name": null,
                "last_name": null,
                "email": "someone1@example.com",
                "avatar": {
                    "origin": "http://example.com/path/to/image.png",
                    "thumb": "http://example.com/path/to/image.png"
                },
            }
        },
        {
            "id": 2,
            "first_name": "Someone2",
            "middle_name": null,
            "last_name": null,
            "full_name": "Someone2",
            "email": "someone2@example.com",
            "avatar": null,
            "status": "accepted",
            "invite_token": null,
            "invite_sent_at": null,
            "account_holder": {
                "id": 2,
                "first_name": "Someone2",
                "middle_name": null,
                "last_name": null,
                "full_name": "Someone2",
                "email": "someone2@example.com",
                "avatar": null
            },
            "death_notice": {
                "status": "authorised",
                "date_of_death": "2016-08-11",
                "initiated_at": "2016-08-10T09:46:03.000Z",
                "authorised_at": "2016-08-10T09:46:36.000Z",
                "setting": {
                    "content": "Proin eget tortor risus. Lorem ipsum dolor sit amet ...",
                    "email_receive_condolences": "some.email@example.com",
                    "address_receive_flowers": "123 Main Street",
                    "ewake_visibility": "public"
                },
                "funeral": {
                    "location": "123 Main Street",
                    "time": "2016-08-18T11:55:00.000Z",
                    "details": null,
                    "sent_at": "2016-08-10T10:28:11.000Z"
                },
                "initiator": {
                    "id": 3,
                    "first_name": "Luan 3",
                    "last_name": null,
                    "middle_name": null,
                    "full_name": "Someone4",
                    "email": "someone3@example.com",
                    "avatar": null
                },
                "authoriser": {
                    "id": 4,
                    "first_name": "Someone4",
                    "last_name": null,
                    "middle_name": null,
                    "full_name": "Someone4",
                    "email": "someone4@example.com",
                    "avatar": null
                }
            }
        }
    ],
    "total": 2,
    "per": 10,
    "sort": "created_at",
    "order": "desc"
}
```

### Add One

> Use to invite someone to become Trusted Friend/Recipient/Grey List of current user.

* Path: `/api/v1/user/friendship/add_one`
* Method: `POST`
* Authenticate: `yes`
* Parameters:

Field | Type | Required | Description
--------- | ------- | ------- | -----------
user[level] | string | `yes` | The frienship level. Allowed values: `trusted_friend`, `lifelink`, `grey_list`
user[email] | string | `yes` | Max length `255`
user[first_name] | string | `yes` | Length `1..25`
user[middle_name] | string | no | Length `1..25`
user[last_name] | string | no | Length `1..25`

* Request example:

```json
{
    "user" : {
        "email": "someone1@example.com",
        "first_name": "John",
        "middle_name": "",
        "last_name": "Smith",
        "level": "trusted_friend"
    }
}
```

* **Success response(200 OK)**:

```json
{
    "http_status_code": 200,
    "message": "Invitation created successfully",
    "user": {
        "id": 1,
        "email": "someone1@example.com",
        "first_name": "Someone1",
        "middle_name": null,
        "last_name": null,
        "full_name": "Someone1",
        "level": "trusted_friend",
        "status": "pending",
        "avatar": {
            "origin": "http://example.com/path/to/image.png",
            "thumb": "http://example.com/path/to/image.png"
        },
        "mail_delivery_status": "pending",
        "invite_sent_at": "2016-08-19T08:42:08.000Z",
        "created_at": "2016-08-19T08:42:08.000Z"
    }
}
```

* Failure response(400 Bad Request):

```json
{
    "message": "Failed to create invitation",
    "http_status_code": 400,
    "error": {
        "code": "BAD_PARAMETERS",
        "errors": [
            {
              "field": "first_name",
              "message": "First name can't be blank"
            }
        ]
    }
}
```

### Add Many
> Use to add multiple invite users to become Trusted Friend/Recipient/Grey List of current user

> Note: the response status always `200`! Check for `success_invitations` and `failure_invitations` with index of each entry in array of request body.

* Path: `/api/v1/user/friendship/add_many`
* Method: `POST`
* Authenticate: `yes`
* Parameters:

> Same as `Add One` but passing users information in an array

Field | Type | Required | Description
------------ | ------- | ------- | -----------
users[level] | string | `yes` | The frienship level. Allowed values: `trusted_friend`, `lifelink`, `grey_list`
users[email] | string | `yes` | Max length `255`
users[first_name] | string | `yes` | Length `1..25`
users[middle_name] | string | no | Length `1..25`
users[last_name] | string | no | Length `1..25`

* Request example:
```json
{
    "users": [
        {
            "email": "someone1@example.com",
            "first_name": "Someone1",
            "middle_name": "",
            "last_name": "",
            "level": "trusted_friend"
        },
        {
            "email": "someone2@example.com",
            "first_name": "Someone2",
            "middle_name": "",
            "last_name": "",
            "level": "lifelink"
        }
    ]
}
```

* **Success response(200 OK)**:
```json
{
    "http_status_code": 200,
    "message": "The invitations created successfully",
    "success_invitations": [
        {
          "id": 1,
          "email": "someone1@example.com",
          "first_name": "Someone1",
          "middle_name": null,
          "last_name": null,
          "full_name": "Someone1",
          "level": "trusted_friend",
          "status": "pending",
          "avatar": null,
          "mail_delivery_status": null,
          "invite_sent_at": null,
          "created_at": "2016-08-20T04:50:29.797Z"
        }
    ],
    "failure_invitations": [
        {
          "email": "someone2@example.com",
          "first_name": "",
          "level": "lifelink",
          "index": 1,
          "error": "First name can't be blank"
        }
    ]
}
```

### Resend invitation email

> Use to resend the invitation email to a `pending` TF or LL friendship

* Path: `/api/v1/user/friendship/resend_invitation`
* Method: `POST`
* Authenticate: `yes`
* Parameters:

Field | Type | Required | Description
------------ | ------- | ------- | -----------
user[email] | string | `yes` | Max length `255`.

* Request example:
```json
{
    "user": {
        "email": "someone@example.com"
    }
}
```

* **Success respon(200 OK)**:
```JSON
{
    "message": "The invitation email has been sent successfully",
    "http_status_code": 200
}
```

* Failure response(400 Bad Request):

> User must wait for 24 hours since the last time sent email

```JSON
{
    "message": "Could not send the invitation. Please wait for a day to try this action",
    "http_status_code": 400,
    "error": {
        "code": "BAD_REQUEST"
    }
}
```

### Update friendship

> Use to update a `pending` friendship. Change level or name for example.
An new invitation email will send if level changed(except Grey List)

* Path: `/api/v1/user/friendship/update_invitation`
* Method: `POST`
* Authenticate: `yes`
* Parameters:

Field | Type | Required | Description
--------- | ------- | ------- | -----------
user[level] | string | `yes` | The frienship level. Allowed values: `trusted_friend`, `lifelink`, `grey_list`
user[email] | string | `yes` | Max length `255`
user[first_name] | string | `yes` | Length `1..25`
user[middle_name] | string | no | Length `1..25`
user[last_name] | string | no | Length `1..25`

* Request example:

```json
{
    "user" : {
        "email": "someone1@example.com",
        "first_name": "John",
        "middle_name": "",
        "last_name": "Smith",
        "level": "trusted_friend"
    }
}
```

* **Success response(200 OK)**:

```json
{
    "http_status_code": 200,
    "message": "The invitation has been updated successfully",
    "user": {
        "id": 1,
        "email": "someone1@example.com",
        "first_name": "Someone1",
        "middle_name": null,
        "last_name": null,
        "full_name": "Someone1",
        "level": "trusted_friend",
        "status": "pending",
        "avatar": {
            "origin": "http://example.com/path/to/image.png",
            "thumb": "http://example.com/path/to/image.png"
        },
        "mail_delivery_status": "pending",
        "invite_sent_at": "2016-08-19T08:42:08.000Z",
        "created_at": "2016-08-19T08:42:08.000Z"
    }
}
```

* Failure response(400 Bad Request):

```json
{
    "message": "Failed to update invitation",
    "http_status_code": 400,
    "error": {
        "code": "BAD_PARAMETERS",
        "errors": [
            {
              "field": "first_name",
              "message": "First name can't be blank"
            }
        ]
    }
}
```

### Accept invitation

> Use to accept an invitation created by Add One/Add Many function.
In the context that current user has been invited to become Lifelink or Trusted friend of some AH user

> Incase the level is `trusted_friend`, the API required be authenticated as user must have an account.
If level is `lifelink`, authenticate is not required.

* Path: `/api/v1/user/friendship/accept_invitation`
* Method: `POST`
* Authenticate: `depends`
* Parameters:

Field | Type | Required | Description
--------- | ------- | ------- | -----------
invite_token | string | `yes` |

* Request example:

```json
{
    "invite_token": "xxx"
}
```

* **Success response(200 OK)**:

```json
{
    "message": "Invitation has been accepted",
    "http_status_code": 200
}
```

* Failure response(400 Bad Request):

> Incase the access token not provided or the token is not valid(already accepted, rejected ot not existing.


```json
{
    "message": "The invitation is already accepted or rejected",
    "http_status_code": 400,
    "error": {
        "code": "BAD_REQUEST"
    }
}
```

### Reject invitation

> Use to reject an invitation created by Add One/Add Many function
In the context that user has been invited to become Lifelink or Trusted friend of some AH user

* Path: `/api/v1/user/friendship/reject_invitation`
* Method: `POST`
* Authenticate: `no`
* Parameters:

Field | Type | Required | Description
--------- | ------- | ------- | -----------
invite_token | string | `yes` |

* Request example:

```json
{
    "invite_token": "xxx"
}
```

* **Success response(200 OK)**:

```json
{
    "message": "The invitation has been rejected",
    "http_status_code": 200
}
```

* Failure response(400 Bad Request):

> Incase the access token not provided or the token is not valid(already accepted, rejected ot not existing.


```json
{
    "message": "The invitation is already accepted or rejected",
    "http_status_code": 400,
    "error": {
        "code": "BAD_REQUEST"
    }
}
```
