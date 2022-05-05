## API documentation (Draft)
- Version: 1.0
- Last update: 2016-07-19

## I. Conventions
  - The HTTP(or HTTPS) protocol will be used for communication between a client side and the server.
  - Uses UTF-8 character encoding. Request headers should include a Content-Type of application/json; charset=utf-8
  - All the responses will be in JSON format.
  - All timestamps are ISO-8601 formatted.
  - All responses from API will be `200 OK` even a server error is occurred. Client side should check `http_status_code` and `error code` to know exactly what happened.

## II. Authentication
To authenticate against backend server, client side need an access token. To receive it, client side make a request to login API. The access token should be included in header of requests which required authorization. Each access token has a expire time and can be revoked anytime.

## III. Errors Handling
  Codes in the 4xx range indicate an error that failed given the information provided (e.g., parameters is not pass validation, access token expired, ...), and codes in the 5xx range indicate an error from server.
  However, not all errors map cleanly onto HTTP response codes, so there is some our own error codes to indicate special states: access token expired, server maintenance ...
  The bellow list is all the error codes from server:

  - `BAD_PARAMETERS` : Some parameters are not passes validations. It usually cause from user inputss.
  - `ACCESS_TOKEN_INVALID`: User access token is not valid or expired. Client side should re-authenticate to get new access token.
  - `BAD_REQUEST`: The request form is illegal or not in expected format from server. It usually a technical problem with client side.
  - `UNAUTHORIZED`: When server failed to find user with given credentials.
  - `ACCESS_DENINED`: The request is not allowed due to user do not have permission to access a resource or post some changes.
  - `RESOURCE_NOT_FOUND`: The request asked for a resource which not exists or has been deleted from server.
  - `ENDPOINT_NOT_EXISTS`: The requested path does not exists, or cannot handler by any endpoint. It might in the case HTTP method is invalid.
  - `SERVER_ERROR`: When an unhandled error occurred. AKA technical problem.
  - `SERVER_MAINTENANCE`: Server maintenance now in progress, it might for deployment.

## IV. API Endpoints
### Register
- Notes:
  - If `invite_token` is correct, the email not need to confirm.
  - If free period is not available, request body is has containt a `coupon_code`, or process payment with two params: `payment_option` and `payment_method_nonce`
- Request:
  - Path: `/api/v1/user/registration/register`
  - Method: `POST`
  - Body:
    ```JSON
    {
      "user": {
        "email": "email@example.com",
        "first_name": "John",
        "last_name": "Smith",
        "password": "123456",
        "level": "account_holder|lifelink",
        "dob": "yyy-mm-dd",
        "gender": "male|female|other",
        "contact_number": "123456",
        "country": "US",
        "avatar": "File"
      },
      "invite_token": "xxx", // Optional; use when register via invite
      "coupon_code": "xxx", // Optional
      "payment_option": "", // `one_off` or `annual_recurring`
      "payment_method_nonce": "xxx" // The braintree payment token
    }
    ```
- Response:
  ```JSON
  {
      "message": "Register successfully",
      "http_status_code": 200,
      "user": {
          "id": 1,
          "email": "email@example.com",
          "secondary_email": null,
          "secondary_email_confirmed": null,
          "first_name": "John",
          "last_name": "Smith",
          "password": "Smith",
          "level": "account_holder",
          "dob": "1990-12-24",
          "gender": "male",
          "contact_number": "123456",
          "country": "US"
      }
  }
  ```
### Login
- Request:
  - Path: `/api/v1/user/authentication/login`
  - Method: `POST`
  - Body:
    ```JSON
    {
      "user": {
          "email": "email@example.com",
          "password": "123456"
      },
    }
    ```
- Response:
    ```JSON
    {
        "message": "Login successfully",
        "http_status_code": 200,
        "user": {
            "id": 1,
            "email": "email@example.com",
            "secondary_email": null,
            "secondary_email_confirmed": null, // true/false if secondary_email is present
            "first_name": "John",
            "last_name": "Smith",
            "level": "account_holder",
            "dob": "1990-12-24",
            "gender": "male",
            "contact_number": "123456",
            "country": "US"
        }
    }
    ```
### Profile
#### Get profile
  - Request:
    - Path: `/api/v1/user/profile/me`
    - Method: `GET`
    - Header:
      ```
      X-User-Access-Token: <User's ID>
      ```
  - Response:
      ```JSON
      {
          "http_status_code": 200,
          "user": {
              "id": 1,
              "email": "email@example.com",
              "secondary_email": null,
              "secondary_email_confirmed": null, // true/false if secondary_email is present
              "first_name": "John",
              "last_name": "Smith",
              "level": "account_holder",
              "dob": "1990-12-24",
              "gender": "male",
              "contact_number": "123456",
              "country": "US",
              "avatar": { // null if not set
                "origin": "http://example.com/xxx.png",
                "thumb": "http://example.com/yyy.png"
              }
          }
      }
      ```

#### Update profile
  - Request:
    - Path: `/api/v1/user/profile/update`
    - Method: `POST`
    - Header:
      ```
      X-User-Access-Token: <User's ID>
      ```
    - Body:
      ```JSON
      {
          "user": {
              "first_name": "John",
              "middle_name": "Cr.",
              "last_name": "Smith",
              "secondary_email": "someemail@example.com", // leave empty to remove if any
              "dob": "yyy-mm-dd",
              "gender": "male|female|other",
              "contact_number": "123456",
              "country": "US",
              "avatar": "File"
          }
      }
      ```
  - Response:
      ```JSON
      {
          "message": "Profile update successfully",
          "http_status_code": 200,
          "user": {
              "id": 1,
              "email": "email@example.com",
              "first_name": "John",
              "middle_name": "Cr.",
              "last_name": "Smith",
              "level": "account_holder",
              "dob": "1990-12-24",
              "gender": "male",
              "contact_number": "123456",
              "country": "US"
          }
      }
      ```

