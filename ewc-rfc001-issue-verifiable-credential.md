# EWC RFC001: Issue Verifiable Credential - v2.0

**Authors:** 
* Mr George Padaytti (iGrant.io, Sweden)
* Mr Lal Chandran (iGrant.io, Sweden)
* Dr Andreas Abraham (ValidatedID, Spain)

**Reviewers:**
* Dr Nikos Triantafyllou (University of the Aegean, Greece)
* Mr Florin Coptil (Bosch, Germany)
* Mr Matteo Mirabelli (Infocert, Italy)
* Dr Mikael Linden (Vero, Finland) 
* Mr Renaud Murat (Archipels, France)
* Mr. Sebastian Bickerle (Lissi ID, Germany)
* Mr. Quentin Drouot (Archipels, France)
* Mr. Edward Curran (Lissi ID, Germany)
* Mr. Björn Astrom (BankID, Sweden)
* Mr. Björn Molin (DIGG, Sweden)
* Mr. Pär W (BankID, Sweden)

**Status:**
Current: Draft 13 alignment
02-Sep-2024: Approved for v2.0 release

**Table of Contents**

- [EWC RFC001: Issue Verifiable Credential - v2.0](#ewc-rfc001-issue-verifiable-credential---v20)
- [1.0	Summary](#10summary)
- [2.0	Motivation](#20motivation)
- [3.0	Messages](#30messages)
  - [3.1	Credential offer](#31credential-offer)
  - [3.2	Credential offer response](#32credential-offer-response)
  - [3.3 Discover request](#33-discover-request)
  - [3.4 Discover response](#34-discover-response)
  - [3.5 Authorisation request](#35-authorisation-request)
    - [3.5.1 Using `authorization_details`](#351-using-authorization_details)
    - [3.5.2 Using `scope`](#352-using-scope)
  - [3.6 Authorisation response](#36-authorisation-response)
  - [3.7 Token request](#37-token-request)
    - [3.7.1 Authorisation code flow](#371-authorisation-code-flow)
    - [3.7.2 Pre-authorised code flow](#372-pre-authorised-code-flow)
  - [3.8 Token response](#38-token-response)
    - [3.8.1 With `authorization_details`](#381-with-authorization_details)
    - [3.8.1 With `scope`](#381-with-scope)
  - [3.9 Credential request](#39-credential-request)
  - [3.10 Credential response](#310-credential-response)
    - [3.10.1  In-time](#3101--in-time)
    - [3.10.2 Deferred](#3102-deferred)
- [4.0	Alternate response format](#40alternate-response-format)
- [5.0	Implementers](#50implementers)
- [6.0	Reference](#60reference)
- [Appendix A: Public key resolution](#appendix-a-public-key-resolution)


# 1.0	Summary

This specification implements OID4VCI workflow for any issuer as per reference specification [1]. This minimises risks towards interoperability across the European Wallet Ecosystem with a standard specification in the EUDI wallet ecosystem as per the ARF [2] requirements. 

# 2.0	Motivation

The EWC LSP must align with the standard protocol for issuing credentials. This is the basis of interoperability between Issuers and Holders across the EWC LSPs. The assumption is that the user is familiar with the EWC-chosen protocols and standards and can refer to original standards references when necessary.

# 3.0	Messages

The credential issuance can be an authorisation flow or a pre-authorised one. These are depicted in the following diagrams, the assumption is that credential offer is obtained by holder wallet prior to discovery using same device (i.e. clicking on a link) or cross device (i.e scanning a QR code) flows.

```mermaid
sequenceDiagram
  autonumber
  participant I as Individual using EUDI Wallet
  participant O as Organisational Wallet (Issuer)
  
  O->>I: GET: Credential Offer
  Note over I,O: Discovery issuer capabilities
  I->>O: GET: /.well-known/openid-credential-issuer
  O-->>I: OpenID credential issuer configuration
  I->>O: GET: /.well-known/oauth-authorization-server
  O-->>I: OAuth authorisation server metadata

  Note over I,O: Authenticate and Authorise
  I->>O: Authorisation request
  O-->>I: Authorisation response
  I->>O: Token request
  O-->>I: Token response

  Note over I,O: Issue credential
  I->>O: POST: Credential request with token
  O-->>I: Credential response with acceptance token
```
Figure 1: Issuance using Authorisation Code Flow based on [1]

```mermaid
sequenceDiagram
    autonumber
    participant I as Individual using EUDI Wallet
    participant O as Organisational Wallet (Issuer)

    O->>I: GET: Credential Offer
    
    Note over I,O: Discovery issuer capabilities
    I->>O: GET: /.well-known/openid-credential-issuer
    O-->>I: OpenID credential issuer configuration
    I->>O: GET: /.well-known/oauth-authorization-server
    O-->>I: OAuth authorisation server metadata

    Note over I,O: Authenticate and Authorise
    I->>O: POST: Pre-authorised token request with transaction code
    O-->>I: Token response

    Note over I,O: Issue credential
    I->>O: POST: Credential request with token
    O-->>I: Credential response with acceptance token
```
Figure 2: Issuance using Pre-Authorisation Code Flow based on [1]

## 3.1	Credential offer

The recommendation is to send the Credential Offer by Reference Using the `credential_offer_uri` Parameter to avoid QR Code data overload. The endpoint that is expected to be embedded in the QR code is:

```
openid-credential-offer://?credential_offer_uri=https://server.example.com/credential-offer
```

Here, the `credential_offer_uri` query param contains the URL in which the credential offer from the issuer can be resolved. 

## 3.2	Credential offer response

Once the `credential_offer_uri` query param is resolved, the response can be either of the following formats. 

For authorised code flow, the credential offer response is as given:

```json
{
    "credential_issuer": "https://server.example.com",
    "credential_configuration_ids": [
        "VerifiablePortableDocumentA1"
    ],
    "grants": {
        "authorization_code": {
            "issuer_state": "eyJhbGciOiJSU0Et...FYUaBy"
        }
    }
}
```

For pre-authorised flow with a transaction code, the credential offer response is as given:

```json
{
   "credential_issuer": "https://server.example.com",
   "credential_configuration_ids": [
      "VerifiablePortableDocumentA1",
   ],
   "grants": {
      "urn:ietf:params:oauth:grant-type:pre-authorized_code": {
         "pre-authorized_code": "oaKazRN8I0IbtZ0C7JuMn5",
         "tx_code": {
            "length": 4,
            "input_mode": "numeric",
            "description": "Please provide the one-time code that was sent via e-mail or offline"
         }
      }
   }
}
```

For pre-authorised flow without a transaction code, the credential response is as given:

```json
{
   "credential_issuer": "https://server.example.com",
   "credential_configuration_ids": [
      "VerifiablePortableDocumentA1",
   ],
   "grants": {
      "urn:ietf:params:oauth:grant-type:pre-authorized_code": {
         "pre-authorized_code": "oaKazRN8I0IbtZ0C7JuMn5",
      }
   }
}
```

The following options are available for different grant types:

**For `authorization_code`**

| Field                    | Description                                                                                                                                                                                                                                                                                                             |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **issuer_state**         | OPTIONAL. String value created by the Credential Issuer and opaque to the Wallet. Used to bind the subsequent Authorization Request with the Credential Issuer to a context set up during previous steps. If received, it MUST be included in the subsequent Authorization Request as the issuer_state parameter value. |
| **authorization_server** | OPTIONAL string that the Wallet can use to identify the Authorization Server when the authorization_servers parameter in the Credential Issuer metadata has multiple entries. It MUST match one of the values in the authorization_servers array.                                                                       |

For `urn:ietf:params:oauth:grant-type:pre-authorized_code`:

| Field                               | Description                                                                                                                                                                                                                                                                                                                                                                                                  |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **pre-authorized_code**             | REQUIRED. The code representing the Credential Issuer's authorisation for the Wallet to obtain Credentials of a certain type. This code MUST be short-lived and single-use. It MUST be included in the subsequent Token Request.                                                                                                                                                                             |
| **tx_code**                         | OPTIONAL. Object specifying whether the Authorization Server expects presentation of a Transaction Code by the End-User along with the Token Request. Intended to bind the Pre-Authorized Code to a certain transaction to prevent replay. If required, it MUST be sent in the tx_code parameter. Read more [here](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html#section-4.1.1). |
| &nbsp;&nbsp;&nbsp;&nbsp;input_mode  | OPTIONAL. String specifying the input character set. Possible values are numeric (only digits) and text (any characters). The default is numeric.                                                                                                                                                                                                                                                            |
| &nbsp;&nbsp;&nbsp;&nbsp;length      | OPTIONAL. Integer specifying the length of the Transaction Code. This helps the Wallet render the input screen and improve the user experience.                                                                                                                                                                                                                                                              |
| &nbsp;&nbsp;&nbsp;&nbsp;description | OPTIONAL. String containing guidance for the Holder of the Wallet on how to obtain the Transaction Code. It is RECOMMENDED to display this description next to the Transaction Code input screen. The length MUST NOT exceed 300 characters.                                                                                                                                                                 |
| **interval**                        | OPTIONAL. The minimum amount of time in seconds that the Wallet SHOULD wait between polling requests to the token endpoint. If no value is provided, Wallets MUST use 5 as the default.                                                                                                                                                                                                                      |
| **authorization_server**            | OPTIONAL string that the Wallet can use to identify the Authorization Server when the authorization_servers parameter in the Credential Issuer metadata has multiple entries. It MUST match one of the values in the authorization_servers array.                                                                                                                                                            |

## 3.3 Discover request

Here, the holder wallet requests the issuer’s authorisation server configurations.

Resolve `/.well-known/openid-credential-issuer` endpoint for `credential_issuer` URI in the credential offer response.

```http
GET https://server.example.com/.well-known/openid-credential-issuer
```

Resolve `/.well-known/oauth-authorization-server` endpoint for `authorization_server` URI present in the response for the above.

```http
GET https://server.example.com/.well-known/oauth-authorization-server
```

## 3.4 Discover response

Once the well-known endpoint for **issuer server** configuration is resolved, the response is as given below with credentials_supported as defined by [6]:

```json
{
  "credential_issuer": "https://server.example.com",
  "authorization_servers": [
    "https://server.example.com"
  ],
  "credential_endpoint": "https://server.example.com/credential",
  "deferred_credential_endpoint": "https://server.example.com/credential_deferred",
  "display": [
    {
      "name": "Issuer",
      "location": "Belgium",
      "locale": "en-GB",
      "description": "For queries about how we manage your data please contact the Data Protection Officer."
    }
  ],
  "credential_configurations_supported": {
    "VerifiablePortableDocumentA1": {
      "format": "dc+sd-jwt",
      "scope": "VerifiablePortableDocumentA1",
      "cryptographic_binding_methods_supported": [
        "jwk"
      ],
      "credential_signing_alg_values_supported": [
        "ES256"
      ],
      "display": [
        {
          "name": "Portable Document A1",
          "locale": "en-GB",
          "background_color": "#12107c",
          "text_color": "#FFFFFF"
        }
      ],
      "vct": "VerifiablePortableDocumentA1",
      "claims": {
        "given_name": {
          "display": [
            {
              "name": "Given Name",
              "locale": "en-GB"
            },
            {
              "name": "Vorname",
              "locale": "de-DE"
            }
          ]
        },
        "last_name": {
          "display": [
            {
              "name": "Surname",
              "locale": "en-GB"
            },
            {
              "name": "Nachname",
              "locale": "de-DE"
            }
          ]
        }
      }
    }
  }
}
```

> [!NOTE]
> The `credential_configurations_supported` field and it's values change based on the supported credential formats 1) `mso_mdoc` 2) `jwt_vc_json` 3) `dc+sd-jwt`
> It is important to consult the relevant documentation for each format to ensure that all required fields and values are correctly configured.
> The supported credential format identifiers in the context of EWC LSPs, can be found [here](https://github.com/EWC-consortium/eudi-wallet-rfcs/blob/main/ewc-supported-formats.csv).

Once the well-known endpoint for authorisation server configuration is resolved, the response is as given below:

```json
{
  "issuer": "https://server.example.com",
  "authorization_endpoint": "https://server.example.com/authorize",
  "pushed_authorization_request_endpoint": "https://server.example.com/par",
  "require_pushed_authorization_requests": true,
  "token_endpoint": "https://server.example.com/token",
  "jwks_uri": "https://server.example.com/.well-known/jwks.json",
  "response_types_supported": [
    "code",
    "vp_token",
    "id_token"
  ],
  "subject_types_supported": [
    "public",
    "pairwise"
  ],
  "id_token_signing_alg_values_supported": [
    "ES256"
  ],
  "pre-authorized_grant_anonymous_access_supported": true,
  "token_endpoint_auth_methods_supported": [
    "none"
  ]
}

```

## 3.5 Authorisation request

There are two possible ways to request the issuance of a specific Credential type in an Authorization Request. One way is to use the `authorization_details` request parameter with one or more authorization details objects of type `openid_credential`. The other method is through the use of `scope`.

> [!NOTE]
> HAIP[6] chapter 4.2 mandatorily requires Pushed Authorisation Request as per [7]

### 3.5.1 Using `authorization_details`

The authorisation request is to grant access to the credential endpoint. Below is an example of such a request:

```http
GET /authorize?
  response_type=code
  &client_id=s6BhdRkqt3
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
  &code_challenge_method=S256
  &authorization_details=[{"type": "openid_credential", "
    credential_configuration_id": "VerifiablePortableDocumentA1"}]
  &redirect_uri=https://client.example.org/cb
Host: server.example.com
```

The query params for the authorisation request with `authorization_details` are as described below:

<table>
  <tr>
    <td>
      <code>response_type</code>
    </td>
    <td>REQUIRED. The value must be "code" to request an authorization code. </td>
  </tr>
  <tr>
    <td>
      <code>client_id</code>
    </td>
    <td>REQUIRED. The identifier for the client making the request. </td>
  </tr>
  <tr>
    <td>
      <code>code_challenge</code>
    </td>
    <td>REQUIRED. The code challenge used for Proof Key for Code Exchange (PKCE) as specified in OAuth 2.0 for Public Clients [5] </td>
  </tr>
  <tr>
    <td>
      <code>code_challenge_method</code>
    </td>
    <td>OPTIONAL. The method used to transform the code verifier. Defaults to "plain" if not present. Possible values are "S256" or "plain", as defined in PKCE for OAuth 2.0. </td>
  </tr>
  <tr>
    <td>
      <code>authorization_details</code>
    </td>
    <td>OPTIONAL. Provides fine-grained access details as specified in the OAuth 2.0 Rich Authorization Requests specification. The `authorization_details` parameter, as defined in Section 2 of [RFC9396], should be used to convey the specifics of the Credentials the Wallet intends to obtain. This specification introduces a new authorization details type, `openid_credential`. [4]. 
   
   An example of W3C VC credential format is as given below:

   ```json
    [
        {
            "type": "openid_credential",
            "credential_configuration_id": "VerifiablePortableDocumentA1",
            "credential_definition": {
                "credentialSubject": {}
            }
        }
    ]
   ```

  The IETF SD-JWT VC is as given:

   ```json
  [
      {
          "type": "openid_credential",
          "format": "dc+sd-jwt",
          "vct": "VerifiablePortableDocumentA1"
      }
  ]
  ```
   </td>
  </tr>
  <tr>
    <td>
      <code>redirect_uri</code>
    </td>
    <td>OPTIONAL. The redirection endpoint where the authorization server will send the user-agent after authorization is complete. </td>
  </tr>
  <tr>
    <td>
      <code>issuer_state</code>
    </td>
    <td>OPTIONAL. A string value representing a specific processing context at the Credential Issuer. This value is usually provided in a Credential Offer from the Credential Issuer to the Wallet and is used to pass the issuer_state value back to the Credential Issuer. </td>
  </tr>
</table>

### 3.5.2 Using `scope`

Below is an example of such a request using `scope` parameter. Here, the wallet gets the `scope` during discovery:

```http
GET https://my-issuer.rocks/auth/authorize?
  response_type=code
  &scope=VerifiablePortableDocumentA1
  &resource=https%3A%2F%2Fcredential-issuer.example.com
  &client_id=s6BhdRkqt3
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
  &code_challenge_method=S256
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
Host: server.example.com
```

The query params for the authorisation request with `scope` are as described below:

<table>
  <tr>
    <td>
      <code>response_type</code>
    </td>
    <td>REQUIRED. The value must be "code" to request an authorization code. </td>
  </tr>
  <tr>
    <td>
      <code>scope</code>
    </td>
    <td>A JSON string that identifies the scope value supported by the Credential Issuer for a particular Credential. The value can be consistent across multiple credential configuration objects. The Authorization Server must uniquely identify the Credential Issuer based on the scope value. The Wallet can use this value in the Authorization Request. Scope values in this Credential Issuer metadata may overlap with those in the scopes_supported parameter of the Authorization Server. </td>
  </tr>
  <tr>
    <td>
      <code>client_id</code>
    </td>
    <td>REQUIRED. The identifier for the client making the request. </td>
  </tr>
  <tr>
    <td>
      <code>redirect_uri</code>
    </td>
    <td>OPTIONAL. The redirection endpoint where the authorization server will send the user-agent after authorization is complete. </td>
  </tr>
  <tr>
    <td>
      <code>code_challenge</code>
    </td>
    <td>REQUIRED. The code challenge used for Proof Key for Code Exchange (PKCE) as specified in OAuth 2.0 for Public Clients [5] </td>
  </tr>
  <tr>
    <td>
      <code>code_challenge_method</code>
    </td>
    <td>OPTIONAL. The method used to transform the code verifier. Defaults to "plain" if not present. Possible values are "S256" or "plain", as defined in PKCE for OAuth 2.0. </td>
  </tr>
  <tr>
    <td>
      <code>issuer_state</code>
    </td>
    <td>OPTIONAL. A string value representing a specific processing context at the Credential Issuer. This value is usually provided in a Credential Offer from the Credential Issuer to the Wallet and is used to pass the issuer_state value back to the Credential Issuer. </td>
  </tr>
</table>

## 3.6 Authorisation response

The credential issuer can **optionally** request additional details to authenticate the client e.g. DID authentication. In this case, the authorisation response will contain a `response_mode` parameter with the value `direct_post`. A sample response is as given:

```http
HTTP/1.1 302 Found
Location: http://localhost:8080?state=22857405-1a41-4db9-a638-a980484ecae1&client_id=https://example.server.com&redirect_uri=https://example.server.com/direct_post&response_type=id_token&response_mode=direct_post&scope=openid&nonce=a6f24536-b109-4623-a41a-7a9be932bdf6&request_uri=https://example.server.com/request_uri
```

Query params for the authorisation response are given below:

<table>
  <tr>
   <td><code>state</code>
   </td>
   <td>The client uses an opaque value to maintain the state between the request and callback.
   </td>
  </tr>
  <tr>
   <td><code>client_id</code>
   </td>
   <td>Decentralised identifier
   </td>
  </tr>
  <tr>
   <td><code>redirect_uri</code>
   </td>
   <td>For redirection of the response
   </td>
  </tr>
  <tr>
   <td><code>response_type</code>
   </td>
   <td>The value must be <code>id_token</code> if the issuer requests DID authentication.
   </td>
  </tr>
  <tr>
   <td><code>response_mode</code>
   </td>
   <td>The value must be <code>direct_post</code>
   </td>
  </tr>
  <tr>
   <td><code>scope</code>
   </td>
   <td>The value must be <code>openid</code>
   </td>
  </tr>
  <tr>
   <td><code>nonce</code>
   </td>
   <td>A value used to associate a client session with an ID token and to mitigate replay attacks
   </td>
  </tr>
  <tr>
   <td><code>request_uri</code>
   </td>
   <td>This is intended for scenarios where the authorization request is large. The URI can be used by the holder to retrieve the authorization request.
   </td>
  </tr>
</table>


The holder wallet then responds with an `id_token` signed by the DID to the direct post endpoint.

```http
POST /direct_post
Content-Type: application/x-www-form-urlencoded
&id_token=eyJraWQiOiJkaW...a980484ecae1
```

If additional details are not requested, the credential issuer will send an authorisation response with a `code` query parameter containing the short-lived authorisation code. A sample response is given below:

```http
HTTP/1.1 302 Found
Location: https://Wallet.example.org/cb?code=SplxlOBeZQQYbYS6WxSbIA
```

## 3.7 Token request

### 3.7.1 Authorisation code flow

The token request for authorised code flow is as given:

```http
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW

&grant_type=authorization_code
&code=SplxlOBeZQQYbYS6WxSbIA
&code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
&redirect_uri=https%3A%2F%2FWallet.example.org%2Fcb
```

This request is made with the following query params:

<table>
  <tr>
   <td><code>grant_type</code>
   </td>
   <td>Grant type for authorisation. E.g. <code>authorization_code</code>
   </td>
  </tr>
  <tr>
   <td><code>client_id</code>
   </td>
   <td>Decentralised identifier
   </td>
  </tr>
  <tr>
   <td><code>code</code>
   </td>
   <td>Authorisation code
   </td>
  </tr>
  <tr>
   <td><code>code_verifier</code>
   </td>
   <td>Wallet-generated secure random token used to validate the original <code>code_challenge</code> provided in the initial Authorization Request
   </td>
  </tr>
   <tr>
   <td><code>redirect_uri</code>
   </td>
   <td>For redirection of the response as per <a href=#https://www.rfc-editor.org/rfc/rfc6749.html#section-3.1.2">IETF RFC6749 Section 3.1.2</a>
   </td>
  </tr>
</table>

### 3.7.2 Pre-authorised code flow

The token request for pre-authorised code flow is as given:

```http
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

&grant_type=urn:ietf:params:oauth:grant-type:pre-authorized_code
&pre-authorized_code=SplxlOBeZQQYbYS6WxSbIA
&tx_code=493536
```

This request is made with the following query params:

<table>
  <tr>
   <td><code>grant_type</code>
   </td>
   <td>Grant type for authorisation. E.g. <code>urn:ietf:params:oauth:grant-type:pre-authorized_code</code>
   </td>
  </tr>
  <tr>
   <td><code>pre-authorized_code</code>
   </td>
   <td>Code representing the Credential Issuer's authorisation for the Wallet to obtain Credentials of a certain type. This code must be short-lived and single-use.
   </td>
  </tr>
  <tr>
   <td><code>tx_code</code>
   </td>
   <td>Specifies if the Authorization Server expects the End-User to present a Transaction Code with the Token Request in a Pre-Authorized Code Flow. If not required, this object is absent by default. The Transaction Code binds the Pre-Authorized Code to a specific transaction, preventing replay attacks. The End-User PIN is set by the issuer and sent to the holder via email, SMS, or another out-of-band method.
   </td>
  </tr>
</table>

## 3.8 Token response

### 3.8.1 With `authorization_details`

If `authorization_details` is used in authorisation request ([refer chapter 3.5.1](#351-using-authorization_details)), the token response will be as given:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp..sHQ",
  "token_type": "bearer",
  "expires_in": 86400,
  "c_nonce": "tZignsnFbp",
  "c_nonce_expires_in": 86400,
  "authorization_details": [
    {
      "type": "openid_credential",
      "credential_configuration_id": "VerifiablePortableDocumentA1",
      "credential_identifiers": [ "VerifiablePortableDocumentA1-Spain", "VerifiablePortableDocumentA1-Sweden", "VerifiablePortableDocumentA1-Germany" ]
    }
  ]
}
```

### 3.8.1 With `scope`

If `scope` is used in authorisation request ([refer chapter 3.5.2](#352-using-scope)), the token response will be as given:

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp..sHQ",
    "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI4a5k..zEF",
    "token_type": "bearer",
    "expires_in": 86400,
    "id_token": "eyJodHRwOi8vbWF0dHIvdGVuYW50L..3Mz",
    "c_nonce": "PAPPf3h9lexTv3WYHZx8ajTe",
    "c_nonce_expires_in": 86400
}
```

## 3.9 Credential request

The Holder wallet makes a Credential Request to the Credential Endpoint as below:

**For W3C VC with credential format identifier** `jwt_vc_json`:

```http
POST /credential
Content-Type: application/json
Authorization: Bearer eyJ0eXAi...KTjcrDMg

{
   "format": "jwt_vc_json",
   "credential_definition": {
      "type": [
         "VerifiableCredential",
         "VerifiablePortableDocumentA1"
      ]
   },
   "proof": {
      "proof_type": "jwt",
      "jwt":"eyJraWQiOiJkaWQ6ZX..zM"
   }
}
```
> [!NOTE]
> In the above, the credentialSubject is optional and is not considered within the scope of EWC LSP. 

**For IETF SD-JWT VC with credential format identifier** `dc+sd-jwt`:


```http
POST /credential
Content-Type: application/json
Authorization: Bearer eyJ0eXAi...KTjcrDMg

{
   "format": "dc+sd-jwt",
   "vct": "SD_JWT_VC_example_in_OpenID4VCI",
   "proof": {
      "proof_type": "jwt",
      "jwt":"eyJ0eXAiOiJvc..1WlA"
   }
}
```

If specified, you can request specific fields to be included in the issued credential. If its not specified, all fields in the credential is included.

## 3.10 Credential response

The credential response can happen in-time or can be deferred as described below.

### 3.10.1  In-time

The In-time flow indicates that the credential is available immediately and the response format is as below: 

```json
{
  "credential": "eyJ0eXAiOi...F0YluuK2Cog",
  "c_nonce": "fGFF7UkhLa",
  "c_nonce_expires_in": 86400
}
```

### 3.10.2 Deferred

If the credential is unavailable, the issuer responds with an acceptance token, indicating credential issuance is deferred. The response is as below:

```json
{
  "transaction_id": "8xLOxBtZp8",
  "c_nonce": "wlbQc6pCJp",
  "c_nonce_expires_in": 86400
}
```

The `transaction_id` is used to identity the deferred transaction when the credential is issued at a later time with the following deferred credential endpoint:

```http
POST /deferred-credential
Authorization: BEARER eyJ0eXAiOiJKV1QiLCJhbGci..zaEhOOXcifQ
{
   "transaction_id": "8xLOxBtZp8"
}
```

> [!NOTE]
> If the response contains `transaction_id` field, then it can be understood the credential is not available now and should be later available through the deferred credential endpoint. An example response is as given below:

```json
{
  "transaction_id": "eyJ0eXAiOiJKV1QiLCJhbGci..zaEhOOXcifQ",
  "c_nonce": "wlbQc6pCJp",
  "c_nonce_expires_in": 86400
}
```

# 4.0	Alternate response format

Standard HTTP response codes shall be supported. Any additional ones can be formulated in the following format.

```
{
  "error": "invalid_request",
  "error_description": "The verifiable credential is expired"
}
```
The table below summarises the success/error responses that can be used:

<table>
  <tr>
   <td><strong>Response format</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>invalid_request
   </td>
   <td>Request failed. E.g. The verifiable credential is expired
   </td>
  </tr>
  <tr>
   <td>invalid_grant
   </td>
   <td>
<ul>

<li>The Authorization Server expects a PIN in the Pre-Authorized Code Flow but the Client provides the wrong PIN.

<li>The end user provides the wrong Pre-Authorized Code or the Pre-Authorized Code has expired.
</li>
</ul>
   </td>
  </tr>
  <tr>
   <td>invalid_client
   </td>
   <td>The Client tried to send a Token Request with a Pre-Authorized Code without Client ID, but the Authorization Server does not support anonymous access
   </td>
  </tr>
</table>

# 5.0	Implementers

Please refer to the [implementers table](https://github.com/EWC-consortium/eudi-wallet-rfcs?tab=readme-ov-file#implementers).

# 6.0	Reference

1. OpenID Foundation (2023), 'OpenID for Verifiable Credential Issuance', Available at: [https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0-ID1.html](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0-ID1.html) (Accessed: July 11, 2024).
2. European Commission (2023) The European Digital Identity Wallet Architecture and Reference Framework (2023-04, v1.1.0)  [Online]. Available at: [https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases) (Accessed: October 16, 2023).
3. OpenID Foundation (2023), 'Self-Issued OpenID Provider v2 (SIOP v2)', Available at: [https://openid.net/specs/openid-connect-self-issued-v2-1_0.html](https://openid.net/specs/openid-connect-self-issued-v2-1_0.html) (Accessed: October 01, 2023)
4. OAuth 2.0 Rich Authorization Requests, Available at: [https://datatracker.ietf.org/doc/html/draft-ietf-oauth-rar-11](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-rar-11) (Accessed: February 01, 2024)
5. Proof Key for Code Exchange by OAuth Public Clients, Available at: [https://datatracker.ietf.org/doc/html/rfc7636](https://datatracker.ietf.org/doc/html/rfc7636) (Accessed: February 01, 2024)
6. OpenID4VC High Assurance Interoperability Profile with SD-JWT VC - draft 00, Available at [https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-sd-jwt-vc-1_0.html](https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-sd-jwt-vc-1_0.html) (Accessed: February 16, 2024)
7. Lodderstedt, T., Campbell, B., Sakimura, N., Tonge, D., and F. Skokan, "OAuth 2.0 Pushed Authorization Requests", RFC 9126, DOI 10.17487/RFC9126, September 2021, <https://www.rfc-editor.org/info/rfc9126>.

# Appendix A: Public key resolution

For a JWT there are multiple ways for resolving the public key using the `kid` header claim:

* If the key identifier is a DID then use a DID resolver to obtain the public key
* If the key identifier is not a DID, then resolve the JWKs endpoint in the AS configuration and match the public key from the JWK set using the key identifier.

Additionally, it is possible to specify JWK directly in the header using `jwk` header claim.
