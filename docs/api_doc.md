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

### Get report by id
> Get reports by id

* Path: `/api/v2/reports/:report_id`
* Method: `GET`
* Authenticate: `yes`
* Parameters:

* **Success response(200 OK)**:
```json
{
    "http_status_code": 200,
    "report": {
        "id": 13,
        "image": null,
        ......
        "carting_water": null,
        "carting_water_cmt": null
    }
}
```

### Update Report

> update a report

* Path: `/api/v2/reports/:id`
* Method: `PUT`
* Authenticate: `yes`
* Parameters:

Field | Type | Required | Description
--------- | ------- | ------- | -----------
report[id] | integer
report[user_id] | integer
report[image]
report[latitude]
report[longitude]
report[rainfall]
report[rainfall_cmt]
report[based_on_a_rain_gauge]
report[based_on_a_rain_gauge_cmt]
report[manage_drought_condition]
report[manage_drought_condition_cmt]
report[drought_severity]
report[drought_severity_cmt]
report[drought_duration]
report[drought_duration_cmt]
report[drought_stage]
report[drought_stage_cmt]
report[land_use]
report[land_use_cmt]
report[pasture_mass]
report[pasture_mass_cmt]
report[percentage_cover]
report[percentage_cover_cmt]
report[percentage_green]
report[percentage_green_cmt]
report[have_you_been_trained_in_pasture_assessment]
report[have_you_been_trained_in_pasture_assessment_cmt]
report[farm_and_plantation_trees_or_shrubs]
report[farm_and_plantation_trees_or_shrubs_cmt]
report[livestock_type]
report[livestock_type_cmt]
report[condition_score]
report[condition_score_cmt]
report[herd_flock_management]
report[herd_flock_management_cmt]
report[stock_breeding]
report[stock_breeding_cmt]
report[last_grazing]
report[last_grazing_cmt]
report[survival_drought_or_hand_feeding]
report[survival_drought_or_hand_feeding_cmt]
report[feeding_duration]
report[feeding_duration_cmt]
report[stock_water]
report[stock_water_cmt]
report[crop_type]
report[crop_type_cmt]
report[crop_stage]
report[crop_stage_cmt]
report[dry_sown]
report[dry_sown_cmt]
report[crop_program]
report[crop_program_cmt]
report[current_yield_estimates]
report[current_yield_estimates_cmt]
report[has_your_crop_failed]
report[has_your_crop_failed_cmt]
report[crop_damage]
report[crop_damage_cmt]
report[stored_soil_water]
report[stored_soil_water_cmt]
report[based_on_soil_moisture_probe]
report[based_on_soil_moisture_probe_cmt]
report[farm_water]
report[farm_water_cmt]
report[natural_water]
report[natural_water_cmt]
report[carting_water]
report[carting_water_cmt]

* Request example:

```json
{
    "report" : {
        "latitude": "-33.321628",
        "longitude": "149.085161"
    }
}
```

* **Success response(200 OK)**:

```json
{
    "http_status_code": 200,
    "message": "Report update successfully",
    "report": {
        "id": 13,
        "image": null,
        "latitude": "-33.321628",
        "longitude": "149.085161",
        ...........
    }
}
```

* Failure response(400 Bad Request):

```json
{
    "message": "An error has occurred",
    "http_status_code": 404,
    "error": {
        "code": "RESOURCE_NOT_FOUND"
    }
}
```

### Create Report
> Create user report

* Path: `/api/v2/reports`
* Method: `POST`
* Authenticate: `yes`
* Parameters:

Field | Type | Required | Description
--------- | ------- | ------- | -----------
report[id] | integer
report[user_id] | integer
report[image]
report[latitude]
report[longitude]
report[rainfall]
report[rainfall_cmt]
report[based_on_a_rain_gauge]
report[based_on_a_rain_gauge_cmt]
report[manage_drought_condition]
report[manage_drought_condition_cmt]
report[drought_severity]
report[drought_severity_cmt]
report[drought_duration]
report[drought_duration_cmt]
report[drought_stage]
report[drought_stage_cmt]
report[land_use]
report[land_use_cmt]
report[pasture_mass]
report[pasture_mass_cmt]
report[percentage_cover]
report[percentage_cover_cmt]
report[percentage_green]
report[percentage_green_cmt]
report[have_you_been_trained_in_pasture_assessment]
report[have_you_been_trained_in_pasture_assessment_cmt]
report[farm_and_plantation_trees_or_shrubs]
report[farm_and_plantation_trees_or_shrubs_cmt]
report[livestock_type]
report[livestock_type_cmt]
report[condition_score]
report[condition_score_cmt]
report[herd_flock_management]
report[herd_flock_management_cmt]
report[stock_breeding]
report[stock_breeding_cmt]
report[last_grazing]
report[last_grazing_cmt]
report[survival_drought_or_hand_feeding]
report[survival_drought_or_hand_feeding_cmt]
report[feeding_duration]
report[feeding_duration_cmt]
report[stock_water]
report[stock_water_cmt]
report[crop_type]
report[crop_type_cmt]
report[crop_stage]
report[crop_stage_cmt]
report[dry_sown]
report[dry_sown_cmt]
report[crop_program]
report[crop_program_cmt]
report[current_yield_estimates]
report[current_yield_estimates_cmt]
report[has_your_crop_failed]
report[has_your_crop_failed_cmt]
report[crop_damage]
report[crop_damage_cmt]
report[stored_soil_water]
report[stored_soil_water_cmt]
report[based_on_soil_moisture_probe]
report[based_on_soil_moisture_probe_cmt]
report[farm_water]
report[farm_water_cmt]
report[natural_water]
report[natural_water_cmt]
report[carting_water]
report[carting_water_cmt]

* Request example:
```json
{
    "report": {
        "user_id": 81,        
        "latitude": "-33.321628",
        "longitude": "149.085161",
        ...........
    }
}
```

* **Success response(200 OK)**:
```json
{
    "http_status_code": 200,
    "message": "Report created successfully",
    "report": {
        "id": 14,
        "latitude": null,
    }
}
```

### Delete a report

> Report delete

* Path: `/api/v2/reports/:id/delete`
* Method: `DELETE`
* Authenticate: `yes`
* Parameters:

* **Success respon(200 OK)**:
```JSON
{
    "http_status_code": 200,
    "message": "Report delete successfully"
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