#### Close account
  - Request:
    - Path: `/api/v1/user/profile/close_account`
    - Method: `POST`
    - Body:
      ```JSON
      {
        "password": "xxx" // the current password
      }
      ```

#### Update password
  - Request:
    - Path: `/api/v1/user/profile/update_password`
    - Method: `POST`
    - Header:
      ```
      X-User-Access-Token: <User's ID>
      ```
    - Body:
      ```JSON
      {
          "user": {
              "password": "newpass",
          }
      }
      ```
  - Response:
      ```JSON
      {
          "message": "Password update successfully",
          "http_status_code": 200
      }
      ```

#### Forgot password
  - Notes:
    - Use for make request to reset password if user has forgot
    - System will send a reset password instructions email if email exists
  - Request:
    - Path: `/api/v1/user/password/forgot`
    - Method: `POST`
    - Body:
      ```JSON
      {
        "user": {
          "email": "somemail@example.com",
        }
      }
      ```

### Friendship
#### Get list trusted friends, lifelinks and grey list
  - Notes:
    - Get user's trusted friends, lifelinks, grey list
    - Use in `Account Holder` tab
    - Specified `level` parameter for filtering, or leave blank to get all
    - Pagination params is required
    - Sortable fields: `first_name`, `last_name`, `full_name`, `email`, `level`, `status`, `created_at`
  - Request:
    - Path: `/api/v1/user/friendship/followers`
    - Method: `GET`
    - Header:
      ```
      X-User-Access-Token: <User's ID>
      ```
    - Body:
      ```JSON
      {
        "level": "<empty>|trusted_friend|lifelink|grey_list",
        "page": 1, // Optional, default is 1
        "per": 10, // Optional, default is 10
        "sort": "<sort field>",
        "order": "asc|desc"
      }
      ```
  - Response:
    ```JSON
    {
      "http_status_code": 200,
      "users": [
        {
          "first_name": "Friend",
          "middle_name": null,
          "last_name": "One",
          "level": "trusted_friend",
          "status": "pending",
          "mail_delivery_status": "pending|failure|success"
          "created_at": "2015-12-26T01:51:50.000Z",
          "invite_sent_at": "2015-12-26T01:51:50.000Z"
        },
        {
          "first_name": "Friend",
          "middle_name": null,
          "last_name": "Two",
          "level": "trusted_friend",
          "status": "accepted",
          "mail_delivery_status": "pending|failure|success",
          "created_at": "2015-12-26T01:51:50.000Z",
          "invite_sent_at": "2015-12-26T01:51:50.000Z"
        },
        {
          "first_name": "Recipient",
          "middle_name": null,
          "last_name": "One",
          "level": "lifelink",
          "status": "accepted",
          "mail_delivery_status": "pending|failure|success",
          "invite_sent_at": "2015-12-26T01:51:50.000Z",
          "created_at": "2015-12-26T01:51:50.000Z"
        },
        ...
      ],
      "total": 3,
      "per": 10,
      "sort": "<sort field>",
      "order": "asc|desc"
    }
    ```

#### Get list following account holders
  - Notes:
    - Get account holders which following by current user
    - Use in `Trusted Friend` tab: filter `level` by `trusted_friend`
    - Use in `Recipient` tab: filter `level` by `lifelink`
    - Pagination params is required
    - Sortable fields: `account_holder_first_name`, `account_holder_last_name` , `account_holder_full_name`, `status`, `created_at`
  - Request:
    - Path: `/api/v1/user/friendship/following`
    - Method: `GET`
    - Header:
      ```
      X-User-Access-Token: <User's ID>
      ```
    - Body:
      ```JSON
      {
        "level": "trusted_friend|lifelink", // Required
        "page": 1, // Optional, default is 1
        "per": 10, // Optional, default is 10.
        "sort": "<sort field>",
        "order": "asc|desc"
      }
      ```
  - Response:
    ```JSON
    {
      "http_status_code": 200,
      "users": [
        {
          "id": 1,
          "first_name": "Account Holder",
          "middle_name": null,
          "last_name": "One",
          "level": "trusted_friend", // Meaning the current user is Trusted Friend of the account holder,
          "initiated_by_account_holder": true, // Saying the relationship is either initiated by AH or current user
          "status": "accepted",
          "created_at": "2015-12-26T01:51:50", // The created time of the relationship
          "invite_sent_at": "2015-12-26T01:51:50.000Z",
          "death_notice": { // Only available if the current user has accepted as trusted friend
            "status": 'open|initiated|authorised', // `open` meaning the death notice not initiated, so the user can initiate.
            "date_of_death": "2015-12-17", // Available when the `status` is `initiated` or `authorised`, use for displaying in popup for confirm death e-notice,
            "funeral": { // null if funeral details is not announced by anyone
              "location": "123 Main Street",
              "time": "2015-12-20 10:00:00",
              "details": "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla porttitor accumsan tincidunt."
            },
            "setting": { // The death notice settings of the account holder. If the AH is not setup yet, use the defaut template.
              "content": "Voluptatem ab facilis ....",
              "email_receive_condolences": "some@one.com",
              "address_receive_flowers": "123 Main street",
              "ewake_visibility": "public"
            },
            "initiated_at": "2016-01-10T14:59:32.000Z",
            "authorised_at": "2016-01-15T15:32:23.000Z",
            "initiator": { // Infor of death notice initiator if initiated
              "first_name": "Quang",
              "last_name": "Nguyen",
              "email": "quang@gmail.com"
            },
            "authoriser": {  // Infor of death notice authoriser if authorised
              "first_name": "Quang",
              "last_name": "Nguyen",
              "email": "quang@gmail.com"
            }
          }
        },
        {
          "id": 2,
          "first_name": "Account Holder",
          "middle_name": null,
          "last_name": "Three",
          "level": "lifelink", // Meaning the current user is Recipient of the account holder
          "status": "pending", // Meaning this relationship still wating for approval by the current user
          "created_at": "2015-12-26T01:51:50.000Z",
          "invite_sent_at": "2015-12-26T01:51:50.000Z",
          "death_notice": null // null due to is pending or lifelink
        },
        ...
      ],
      "total": 3,
      "per": 10.
      "sort": "<sort field>",
      "order": "asc|desc"
    }
    ```

