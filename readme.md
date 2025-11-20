---
title: OAuth 2.0 Integration Guide
layout: home
---
This document describes how to integrate with our OAuth 2.0 API for authentication and user data retrieval. It covers the full flow: authorization, token exchange, and user information retrieval. This guide is framework-agnostic and can be applied in web, mobile, or server-side applications.

## Base URLs

**Development Environment:**

```
https://dev.losgehts.at/
```

**Production Environment:**

```
https://ident.losgehts.at/
```

## 1. Overview

OAuth 2.0 allows your application to access user data securely without handling credentials directly. The typical flow involves:

1. Redirecting the user to the authorization endpoint.
2. Receiving an authorization code.
3. Exchanging the code for an access token.
4. Using the token to retrieve user information.

## 2. Authorization Endpoint

**URL:**

```
GET /oauth/authorize
```

### Query Parameters

| Parameter  | Required | Description |
|------------|----------|-------------|
| client_id  | Yes      | The unique client identifier of your application. |
| state      | Yes      | A unique string used to maintain state between the request and callback (e.g., session ID, affiliate tag, or encrypted UserID). Returned in the callback to help prevent CSRF attacks. |
| scope      | Yes      | A comma-separated list of scopes defining the permissions being requested. Supported scopes include: `signup`, `kyc`, and `sof`. |
| locale     | No       | A two-letter country/language code used for localization (e.g., `de`, `at`, `us`). Primarily determines the UI language. |
| cc         | No       | Enforces a specific country context in a multi-country setup (e.g., `AT`). If not provided, the country will be auto-detected based on the user's geolocation. |


### Example

**Development:**

```
https://dev.losgehts.at/oauth/authorize?client_id=40&state=abc123&scope=signup&locale=de
```

**Production:**

```
https://ident.losgehts.at/oauth/authorize?client_id=40&state=abc123&scope=signup&locale=de
```

**Security Note:** Always validate the `state` parameter in the callback.

## 3. Callback Handling

After authorization, the user is redirected to your callback URL with:

* `code` — The authorization code
* `state` — The same value you supplied

**Example callback URL:**

```
https://yourapp.com/callback?code=AUTH_CODE&state=abc123
```

## 4. Token Exchange

Exchange the authorization code for an access token.

**Endpoint:**

```
POST /oauth/token
Content-Type: application/json
```

### Request Body

```json
{
  "code": "AUTHORIZATION_CODE",
  "state": "STATE_VALUE",
  "client_id": "CLIENT_ID",
  "client_secret": "CLIENT_SECRET",
  "grant_type": "authorization_code"
}
```

### Response Example

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "xyz123"
}
```

**Important:** Store the access token securely and never expose it in client-side code.

## 5. User Information Endpoint

Use the access token to retrieve user information.

**Endpoint:**

```
POST /oauth/userinfo
Content-Type: application/json
Authorization: Bearer ACCESS_TOKEN
```

### Request Body

```json
{
  "token": "ACCESS_TOKEN",
  "client_id": "YOUR_CLIENT_ID",
  "client_secret": "YOUR_CLIENT_SECRET"
}
```

### Response Example

```json
{
  "success": true,
  "verificationId": "123456",
  "clientId": "40",
  "clientName": "My App",
  "verificationStatus": 1,
  "email": "user@example.com",
  "emailConfirmed": true,
  "firstName": "John",
  "firstNameVerified": true,
  "lastName": "Doe",
  "lastNameVerified": true,
  "fullName": "John Doe",
  "dateOfBirth": "1990-01-01",
  "gender": "male",
  "nationality": "US",
  "street": "123 Main St",
  "houseNumber": "1A",
  "zipCode": "12345",
  "town": "Sample City",
  "country": "US",
}
```

### Verification Status Values

The `verificationStatus` field can contain the following values:

| Status ID | Status  | Description                                           |
| --------- | ------- | ----------------------------------------------------- |
| 0         | Pending | Verification process is still in progress             |
| 1         | Full    | Complete verification has been successfully completed |
| 2         | Passive | Passive verification has been completed               |
| 3         | Failed  | Verification process has failed                       |

### Complete Response Fields Reference

The following table shows all possible fields that may be returned in the userinfo response:

| Field                 | Type    | Description                                  | Format |
|-----------------------|---------|----------------------------------------------|--------|
| VerificationId        | Integer | Primary identifier for the verification record |  |
| Email                 | Text    | User's email address                          |  |
| EmailConfirmed        | Boolean | Whether email has been confirmed             |  |
| VerificationStatusId  | Integer | Status of the verification process           |  |
| Password              | Text    | User's password (encrypted)                  |  |
| FirstName             | Text    | User's first name                             |  |
| FirstNameVerified     | Boolean | Whether first name has been verified         |  |
| LastName              | Text    | User's last name                              |  |
| LastNameVerified      | Boolean | Whether last name has been verified          |  |
| DateOfBirth           | Text    | User's date of birth                          | ISO 8601 calendar date format (YYYY-MM-DD) |
| DateOfBirthVerified   | Boolean | Whether date of birth has been verified      |  |
| Gender                | Text    | User's gender                                 | MALE, FEMALE, OTHER |
| GenderVerified        | Boolean | Whether gender has been verified             |  |
| Nationality           | Text    | User's nationality                            | ISO 3166-1 alpha-2 (two uppercase letters) |
| NationalityVerified   | Boolean | Whether nationality has been verified        |  |
| ZipCode               | Text    | User's postal/zip code                        |  |
| ZipCodeVerified       | Boolean | Whether zip code has been verified           |  |
| Town                  | Text    | User's town/city                              |  |
| TownVerified          | Boolean | Whether town has been verified               |  |
| Street                | Text    | User's street name                            |  |
| StreetVerified        | Boolean | Whether street has been verified             |  |
| Country               | Text    | User's country                                | ISO 3166-1 alpha-2 (two uppercase letters) |
| CountryVerified       | Boolean | Whether country has been verified            |  |
| PhoneNumber           | Text    | User's phone number                           |  E.164 standard |
| PhoneNumberVerified   | Boolean | Whether phone number has been verified       |  |
| Lang                  | Text    | User's language preference                    | ISO 3166-1 alpha-2 (two uppercase letters) |


**Notes:**

* Fields ending in 'Verified' indicate that the value has been confirmed.
* Some fields are only returned if the requested scopes include them.

## 6. Handling the `state` Parameter

1. Generate a unique state per OAuth request.
2. Include it in the `/oauth/authorize` redirect.
3. Validate that the returned state matches the original before exchanging the code.

This protects against CSRF attacks.

## 7. Typical OAuth Flow

1. **User Authorization** → Redirect to `/oauth/authorize`
2. **Receive Code** → Extract code and state from callback
3. **Token Exchange** → POST to `/oauth/token`
4. **Fetch User Info** → POST to `/oauth/userinfo`
5. **Store & Use Data** → Store tokens securely

## 8. Security Considerations

* Always validate the state parameter
* Use HTTPS for all OAuth traffic
* Never expose `client_secret` in browsers or JS
* Store tokens securely (server-side or encrypted local storage for mobile)
