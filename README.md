# OAuth 2.0 Integration Guide

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

| Parameter | Required | Description                                                                      |
| --------- | -------- | -------------------------------------------------------------------------------- |
| client_id | Yes      | Your application's client ID.                                                    |
| state     | Yes      | Unique string (session ID, affiliate tag). Returned in callback to prevent CSRF. |
| scope     | Yes      | Comma-separated list of requested scopes. Available scopes: signup, kyc, sof     |
| locale    | No       | Country code for localization (e.g., de, at, us)                                 |

### Example

**Development:**

```
https://dev.losgehts.at/oauth/authorize?client_id=40&state=abc123&scope=signup,kyc&locale=de
```

**Production:**

```
https://ident.losgehts.at/oauth/authorize?client_id=40&state=abc123&scope=signup,kyc&locale=de
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
  "token": "ACCESS_TOKEN"
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

### Verification Providers

The verification provider IDs correspond to the following services:

| Provider ID | Name                    | Description    |
|-------------|-------------------------|----------------|
| 0           | AT - Bank Ident         |                |
| 1           | AT - ID Austria         |                |
| 2           | Bank #1                 |                |
| 3           | Bank #2                 |                |
| 4           | Open Banking #1         |                |
| 5           | Open Banking #2         |                |
| 6           | AT - Company Register   |                |
| 7           | EU - Company Register   |                |
| 8           | SMS Verification        | OTP confirmed  |
| 9           | N/A                     |                |
| 10          | DE - Carrier Match      |                |

### Complete Response Fields Reference

The following table shows all possible fields that may be returned in the userinfo response:

| Field                                    | Type    | Description                                       |
| ---------------------------------------- | ------- | ------------------------------------------------- |
| VerificationId                           | Integer | Primary identifier for the verification record    |
| CreatedAt                                | Text    | Timestamp when the record was created             |
| ModifiedAt                               | Text    | Timestamp when the record was last modified       |
| ClientId                                 | Integer | Your application's client ID                      |
| VerificationStatusId                     | Integer | Status of the verification process                |
| VerifiedAt                               | Text    | Timestamp when verification was completed         |
| Email                                    | Text    | User's email address                              |
| EmailVerificationProviderId              | Integer | ID of email verification provider                 |
| EmailPassiveVerificationProviderId       | Integer | ID of passive email verification provider         |
| EmailVerified                            | Boolean | Whether email has been verified                   |
| EmailConfirmed                           | Boolean | Whether email has been confirmed                  |
| FirstName                                | Text    | User's first name                                 |
| FirstNameVerificationProviderId          | Integer | ID of first name verification provider            |
| FirstNamePassiveVerificationProviderId   | Integer | ID of passive first name verification provider    |
| FirstNameVerified                        | Boolean | Whether first name has been verified              |
| LastName                                 | Text    | User's last name                                  |
| LastNameVerificationProviderId           | Integer | ID of last name verification provider             |
| LastNamePassiveVerificationProviderId    | Integer | ID of passive last name verification provider     |
| LastNameVerified                         | Boolean | Whether last name has been verified               |
| FullName                                 | Text    | User's full name                                  |
| FullNameVerificationProviderId           | Integer | ID of full name verification provider             |
| FullNamePassiveVerificationProviderId    | Integer | ID of passive full name verification provider     |
| FullNameVerified                         | Boolean | Whether full name has been verified               |
| MaidenName                               | Text    | User's maiden name                                |
| MaidenNameVerificationProviderId         | Integer | ID of maiden name verification provider           |
| MaidenNamePassiveVerificationProviderId  | Integer | ID of passive maiden name verification provider   |
| MaidenNameVerified                       | Boolean | Whether maiden name has been verified             |
| DateOfBirth                              | Text    | User's date of birth                              |
| DateOfBirthVerificationProviderId        | Integer | ID of date of birth verification provider         |
| DateOfBirthPassiveVerificationProviderId | Integer | ID of passive date of birth verification provider |
| DateOfBirthVerified                      | Boolean | Whether date of birth has been verified           |
| BirthPlace                               | Text    | User's place of birth                             |
| BirthPlaceVerificationProviderId         | Integer | ID of birth place verification provider           |
| BirthPlacePassiveVerificationProviderId  | Integer | ID of passive birth place verification provider   |
| BirthPlaceVerified                       | Boolean | Whether birth place has been verified             |
| Gender                                   | Text    | User's gender                                     |
| GenderVerificationProviderId             | Integer | ID of gender verification provider                |
| GenderPassiveVerificationProviderId      | Integer | ID of passive gender verification provider        |
| GenderVerified                           | Boolean | Whether gender has been verified                  |
| Nationality                              | Text    | User's nationality                                |
| NationalityVerificationProviderId        | Integer | ID of nationality verification provider           |
| NationalityPassiveVerificationProviderId | Integer | ID of passive nationality verification provider   |
| NationalityVerified                      | Boolean | Whether nationality has been verified             |
| Country                                  | Text    | User's country                                    |
| CountryVerificationProviderId            | Integer | ID of country verification provider               |
| CountryPassiveVerificationProviderId     | Integer | ID of passive country verification provider       |
| CountryVerified                          | Boolean | Whether country has been verified                 |
| ZipCode                                  | Text    | User's postal/zip code                            |
| ZipCodeVerificationProviderId            | Integer | ID of zip code verification provider              |
| ZipCodePassiveVerificationProviderId     | Integer | ID of passive zip code verification provider      |
| ZipCodeVerified                          | Boolean | Whether zip code has been verified                |
| Town                                     | Text    | User's town/city                                  |
| TownVerificationProviderId               | Integer | ID of town verification provider                  |
| TownPassiveVerificationProviderId        | Integer | ID of passive town verification provider          |
| TownVerified                             | Boolean | Whether town has been verified                    |
| Street                                   | Text    | User's street name                                |
| StreetVerificationProviderId             | Integer | ID of street verification provider                |
| StreetPassiveVerificationProviderId      | Integer | ID of passive street verification provider        |
| StreetVerified                           | Boolean | Whether street has been verified                  |
| HouseNumber                              | Text    | User's house number                               |
| HouseNumberVerificationProviderId        | Integer | ID of house number verification provider          |
| HouseNumberPassiveVerificationProviderId | Integer | ID of passive house number verification provider  |
| HouseNumberVerified                      | Boolean | Whether house number has been verified            |
| PhoneNumber                              | Text    | User's phone number                               |
| PhoneNumberInternational                 | Text    | Phone number in international format              |
| PhoneNumberNational                      | Text    | Phone number in national format                   |
| PhoneCountryCode                         | Text    | Country code for phone number                     |
| PhoneCountryPrefix                       | Text    | Country prefix for phone number                   |
| PhoneNumberVerificationProviderId        | Integer | ID of phone number verification provider          |
| PhoneNumberPassiveVerificationProviderId | Integer | ID of passive phone number verification provider  |
| PhoneNumberVerified                      | Boolean | Whether phone number has been verified            |
| PhoneNumberConfirmed                     | Boolean | Whether phone number has been confirmed           |
| AddressReferenceId                       | Text    | Reference ID for address                          |
| Region                                   | Text    | User's region                                     |
| RegionCode                               | Text    | Region code                                       |
| Password                                 | Text    | User's password (encrypted)                       |
| InvitedAt                                | Text    | Timestamp when user was invited                   |
| Lang                                     | Text    | User's language preference                        |
| OAuthState                               | Text    | OAuth state parameter                             |
| OAuthScope                               | Text    | OAuth scope requested                             |
| OAuthCode                                | Text    | OAuth authorization code                          |
| CfIpCountry                              | Text    | Cloudflare IP country                             |
| CfRay                                    | Text    | Cloudflare Ray ID                                 |
| CfConnectingIp                           | Text    | Cloudflare connecting IP                          |
| CfVisitor                                | Text    | Cloudflare visitor data                           |
| CfContinent                              | Text    | Cloudflare continent                              |
| CfCountry                                | Text    | Cloudflare country                                |
| CfRegion                                 | Text    | Cloudflare region                                 |
| CfRegionCode                             | Text    | Cloudflare region code                            |
| CfCity                                   | Text    | Cloudflare city                                   |
| CfPostalCode                             | Text    | Cloudflare postal code                            |
| CfTimezone                               | Text    | Cloudflare timezone                               |
| CfLatitude                               | Text    | Cloudflare latitude                               |
| CfLongitude                              | Text    | Cloudflare longitude                              |
| UserAgent                                | Text    | User's browser user agent                         |
| AcceptLanguage                           | Text    | User's accepted languages                         |
| Referer                                  | Text    | HTTP referer header                               |

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