#### Add one
  - Notes:
    - Use for invite an anyone to become Trusted Friend/Recipient/Grey List of account holder user.
  - Request:
    - Path: `/api/v1/user/friendship/add_one`
    - Method: `POST`
    - Header:
      ```
      X-User-Access-Token: <User's ID>
      ```
    - Body:
      ```JSON
      {
        "user" : {
          "email": "email@example.com",// required
          "first_name": "John", // required
          "middle_name": "",
          "last_name": "Smith", // required
          "level": "trusted_friend|lifelink|grey_list" // required
        }
      }
      ```
  - Response:
    ```JSON
    {
      "message": "Invitation created successfully",
      "http_status_code": 200
    }
    ```

#### Add Many
  - Notes:
    - Use for multiple invite to become Trusted Friend/Recipient/Grey List of account holder user.
  - Request:
    - Path: `/api/v1/user/friendship/add_many`
    - Method: `POST`
    - Header:
      ```
      X-User-Access-Token: <User's ID>
      ```
    - Body:
      ```JSON
      {
        "users": [
          {
            "email": "email1@example.com",// required
            "first_name": "John", // required
            "middle_name": "",
            "last_name": "Smith", // required
            "level": "trusted_friend|lifelink|grey_list" // required
          },
          {
            "email": "email2@example.com",// required
            "first_name": "Britney", // required
            "middle_name": "",
            "last_name": "Spear", // required
            "level": "trusted_friend|lifelink|grey_list" // required
          }
        ]
      }
      ```
  - Success response:
    ```JSON
    {
      "message": "Invitations created successfully",
      "http_status_code": 200,
      "success_invitations": [
        {
          "id": 1,
          "first_name": "John",
          "middle_name": null,
          "last_name": "Doe",
          "level": "trusted_friend",
          "status": "pending",
          "mail_delivery_status": "pending|failure|success"
          "created_at": "2015-12-26T01:51:50.000Z",
        },
        {
          "id": 2,
          "first_name": "Friend",
          "middle_name": null,
          "last_name": "Two",
          "level": "trusted_friend",
          "status": "accepted",
          "mail_delivery_status": "pending|failure|success",
          "created_at": "2015-12-26T01:51:50.000Z"
        }
      ],
      "failure_invitations": [
        {
          "first_name": "John",
          "middle_name": null,
          "email": "email@example.com",
          "last_name": "Doe",
          "level": "lifelink",
          "error": "Email is already invited"
        }
      ]
    }
    ```

#### Resend invitation email
- Notes: Use in user details modal after click to a row in `Account Holder` tab
- Request:
  - Path: `/api/v1/user/friendship/resend_invitation`
  - Method: `POST`
  - Header:
    ```
    X-User-Access-Token: <User's ID>
    ```
  - Body:
    ```JSON
    {
      "user": {
        "email": "inviteemail@example.com", // required
      }
    }
    ```

- Success response:
  ```JSON
  {
    "message": "The invitation email has been sent",
    "http_status_code": 200
  }
  ```

#### Update invitation
- Notes:
  - Use to update an invitation created by Add One/Add Many function
  - Use in `Account Holder` tab
- Request:
  - Path: `/api/v1/user/friendship/update_invitation`
  - Method: `POST`
  - Header:
    ```
    X-User-Access-Token: <User's ID>
    ```
  - Body:
    ```JSON
    {
      "user": {
        "email": "newemail@example.com", // required, readonly
        "first_name": "John", // optional
        "middle_name": "", // optional
        "last_name": "Smith", // optional
        "level": "trusted_friend|lifelink|grey_list" // optional
      }
    }
    ```
- Success response:
  ```JSON
  {
    "message": "Invitation has been updated",
    "http_status_code": 200
  }
  ```

#### Accept invitation
  - Notes:
    - Use to accept an invitation created by Add One/Add Many function
    - In the context that current user has been invited to become lifelink or trusted friend of AH users
    - Use in `Trusted Friend` or `Recipient` tab or modal show pending invitation
  - Request:
    - Path: `/api/v1/user/friendship/accept_invitation`
    - Method: `POST`
    - Body:
      ```JSON
      {
        "invite_token": "xxx"
      }
      ```
  - Response:
    ```JSON
    {
      "message": "Invitation has been accepted",
      "http_status_code": 200
    }
    ```

#### Reject invitation
- Notes:
  - Use to reject an invitation created by Add One/Add Many function
  - In the context that current user has been invited to become lifelink or trusted friend of AH users
  - Use in `Trusted Friend` or `Recipient` tab or modal show pending invitation
  - This action is can not be undo since the invitation will be deleted
- Request:
  - Path: `/api/v1/user/friendship/reject_invitation`
  - Method: `POST`
  - Body:
    ```JSON
    {
      "invite_token": "xxx"
    }
    ```
