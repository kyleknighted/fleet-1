# REST API
- [Overview](#overview)
  - [fleetctl](#fleetctl)
  - [Current API](#current-api)
- [Authentication](#authentication)
  - [Log in](#log-in)
  - [Log out](#log-out)
  - [Forgot password](#forgot-password)
  - [Change password](#change-password)
  - [Me](#me)
  - [SSO config](#sso-config)
  - [Initiate SSO](#initiate-sso)
- [Hosts](#hosts)
  - [List hosts](#list-hosts)
- [Users](#users)
  - [List all users](#list-all-users)
  - [Create a user account with an invitation](#create-a-user-account-with-an-invitation)
  - [Create a user account without an invitation](#create-a-user-account-without-an-invitation)
  - [Get user information](#get-user-information)

## Overview

Fleet is powered by a Go API server which serves three types of endpoints:

- Endpoints starting with `/api/v1/osquery/` are osquery TLS server API endpoints. All of these endpoints are used for talking to osqueryd agents and that's it.
- Endpoints starting with `/api/v1/kolide/` are endpoints to interact with the Fleet data model (packs, queries, scheduled queries, labels, hosts, etc) as well as application endpoints (configuring settings, logging in, session management, etc).
- All other endpoints are served the React single page application bundle. The React app uses React Router to determine whether or not the URI is a valid route and what to do.

Only osquery agents should interact with the osquery API, but we'd like to support the eventual use of the Fleet API extensively. The API is not very well documented at all right now, but we have plans to:

- Generate and publish detailed documentation via a tool built using [test2doc](https://github.com/adams-sarah/test2doc) (or similar).
- Release a JavaScript Fleet API client library (which would be derived from the [current](https://github.com/fleetdm/fleet/blob/master/frontend/kolide/index.js) JavaScript API client).
- Commit to a stable, standardized API format.

### fleetctl

Many of the operations that a user may wish to perform with an API are currently best performed via the [fleetctl](./2-fleetctl-CLI.md) tooling. These CLI tools allow updating of the osquery configuration entities, as well as performing live queries.

### Current API

The general idea with the current API is that there are many entities throughout the Fleet application, such as:

- Queries
- Packs
- Labels
- Hosts

Each set of objects follows a similar REST access pattern.

- You can `GET /api/v1/kolide/packs` to get all packs
- You can `GET /api/v1/kolide/packs/1` to get a specific pack.
- You can `DELETE /api/v1/kolide/packs/1` to delete a specific pack.
- You can `POST /api/v1/kolide/packs` (with a valid body) to create a new pack.
- You can `PATCH /api/v1/kolide/packs/1` (with a valid body) to modify a specific pack.

Queries, packs, scheduled queries, labels, invites, users, sessions all behave this way. Some objects, like invites, have additional HTTP methods for additional functionality. Some objects, such as scheduled queries, are merely a relationship between two other objects (in this case, a query and a pack) with some details attached.

All of these objects are put together and distributed to the appropriate osquery agents at the appropriate time. At this time, the best source of truth for the API is the [HTTP handler file](https://github.com/fleetdm/fleet/blob/master/server/service/handler.go) in the Go application. The REST API is exposed via a transport layer on top of an RPC service which is implemented using a micro-service library called [Go Kit](https://github.com/go-kit/kit). If using the Fleet API is important to you right now, being familiar with Go Kit would definitely be helpful.



## Authentication

Making authenticated requests to the Fleet server requires that you are granted permission to access data. The Fleet Authentication API enables you to receive an authorization token.

All Fleet API requests are authenticated unless noted in the documentation. This means that almost all Fleet API requests will require sending the API token in the request header.

The typical steps to making an authenticated API request are outlined below.

First, utilize the `/login` endpoint to receive an API token. For SSO users, username/password login is disabled and the API token can be retrieved from the "Settings" page in the UI.

`POST /api/v1/kolide/login`

Request body

```
{
  "username": "janedoe@example.com",
  "passsword": "VArCjNW7CfsxGp67"
}
```

Default response

`Status: 200`

```
{
  "user": {
    "created_at": "2020-11-13T22:57:12Z",
    "updated_at": "2020-11-13T22:57:12Z",
    "id": 1,
    "username": "jane",
    "name": "",
    "email": "janedoe@example.com",
    "admin": true,
    "enabled": true,
    "force_password_reset": false,
    "gravatar_url": "",
    "sso_enabled": false
  },
  "token": "{your token}"
}
```

Then, use the token returned from the `/login` endpoint to authenticate further API requests. The example below utilizes the `/hosts` endpoint.

`GET /api/v1/kolide/hosts`

Request header

```
Authorization: Bearer <your token>
```

Default response

`Status: 200`

```
{
  "hosts": [
    {
      "created_at": "2020-11-05T05:09:44Z",
      "updated_at": "2020-11-05T06:03:39Z",
      "id": 1,
      "detail_updated_at": "2020-11-05T05:09:45Z",
      "label_updated_at": "2020-11-05T05:14:51Z",
      "seen_time": "2020-11-05T06:03:39Z",
      "hostname": "2ceca32fe484",
      "uuid": "392547dc-0000-0000-a87a-d701ff75bc65",
      "platform": "centos",
      "osquery_version": "2.7.0",
      "os_version": "CentOS Linux 7",
      "build": "",
      "platform_like": "rhel fedora",
      "code_name": "",
      "uptime": 8305000000000,
      "memory": 2084032512,
      "cpu_type": "6",
      "cpu_subtype": "142",
      "cpu_brand": "Intel(R) Core(TM) i5-8279U CPU @ 2.40GHz",
      "cpu_physical_cores": 4,
      "cpu_logical_cores": 4,
      "hardware_vendor": "",
      "hardware_model": "",
      "hardware_version": "",
      "hardware_serial": "",
      "computer_name": "2ceca32fe484",
      "primary_ip": "",
      "primary_mac": "",
      "distributed_interval": 10,
      "config_tls_refresh": 10,
      "logger_tls_period": 8,
      "additional": {},
      "enroll_secret_name": "default",
      "status": "offline",
      "display_text": "2ceca32fe484"
    },
    {
      "created_at": "2020-11-05T05:09:44Z",
      "updated_at": "2020-11-05T06:03:39Z",
      "id": 2,
      "detail_updated_at": "2020-11-05T05:09:45Z",
      "label_updated_at": "2020-11-05T05:14:52Z",
      "seen_time": "2020-11-05T06:03:40Z",
      "hostname": "4cc885c20110",
      "uuid": "392547dc-0000-0000-a87a-d701ff75bc65",
      "platform": "centos",
      "osquery_version": "2.7.0",
      "os_version": "CentOS 6.8.0",
      "build": "",
      "platform_like": "rhel",
      "code_name": "",
      "uptime": 8305000000000,
      "memory": 2084032512,
      "cpu_type": "6",
      "cpu_subtype": "142",
      "cpu_brand": "Intel(R) Core(TM) i5-8279U CPU @ 2.40GHz",
      "cpu_physical_cores": 4,
      "cpu_logical_cores": 4,
      "hardware_vendor": "",
      "hardware_model": "",
      "hardware_version": "",
      "hardware_serial": "",
      "computer_name": "4cc885c20110",
      "primary_ip": "",
      "primary_mac": "",
      "distributed_interval": 10,
      "config_tls_refresh": 10,
      "logger_tls_period": 8,
      "additional": {},
      "enroll_secret_name": "default",
      "status": "offline",
      "display_text": "4cc885c20110"
    },
  ]
}
```

### Log in

Authenticates the user with the specified credentials. Use the token returned from this endpoint to authenticate further API requests.

`POST /api/v1/kolide/login`

#### Parameters

| Name     | Type   | In   | Description                                   |
| -------- | ------ | ---- | --------------------------------------------- |
| username | string | body | **Required**. The user's email.               |
| password | string | body | **Required**. The user's plain text password. |

#### Example

`POST /api/v1/kolide/login`

##### Request body

```
{
  "username": "janedoe@example.com",
  "passsword": "VArCjNW7CfsxGp67"
}
```

##### Default response

`Status: 200`

```
{
  "user": {
    "created_at": "2020-11-13T22:57:12Z",
    "updated_at": "2020-11-13T22:57:12Z",
    "id": 1,
    "username": "jane",
    "name": "",
    "email": "janedoe@example.com",
    "admin": true,
    "enabled": true,
    "force_password_reset": false,
    "gravatar_url": "",
    "sso_enabled": false
  },
  "token": "{your token}"
}
```

---

### Log out

Logs out the authenticated user.

`POST /api/v1/kolide/logout`

#### Example

`POST /api/v1/kolide/logout`

##### Default response

`Status: 200`

---

### Forgot password

Sends a password reset email to the specified email. Requires that SMTP is configured for your Fleet server.

`POST /api/v1/kolide/forgot_password`

#### Parameters

| Name  | Type   | In   | Description                                                             |
| ----- | ------ | ---- | ----------------------------------------------------------------------- |
| email | string | body | **Required**. The email of the user requesting the reset password link. |

#### Example

`POST /api/v1/kolide/forgot_password`

##### Request body

```
{
  "email": "janedoe@example.com"
}
```

##### Default response

`Status: 200`

##### Unknown error

`Status: 500`

```
{
  "message": "Unknown Error",
  "errors": [
    {
      "name": "base",
      "reason": "email not configured",
    }
  ]
}
```

---

### Change password

`POST /api/v1/kolide/change_password`

Changes the password for the authenticated user.

#### Parameters

| Name         | Type   | In   | Description                            |
| ------------ | ------ | ---- | -------------------------------------- |
| old_password | string | body | **Required**. The user's old password. |
| new_password | string | body | **Required**. The user's new password. |

#### Example

`POST /api/v1/kolide/change_password`

##### Request body

```
{
  "old_password": "VArCjNW7CfsxGp67",
  "new_password": "zGq7mCLA6z4PzArC",
}
```

##### Default response

`Status: 200`

##### Validation failed

`Status: 422 Unprocessable entity`

```
{
  "message": "Validation Failed",
  "errors": [
    {
      "name": "old_password",
      "reason": "old password does not match"
    }
  ]
}
```

---

### Me

Retrieves the user data for the authenticated user.

`POST /api/v1/kolide/me`

#### Example

`POST /api/v1/kolide/me`

##### Default response

`Status: 200`

```
{
  "user": {
    "created_at": "2020-11-13T22:57:12Z",
    "updated_at": "2020-11-16T23:49:41Z",
    "id": 1,
    "username": "jane",
    "name": "",
    "email": "janedoe@example.com",
    "admin": true,
    "enabled": true,
    "force_password_reset": false,
    "gravatar_url": "",
    "sso_enabled": false
  }
}
```

---

### Perform required password reset

Resets the password of the authenticated user. Requires that `force_password_reset` is set to `true` prior to the request.

`POST /api/v1/kolide/perform_require_password_reset`

#### Example

`POST /api/v1/kolide/perform_required_password_reset`

##### Request body

```
{
  "new_password": "sdPz8CV5YhzH47nK"
}
```

##### Default response

`Status: 200`

```
{
  "user": {
    "created_at": "2020-11-13T22:57:12Z",
    "updated_at": "2020-11-17T00:09:23Z",
    "id": 1,
    "username": "jane",
    "name": "",
    "email": "janedoe@example.com",
    "admin": true,
    "enabled": true,
    "force_password_reset": false,
    "gravatar_url": "",
    "sso_enabled": false
  }
}
```

---

### SSO config

Gets the current SSO configuration.

`GET /api/v1/kolide/sso`

#### Example

`GET /api/v1/kolide/sso`

##### Default response

`Status: 200`

```
{
  "settings": {
    "idp_name": "IDP Vendor 1",
    "idp_image_url": "",
    "sso_enabled": false
  }
}
```

---

### Initiate SSO

`POST /api/v1/kolide/sso`

#### Parameters

| Name      | Type   | In   | Description                                                                |
| --------- | ------ | ---- | -------------------------------------------------------------------------- |
| relay_url | string | body | **Required**. The relative url to be navigated to after succesful sign in. |

#### Example

`POST /api/v1/kolide/sso`

##### Request body

```
{
  "relay_url": "/hosts/manage"
}
```

##### Default response

`Status: 200`

##### Unknown error

`Status: 500`

```
{
  "message": "Unknown Error",
  "errors": [
    {
      "name": "base",
      "reason": "InitiateSSO getting metadata: Get \"https://idp.example.org/idp-meta.xml\": dial tcp: lookup idp.example.org on [2001:558:feed::1]:53: no such host"
    }
  ]
}
```

---

## Hosts

### List hosts

`GET /api/v1/kolide/hosts`

#### Parameters

| Name                    | Type    | In    | Description                                                                                                                                                                                                                                                                                    |
| ----------------------- | ------- | ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| page                    | integer | query | Page number of the results to fetch.                                                                                                                                                                                                                                                           |
| per_page                | integer | query | Results per page.                                                                                                                                                                                                                                                                              |
| order_key               | string  | query | What to order results by. Can be any column in the hosts table.                                                                                                                                                                                                                                |
| status                  | string  | query | Indicates the status of the hosts to return. Can either be `new`, `online`, `offline`, or `mia`.                                                                                                                                                                                               |
| additional_info_filters | string  | query | A comma-delimited list of fields to include in each host's additional information object. See [Fleet Configuration Options](https://github.com/fleetdm/fleet/blob/master/docs/1-Using-Fleet/2-fleetctl-CLI.md#fleet-configuration-options) for an example configuration with hosts' additional information. |

#### Example

`GET /api/v1/kolide/hosts?page=0&per_page=100&order_key=host_name`

##### Request query parameters

```
{
  "page": 0,
  "per_page": 100,
  "order_key": "host_name",
}
```

##### Default response

`Status: 200`

```
{
  "hosts": [
    {
      "created_at": "2020-11-05T05:09:44Z",
      "updated_at": "2020-11-05T06:03:39Z",
      "id": 1,
      "detail_updated_at": "2020-11-05T05:09:45Z",
      "label_updated_at": "2020-11-05T05:14:51Z",
      "seen_time": "2020-11-05T06:03:39Z",
      "hostname": "2ceca32fe484",
      "uuid": "392547dc-0000-0000-a87a-d701ff75bc65",
      "platform": "centos",
      "osquery_version": "2.7.0",
      "os_version": "CentOS Linux 7",
      "build": "",
      "platform_like": "rhel fedora",
      "code_name": "",
      "uptime": 8305000000000,
      "memory": 2084032512,
      "cpu_type": "6",
      "cpu_subtype": "142",
      "cpu_brand": "Intel(R) Core(TM) i5-8279U CPU @ 2.40GHz",
      "cpu_physical_cores": 4,
      "cpu_logical_cores": 4,
      "hardware_vendor": "",
      "hardware_model": "",
      "hardware_version": "",
      "hardware_serial": "",
      "computer_name": "2ceca32fe484",
      "primary_ip": "",
      "primary_mac": "",
      "distributed_interval": 10,
      "config_tls_refresh": 10,
      "logger_tls_period": 8,
      "additional": {},
      "enroll_secret_name": "default",
      "status": "offline",
      "display_text": "2ceca32fe484"
    },
    {
      "created_at": "2020-11-05T05:09:44Z",
      "updated_at": "2020-11-05T06:03:39Z",
      "id": 2,
      "detail_updated_at": "2020-11-05T05:09:45Z",
      "label_updated_at": "2020-11-05T05:14:52Z",
      "seen_time": "2020-11-05T06:03:40Z",
      "hostname": "4cc885c20110",
      "uuid": "392547dc-0000-0000-a87a-d701ff75bc65",
      "platform": "centos",
      "osquery_version": "2.7.0",
      "os_version": "CentOS 6.8.0",
      "build": "",
      "platform_like": "rhel",
      "code_name": "",
      "uptime": 8305000000000,
      "memory": 2084032512,
      "cpu_type": "6",
      "cpu_subtype": "142",
      "cpu_brand": "Intel(R) Core(TM) i5-8279U CPU @ 2.40GHz",
      "cpu_physical_cores": 4,
      "cpu_logical_cores": 4,
      "hardware_vendor": "",
      "hardware_model": "",
      "hardware_version": "",
      "hardware_serial": "",
      "computer_name": "4cc885c20110",
      "primary_ip": "",
      "primary_mac": "",
      "distributed_interval": 10,
      "config_tls_refresh": 10,
      "logger_tls_period": 8,
      "additional": {},
      "enroll_secret_name": "default",
      "status": "offline",
      "display_text": "4cc885c20110"
    },
  ]
}
```

---

## Users

The Fleet server exposes a handful of API endpoints that handles common user management operations. All the following endpoints require prior authentication meaning you must first log in successfully before calling any of the endpoints documented below.

### List all users

Returns a list of all enabled users

`GET /api/v1/kolide/users`

#### Parameters

None.

#### Example

`GET /api/v1/kolide/users`

##### Request query parameters

None.

##### Default response

`Status: 200`

```
{
  "users": [
    {
      "created_at": "2020-12-10T03:52:53Z",
      "updated_at": "2020-12-10T03:52:53Z",
      "id": 1,
      "username": "janedoe",
      "name": "",
      "email": "janedoe@example.com",
      "admin": true,
      "enabled": true,
      "force_password_reset": false,
      "gravatar_url": "",
      "sso_enabled": false
    }
  ]
}
```

##### Failed authentication

`Status: 401 Authentication Failed`

```
{
  "message": "Authentication Failed",
  "errors": [
    {
      "name": "base",
      "reason": "username or email and password do not match"
    }
  ]
}
```

### Create a user account with an invitation

Creates a user account after an invited user provides registration information and submits the form.

`POST /api/v1/kolide/users`

#### Parameters

| Name                  | Type   | In   | Description                                                     |
| --------------------- | ------ | ---- | --------------------------------------------------------------- |
| email                 | string | body | **Required**. The email address of the user.                    |
| invite_token          | string | body | **Required**. Token provided to the user in the invitation email. |
| name                  | string | body | The name of the user.                                           |
| username              | string | body | **Required**. The username chosen by the user                   |
| password              | string | body | **Required**. The password chosen by the user.                  |
| password_confirmation | string | body | **Required**. Confirmation of the password chosen by the user.  |

#### Example

`POST /api/v1/kolide/users`

##### Request query parameters

```
{
  "email": "janedoe@example.com",
  "invite_token": "SjdReDNuZW5jd3dCbTJtQTQ5WjJTc2txWWlEcGpiM3c=",
  "name": "janedoe",
  "username": "janedoe",
  "password": "test-123",
  "password_confirmation": "test-123"
}
```

##### Default response

`Status: 200`

```
{
  "user": {
    "created_at": "0001-01-01T00:00:00Z",
    "updated_at": "0001-01-01T00:00:00Z",
    "id": 2,
    "username": "janedoe",
    "name": "janedoe",
    "email": "janedoe@example.com",
    "admin": false,
    "enabled": true,
    "force_password_reset": false,
    "gravatar_url": "",
    "sso_enabled": false
  }
}
```

##### Failed authentication

`Status: 401 Authentication Failed`

```
{
  "message": "Authentication Failed",
  "errors": [
    {
      "name": "base",
      "reason": "username or email and password do not match"
    }
  ]
}
```

##### Expired or used invite code

`Status: 404 Resource Not Found`

```
{
  "message": "Resource Not Found",
  "errors": [
    {
      "name": "base",
      "reason": "Invite with token SjdReDNuZW5jd3dCbTJtQTQ5WjJTc2txWWlEcGpiM3c= was not found in the datastore"
    }
  ]
}
```

##### Validation failed

`Status: 422 Validation Failed`

The same error will be returned whenever one of the required parameters fails the validation.

```
{
  "message": "Validation Failed",
  "errors": [
    {
      "name": "username",
      "reason": "cannot be empty"
    }
  ]
}
```

### Create a user account without an invitation

Creates a user account without requiring an invitation, the user is enabled immediately.

`POST /api/v1/kolide/users/admin`

#### Parameters

| Name       | Type    | In   | Description                                      |
| ---------- | ------- | ---- | ------------------------------------------------ |
| username   | string  | body | **Required**. The user's username.               |
| email      | string  | body | **Required**. The user's email address.          |
| password   | string  | body | **Required**. The user's password.               |
| invited_by | integer | body | **Required**. ID of the admin creating the user. |
| admin      | boolean | body | **Required**. Whether the user has admin privileges. |

#### Example

`POST /api/v1/kolide/users/admin`

##### Request query parameters

```
{
  "username": "janedoe",
  "email": "janedoe@example.com",
  "password": "test-123",
  "invited_by":1,
  "admin":true
}
```

##### Default response

`Status: 200`

```
{
  "user": {
    "created_at": "0001-01-01T00:00:00Z",
    "updated_at": "0001-01-01T00:00:00Z",
    "id": 5,
    "username": "janedoe",
    "name": "",
    "email": "janedoe@example.com",
    "admin": false,
    "enabled": true,
    "force_password_reset": false,
    "gravatar_url": "",
    "sso_enabled": false
  }
}
```

##### Failed authentication

`Status: 401 Authentication Failed`

```
{
  "message": "Authentication Failed",
  "errors": [
    {
      "name": "base",
      "reason": "username or email and password do not match"
    }
  ]
}
```

##### User doesn't exist

`Status: 404 Resource Not Found`

```
{
  "message": "Resource Not Found",
  "errors": [
    {
      "name": "base",
      "reason": "User with id=1 was not found in the datastore"
    }
  ]
}
```

### Get user information

Returns all information about a specific user.

`GET /api/v1/kolide/users/{id}`

#### Parameters

| Name | Type    | In    | Description                  |
| ---- | ------- | ----- | ---------------------------- |
| id   | integer | query | **Required**. The user's id. |

#### Example

`GET /api/v1/kolide/users/2`

##### Request query parameters

```
{
  "id": 1
}
```

##### Default response

`Status: 200`

```
{
  "user": {
    "created_at": "2020-12-10T05:20:25Z",
    "updated_at": "2020-12-10T05:24:27Z",
    "id": 2,
    "username": "janedoe",
    "name": "janedoe",
    "email": "janedoe@example.com",
    "admin": true,
    "enabled": true,
    "force_password_reset": false,
    "gravatar_url": "",
    "sso_enabled": false
  }
}
```

##### Failed authentication

`Status: 401 Authentication Failed`

```
{
  "message": "Authentication Failed",
  "errors": [
    {
      "name": "base",
      "reason": "username or email and password do not match"
    }
  ]
}
```

##### User doesn't exist

`Status: 404 Resource Not Found`

```
{
  "message": "Resource Not Found",
  "errors": [
    {
      "name": "base",
      "reason": "User with id=5 was not found in the datastore"
    }
  ]
}
```

---