- Response:
  ```JSON
  {
    "message": "Invitation has been rejected",
    "http_status_code": 200
  }
  ```

#### Remove follower
- Notes:
  - Use to remove a Trusted Friend, Recipient or Grey List in `Account Holder` tab
  - This action is use for both established relationships and invitations(pending for confirmation)
  - This action is can not be undo.
- Request:
  - Path: `/api/v1/user/friendship/remove_follower`
  - Method: `POST`
  - Header:
    ```
    X-User-Access-Token: <User's ID>
    ```
  - Body:
    ```JSON
    {
      "user" : {
        "email": "email@example.com", // required
      }
    }
    ```
- Response:
  ```JSON
  {
    "message": "The relationship has been removed",
    "http_status_code": 200
  }
  ```

#### Unfollow Account Holder
- Notes:
  - Use to remove Account Holder from LifeLink tab
- Request:
  - Path: `/api/v1/user/friendship/unfollow`
  - Method: `POST`
  - Header:
    ```
    X-User-Access-Token: <User's ID>
    ```
  - Body:
    ```JSON
    {
      "user" : {
        "email": "email@example.com", // required, Account Holder's email
      }
    }
    ```
- Response:
  ```JSON
  {
    "message": "The relationship has been removed",
    "http_status_code": 200
  }
  ```


### Death E-Notice
#### Get info
- Notes:
  - Use to get current setting of Death E-Notice
- Request:
  - Path: `/api/v1/user/death_notice/info`
  - Method: `GET`
  - Header:
    ```
    X-User-Access-Token: <User's ID>
    ```
  - Body: empty
- Response:
  ```JSON
  {
    "http_status_code": 200,
    "death_notice": { // null if not setup
      "content": "Proin eget tortor risus. Cras ultricies ...",
      "email_receive_condolences": "",
      "address_receive_flowers": "",
      "ewake_visibility": "public|lifelink_only"
    }
  }
  ```

#### Save
- Notes:
  - Use for saving(for both create and update) Death E-notice
- Request:
  - Path: `/api/v1/user/death_notice/save`
  - Method: `POST`
  - Header:
    ```
    X-User-Access-Token: <User's ID>
    ```
  - Body:
    ```JSON
    {
      "death_notice": {
        "content": "Proin eget tortor risus. Cras ultricies ...", // required
        "email_receive_condolences": "somemail@example.com",
        "address_receive_flowers": "123 Main Street",
        "ewake_visibility": "public|lifelink_only" // required
      }
    }
    ```

#### Initiate
- Notes:
  - Use for Initiate death notice for a AH user
  - Use in `Trusted Friend` tab
- Request:
  - Path: `/api/v1/user/death_notice/initiate`
  - Method: `POST`
  - Header:
    ```
    X-User-Access-Token: <User's ID>
    ```
  - Body:
    ```JSON
    {
      "initiation": {
        "email": "someone@example.com", // required; the email address of Account Holder
        "date_of_death": "yyy-mm-dd", // required
        "confirm_email_address": true/false, // required; can be 'yes'/'no' or 1/0; choosen of user on `email to receive condolences` checkbox
        "confirm_physical_address": true/false, // required; can be 'yes'/'no' or 1/0; choosen of user on`physical address to receive flowers` checkbox
      }
    }
    ```
- Success response:
  ```JSON
    {
      "http_status_code": 200,
      "message": "The death e-notice of <AH's full name> has been initiated. Please waiting for confirmation .."
    }
  ```
- Failure response: there are 5 failure states:
** The Account Holder email is not found in system.
** The current user is not a trusted friend(accepted) of the AH user.
** The Account Holder has only one trusted friend.
** The death notice is already initiated.
** The `date of death` is not valid. Error code is BAD_PARAMETERS in this case.

#### Authorise
- Notes:
  - Use for authorise a death e-notice for a AH user
  - Available if the status of AH user' death e-notice is `initiated`
  - Use in `Trusted Friend` tab
- Request:
  - Path: `/api/v1/user/death_notice/authorise`
  - Method: `POST`
  - Header:
    ```
    X-User-Access-Token: <User's ID>
    ```
  - Body:
    ```JSON
    {
      "authorisation": {
        "email": "someone@example.com", // required; the email address of Account Holder
        "confirm_email_address": true/false, // required; can be 'yes'/'no' or 1/0; choosen of user on `email to receive condolences` checkbox
        "confirm_physical_address": true/false, // required; can be 'yes'/'no' or 1/0; choosen of user on`physical address to receive flowers` checkbox
      }
    }
    ```
- Success response:
  ```JSON
    {
      "http_status_code": 200,
      "message": "The death e-notice of <AH's full name> has been successfully confirmed and authorised"
    }
  ```
- Failure response: there are 5 failure states:
** The Account Holder email is not found in system.
** The current user is not a trusted friend(accepted) of the AH user.
** The death notice is not initiated yet.
** The death notice is already authorised.
** The death e-notice was initiated by the current user, this action only can do by another trusted friend.

#### Send funeral details
- Notes:
  - Available if the status of AH user' death e-notice is `authorised`
  - Use in `Trusted Friend` tab
- Request:
  - Path: `/api/v1/user/death_notice/send_funeral_details`
  - Method: `POST`
  - Header:
    ```
    X-User-Access-Token: <User's ID>
    ```
  - Body:
    ```JSON
    {
      "funeral_details": {
        "email": "someone@example.com", // required; the email address of Account Holder
        "time": "yyyy-mm-dd hh:mm:ss", // required;
        "location": "123 Main Street", // required; maxlength is 500
        "details": "Curabitur non nulla sit amet nisl tempus ..." //requied; maxlength is 10000
      }
    }
    ```
- Success response:
  ```JSON
    {
      "http_status_code": 200,
      "message": "The funeral details of <AH's full name> has been sent successfully"
    }
  ```
- Failure response: there are 4 failure states:
** The Account Holder email is not found in system.
** The current user is not a trusted friend(accepted) of the AH user.
** The death notice is not authorised.
** The funeral details is already sent

### eWake
#### Get eWake info and messages
- Request:
  - Path: `/api/v1/user/ewake/{account_holder_id}`
  - Method: `GET`
  - Body: empty
- Response:
  ```JSON
  {
    "ewake": {
      "share_url": "http://example.com/ewake/john-doe", // URL to share for view eWake
      "account_holder": { // This is Account Holder of eWake
        "id": 1,
        "first_name": "John",
        "last_name": "Doe",
        "middle_name": null,
        "avatar": null
      },
      "final_message": { // Null if AH did not setup Notice of passing or hidden by administrator
        "id": 5,
        "content": "Sed porttitor lectus nibh",
        "created_at": "2016-07-21T10:54:26.000Z",
        "author": {
          "id": 15,
          "first_name": "Thanh",
          "last_name": "Luan",
          "middle_name": null,
          "full_name": "Thanh Luan"
        }
      },
      "ewake_messages": [ // Collection of all messages in eWake
        {
          "id": 1,
          "content": "Praesent sapien massa convallis a pellent ...",
          "created_at": "2016-01-27T02:48:45.000Z",
          "attachments": [ // array of photos attached in the messages
            {
              "id": 1,
              "content_type": "image/png",
              "origin": "http://example.com/xxx.png",
              "thumb": "http://example.com/yyy.png"
            }
          ],
          "author": {
            "id": 2,
            "first_name": "Peter",
            "last_name": null,
            "middle_name": null,
            "avatar": { // null if not set
              "origin": "http://example.com/xxx.png",
              "thumb": "http://example.com/yyy.png"
            }
          }
        },
        {
          "id": 1,
          "content": "Praesent sapien massa, convallis a pellent",
          "created_at": "2016-01-27T02:48:45.000Z",
          "attachments": [], // empty mean no photos attached
          "author": {
            "id": 2,
            "first_name": "Alan",
            "last_name": null,
            "middle_name": null,
            "avatar": null
          }
        }
      ]
    }
  }
  ```

#### Post a message
- Request:
  - Path: `/api/v1/user/ewake/{account_holder_id}/messages`
  - Method: `POST`
  - Body:
  ```JSON
  {
    "ewake_message": {
      "content": "Praesent sapien massa, convallis a pellentesque",
      "attachments": [] // The field name is "attachments[]"
    }
  }
  ```
- Response:
  ```JSON
  {
    "http_status_code": 200,
    "message": "Your message has been posted to eWake",
    "ewake_message": {
      "id": 1,
      "content": "Praesent sapien massa, convallis a pellentesque",
      "created_at": "2016-01-27T16:33:23.801Z",
      "author": {
        "id": 1,
        "first_name": "John",
        "last_name": "Doe",
        "middle_name": null,
      },
      "attachments": [
        {
          "id": 1,
          "content_type": "image/png",
          "origin": "http://example.com/xxx.png",
          "thumb": "http://example.com/yyy.png"
        }
      ]
    }
  }
  ```

#### Update a message
- Request:
  - Path: `/api/v1/user/ewake/{account_holder_id}/messages/{message_id}`
  - Method: `PUT`
  - Body:
  ```JSON
  {
    "ewake_message": {
      "content": "Praesent sapien massa, convallis a pellentesque"
    }
  }
  ```
- Response:
  ```JSON
  {
    "http_status_code": 200,
    "message": "Your message has been updated",
    "ewake_message": {
      "id": 1,
      "content": "Praesent sapien massa, convallis a pellentesque",
      "created_at": "2016-01-27T16:33:23.801Z",
      "author": {
        "id": 1,
        "first_name": "John",
        "last_name": "Doe",
        "middle_name": null,
      },
      "attachments": [
        {
          "id": 1,
          "content_type": "image/png",
          "origin": "http://example.com/xxx.png",
          "thumb": "http://example.com/yyy.png"
        }
      ]
    }
  }
  ```

#### Delete a message
- Description: Use to delete own posted message
- Request:
  - Path: `/api/v1/user/ewake/{account_holder_id}/messages/{message_id}`
  - Method: `DELETE`
  - Body: empty
- Response:
  ```JSON
  {
    "http_status_code": 200,
    "message": "Your message has been removed from the eWake"
  }
  ```

#### Report a message
- Description: Use to report a posted message of someone. After report successfully, an support request thread will be created with category "Report Inappropriate".
- Request:
  - Path: `/api/v1/user/ewake/{account_holder_id}/messages/{message_id}/report`
  - Method: `POST`
  - Body:
  ```JSON
  {
    "comment": "Sed porttitor lectus nibh. Mauris blandit ..."// Optional; Maxlength is 500
  }
  ```
- Response:
  ```JSON
  {
    "http_status_code": 200,
    "message": "Thanks for for report"
  }
  ```

### Joining invitation
#### Get saved templates
- Notes:
  - Use for get user-defined templates
- Request:
  - Path: `/api/v1/user/join_invitation/templates`
  - Method: `GET`
  - Body: empty
- Response
  ```JSON
  {
    "join_invitation_templates": [
      {
        "id": 1,
        "title": "Personal invitaion template xx",
        "content": "Omnis delectus  ... ",
        "type": "personal"
      },
      {
        "id": 2,
        "title": "Standard invitaion template xx",
        "content": "Est et voluptas ...",
        "type": "standard"
      },
      ...
    ]
  }
  ```

#### Save template
- Notes:
  - Save user-defined templates
  - If `title` is already exists in user's templates, the exist template will be updated
- Request:
  - Path: `/api/v1/user/join_invitation/templates`
  - Method: `POST`
  - Body:
  ```JSON
  {
    "join_invitation_template": {
      "source_id": 1, // Id of reference template
      "title": "Title for template", // uniqueness per user, non case-sensitive
      "type": "standard|personal",
      "content": "Est et voluptas ...",
      "override": "yes|no", "1|0", true|false // To update the source template, only make sense incase current user is author of the source template, use to rename or update template content
    }
  }
  ```
- Response
  ```JSON
  {
    "http_status_code": 200,
    "message": "Your template has been save",
    "join_invitation_template": {
      "id": 1,
      "title": "Title for template",
      "content": "Est et voluptas ...",
      "type": "personal"
    }
  }
#### Get list invites
- Notes:
  - Use for listing invitations in `Invite` tab
  - Sortable fields: `first_name`, `last_name`, `full_name`, `email`, `status`, `type`, `account_creating_status`, `sent_at`, `created_at`
- Request:
  - Path: `/api/v1/user/join_invitation/list`
  - Method: `GET`
  - Body:
  ```JSON
  {
    "page": 1, // Optional, default is 1
    "per": 10, // Optional, default is 10
    "sort": "<sort field>",
    "order": "asc|desc"
  }
  ```
- Response
  ```JSON
  {
    "users": [
      {
        "id": 1,
        "email": "someone@exampl.com",
        "first_name": "John",
        "last_name": "Doe",
        "middle_name": "",
        "full_name": "",
        "invitee_joined": "true|false",
        "status": "pending", // Recipient acceptance status
        "type": "<null>|standard|personal",
        "content": null, // Email body sent
        "sent_at": null,
        "mail_delivery_status": "pending|failure|success",
        "account_created": "no", // Or 'yes', 'n_a'
        "created_at": "2016-01-04T17:11:26.000Z"
      },
      ...
    ],
    "total": 3,
    "per": 10,
    "sort": "<sort field>",
    "order": "asc|desc"
    "account_created_count": 2 // the number of invitations which the invitee has created account
  }
  ```

#### Add One
- Notes:
  - Very similar to Add One friendship API
  - Use for add one invite to the list.
- Request:
  - Path: `/api/v1/user/join_invitation/add_one`
  - Method: `POST`
  - Body:
  ```JSON
  {
    "user": {
      "email": "someone@example.com", // Required
      "first_name": "John", // Required
      "last_name": "Doe", // Required
      "middle_name": ""
    },
    "type": "standard|personal", // Required
    "template_id": 1, // Required
    "content": "Sed porttitor lectus nibh ...", // Required, the content of email
    "send": "yes|no", "1|0", true|false
  }
  ```

#### Update invite
- Notes:
  - The same with Add One API but use different HTTP method
  - Use for update a created invite.
- Request:
  - Path: `/api/v1/user/join_invitation/add_one`
  - Method: `PUT`
  - Body:
  ```JSON
  {
    "user": {
      "email": "someone@example.com", // Required & Read-Only
      "first_name": "John", // Required
      "last_name": "Doe", // Required
      "middle_name": ""
    },
    "type": "standard|personal", // Required
    "template_id": 1, // Required
    "content": "Sed porttitor lectus nibh ...", // Required, the content of email
    "send": "yes|no", "1|0", true|false
  }
  ```

#### Remove invitation
- Notes: Use to remove an invitation. An error will be thrown if the nvitee joined.
- Request:
  - Path: `/api/v1/user/join_invitation/remove_invitation`
  - Method: `POST`
  - Header:
    ```
    X-User-Access-Token: <User's ID>
    ```
  - Body:
    ```JSON
    {
      "user": {
        "email": "inviteemail@example.com", // required
      }
    }
    ```

- Success response:
  ```JSON
  {
    "message": "The invitation has been removed",
    "http_status_code": 200
  }
  ```

#### Add Many
- Notes:
  - Very similar to Add Many friendship API
  - Use for add many invite to the list.
  - Email will be send if params "send" == "yes"
- Request:
  - Path: `/api/v1/user/join_invitation/add_many`
  - Method: `POST`
  - Body:
  ```JSON
  {
    "users": [
      {
        "email": "someone@example.com", // Required
        "first_name": "John", // Required
        "last_name": "Doe", // Required
        "middle_name": ""
      },
      {
        "email": "someoneelse@example.com",
        "first_name": "John",
        "last_name": "Doe",
        "middle_name": ""
      }
    ],
    "type": "standard|personal", // Required
    "template_id": 1, // Required
    "content": "Sed porttitor lectus nibh ...", // Required, the content of email,
    "send": "yes|no", "1|0", true|false
  }
  ```
- Success response:
  ```JSON
  {
    "message": "Invitations created successfully",
    "http_status_code": 200,
    "success_invitations": [
      {
        "email": "someone@exampl.com",
        "first_name": "John",
        "last_name": "Doe",
        "full_name": "",
        "invitee_joined": "true|false",
        "status": "pending", // Recipient acceptance status
        "type": "<null>|standard|personal",
        "content": null, // Email body sent
        "sent_at": null,
        "mail_delivery_status": "pending|failure|success",
        "account_created": "no", // Or 'yes', 'n_a'
        "created_at": "2016-01-04T17:11:26.000Z"
      }
    ],
    "failure_invitations": [
      {
        "email": "someone",
        "first_name": "John",
        "last_name": "Doe",
        "middle_name": "",
        "error": "Email is invalid"
      },
      {
        "email": "someone@example.com",
        "first_name": "John",
        "last_name": "Doe",
        "middle_name": "",
        "error": "Email is already invited"
      }
    ]
  }
  ```

#### Get unregistered list
- Notes:
  - This API will return only invitees who not joined to the system.
  - Use for `Send Invitation` functionality to lit invites for select to sending email.
  - Return all the records without pagination.
- Request:
  - Path: `/api/v1/user/join_invitation/unregistered_list`
  - Method: `GET`
  - Body: empty
- Response
  ```JSON
  {
    "users": [
      {
        "email": "someone@example.com",
        "first_name": "John",
        "last_name": "Doe",
        "status": "pending", // Recipient acceptance status
        "type": "standard|personal",
        "content": null, // Email body sent
        "sent_at": null,
        "account_created": "no", // Or 'yes', 'n_a'
        "mail_delivery_status": "pending|failure|success",
        "created_at": "2016-01-04T17:11:26.000Z"
      },
      ...
    ]
  }
  ```

#### Send invitation emails
- Notes:
  - Use in `Send Invitation` page
- Request:
  - Path: `/api/v1/user/join_invitation/send`
  - Method: `POST`
  - Body:
  ```JSON
  {
    "emails": [ // Array emails of users
      "someone@example.com", "someonelse@example.com"
    ]
  }
  ```

#### Resend invitation email
- Notes:
  - Use in `User details` popup after clicking on a invite from `Invite` tab
- Request:
  - Path: `/api/v1/user/join_invitation/resend_invitation`
  - Method: `POST`
  - Body:
  ```JSON
  {
    "email": "someone@example.com"
  }
  ```

#### Get pending lifelinks(personal invitation)
- Notes:
  - Sortable fields: `inviter_first_name`, `inviter_last_name`, `inviter_full_name`, `inviter_email`, `sent_at`
- Request:
  - Path: `/api/v1/user/join_invitation/pending_lifelinks`
  - Method: `GET`
  - Body:
  ```JSON
  {
    "users": [
      {
        "first_name": "Join", // First name of inviter
        "last_name": "Smith",
        "middle_name": "",
        "email": "email1@example.com",
        "invite_token": "xxx",
        "sent_at": "2016-01-13T07:55:30.000Z"
      },
      {
        "first_name": "Someone",
        "last_name": "Else",
        "middle_name": "",
        "email": "email2@example.com",
        "invite_token": "xxx",
        "sent_at": "2016-01-13T07:55:30.000Z"
      }
    ]
  }
  ```

#### Accept pending lifelinks
- Request:
  - Path: `/api/v1/user/join_invitation/accept_lifelinks`
  - Method: `POST`
  - Body:
  ```JSON
  {
    "invite_tokens": [
      "xxx",
      "yyy",
      "zzz"
    ]
  }
  ```

#### Reject pending lifelinks
- Note: use for reject Personal invites
- Request:
  - Path: `/api/v1/user/join_invitation/reject_lifelinks`
  - Method: `POST`
  - Body:
  ```JSON
  {
    "invite_tokens": [
      "xxx",
      "yyy",
      "zzz"
    ]
  }
  ```

#### Reject a invite
- Request:
  - Path: `/api/v1/user/join_invitation/reject_invitation`
  - Method: `POST`
  - Body:
  ```JSON
  {
    "invite_token": "xxx"
  }
  ```

### Subscription
#### Info
- Request:
  - Path: `/api/v1/user/subscription/info`
  - Method: `GET`
  - Body: empty
- Response:
  ```JSON
  {
    "http_status_code": 200,
    "subscription": {
      "purchase_method": "free|coupon|payment", // `free` mean use has registed in free period, `coupon` mean user registed by coupon code, `payment` mean use use Paypal or credit card
      "payment_option": "one_off|annual_recurring",
      "payment_method": { // available if `purchase_method` is `payment`
        "type": "paypal", // or 'credit_card' see sample on `transaction`
        "details": {
          "email": "jane.doe@example.com",
          "image_url": "https://assets.braintreegateway.com/payment_method_logo/paypal.png"
        }
      },
      "billing_period_start_date": null, // Current billing period start date
      "billing_period_end_date": null, // Current billing period end date
      "next_billing_date": null, // Next due
      "last_paid_at": null, // Last transaction time
      "status": "active|pending|past_due|unpaid|canceled", // pending mean user just registered as AH but not actived yet.
      "created_at": "2016-04-06T16:48:14.000Z",
      "transactions": [
        {
          "id": "5snd0cx6",
          "amount": 5,
          "description": "Active Account Holder subscription",
          "payment_method": {
            "type": "credit_card",
            "details": {
              "card_type": "Visa",
              "cardholder_name": "John Doe",
              "expiration_month": "12",
              "expiration_year": "2020",
              "image_url": "https://assets.braintreegateway.com/payment_method_logo/visa.png",
              "bin": "401288", // First 6 numbers
              "last_4": "1881" // Last 4 numbers
            }
          },
          "status": "settling",
          "receipt_url": "http://example.com/not-implemented-yet"
        }
      ]
    }
  }
  ```

#### Upgrade to Account Holder
- Request:
  - Path: `/api/v1/user/subscription/upgrade_account_holder`
  - Method: `POST`
  - Body:
  ```JSON
  {
    "country": "US", // Required
    "dob": "1990-01-01", // Required
    "gender": "male|female|other" // Required
    "contact_number": "xxx", // Optional
    "coupon_code": "xxx"
  }
  ```
- Response:
  ```JSON
  {
    "http_status_code": 200,
    "message": "Your account has successfully upgraded to Account Holder"
  }
  ```

#### Update payment method
- Notes:
  - Available only for Annual Recurring subscription
- Request:
  - Path: `/api/v1/user/subscription/update_payment_method`
  - Method: `POST`
  - Body:
  ```JSON
    {
      "payment_method_nonce": "xxx", // required
    }
  ```
- Response:
  ```JSON
  {
    "http_status_code": 200,
    "message": "Your payment method has successfully updated"
  }
  ```

### Support Requests
#### List support requests
- Notes:
  - Use for getting list of requests requested by the user
  - Return all the records without pagination.
  - Sortable fields: `id`, `subject`, `category_id`, `messages_count`, `last_message_at`, `created_at`
- Request:
  - Path: `/api/v1/help/support_requests`
  - Method: `GET`
  - Body: empty

- Response:
  ```JSON
  {
    "http_status_code": 200,
    "support_requests": [
      {
        "id": 2,
        "subject": "Request 2nd",
        "details": "Cannot change password",
        "category": {
          "id": 1,
          "name": "Tech Support"
        },
        "created_at": "2016-01-13T07:55:30.000Z",
        "last_message_at": "2016-01-13T07:55:30.000Z", // This is created time of thread if no message posted
        "messages_count": 0,
        "unread_messages_count": 0,
        "status": "open|closed"
      },
      {
        "id": 1,
        "subject": "Request 1st",
        "details": "Could not add trusted friend",
        "category": {
          "id": 1,
          "name": "Tech Support"
        },
        "created_at": "2016-01-13T07:21:13.000Z",
        "last_message_at": "2016-01-13T07:21:13.000Z",
        "messages_count": 0,
        "unread_messages_count": 0,
        "status": "open|closed"
      }
    ],
    "sort": "id",
    "order": "desc"
  }
  ```

#### Create a support request
- Request:
  - Path: `/api/v1/help/support_requests`
  - Method: `POST`
  - Body:
  ```JSON
  {
    "support_request": {
      "subject": "Ded porttitor lectus nibh", // Required
      "category_id": 1, // Required
      "details": "Sed porttitor lectus nibh ...", // Required
    }
  }
  ```
- Response:
  ```JSON
  {
    "http_status_code": 200,
    "message": "Support request created successfully"
  }
  ```

#### Get messages
- Notes: Use for displaying messages in a support request thread
- Request:
  - Path: `/api/v1/help/support_requests/{id}/messages`
  - Method: `GET`
  - Body: empty
- Response:
  ```JSON
  {
    "http_status_code": 200,
    "support_request_messages": [
      {
        "content": "Thank for your support aliquet ",
        "is_from_user": true,
        "created_at": "2016-01-13T18:12:07.000Z"
      },
      {
        "content": "I will check it soon.",
        "is_from_user": false, // It's reply from admin
        "created_at": "2016-01-13T10:12:07.000Z"
      }
    ]
  }
  ```

#### Post message
- Request:
  - Path: `/api/v1/help/support_requests/{id}/messages`
  - Method: `POST`
  - Body:
  ```JSON
    {
      "support_request_message": {
        "content": "Sed porttitor lectus nibh. Curabitur ..." // Required
      }
    }
  ```
- Response:
  ```JSON
  {
    "http_status_code": 200,
    "message": "Your message has been sent successfully"
  }
  ```

#### Close support request
- Request:
  - Path: `/api/v1/help/support_requests/{id}/close`
  - Method: `POST`
  - Body: empty
- Response:
  ```JSON
  {
    "http_status_code": 200,
    "message": "Your support thread has been successfully closed"
  }
  ```

### Shared data
- Notes: all these endpoints does not require to authorize user
#### Get all shared data
- Notes:
  - Use to get the data likes: pre-registed templates, categories ... which use frequently in system
  - List of returning data:
    - Support Request Categories
    - Death E-Notice Templates
    - Join Invitation Templates
    - FAQ
    - Term and conditions
    - Free period availability
- Request:
  - Path: `/api/v1/shared/dump`
  - Method: 'GET'
  - Body: empty
- Response:
  ```JSON
  {
    "http_status_code": 200,
    "support_request_categories": [
      {
        "id": 1,
        "name": "Tech Support"
      },
      {
        "id": 2,
        "name": "Pricing"
      },
      {
        "id": 3,
        "name": "Other"
      }
    ],
    "death_notice_templates": [
      {
        "id": 1,
        "title": "Death E-Notice template xx",
        "content": "Voluptatem ab facilis ...",
        "default": true // Use when the account holder's death notice is not setup yet
      },
      {
        "id": 2,
        "title": "Death E-Notice template yy",
        "content": "Dolor fugiat pariatur ...",
        "default": false
      }
    ],
    "join_invitation_templates": [
      {
        "id": 1,
        "title": "Personal invitaion template xx",
        "content": "Omnis delectus  ... ",
        "type": "personal"
      },
      {
        "id": 2,
        "title": "Standard invitaion template xx",
        "content": "Est et voluptas ...",
        "type": "standard"
      }
    ],
    "terms_and_conditions": "<content in HTML>",
    "frequently_asked_questions": "<content in HTML>",
    "free_period_availability": true/false,
    "maximum_number_of_contacts_uploadable_once": 10
  }
  ```

### Payment

#### Get client token
- Request:
  - Path: `/api/v1/payment/token`
  - Method: 'GET'
  - Body: empty
- Response:
  ```JSON
  {
    "http_status_code": 200,
    "token": "xxx"
  }
