## Technical Requirement Description

### XS2A Interface Specification
The XS2A Interface is designed as a B2B interface between a TPP server and the ASPSP server. For time being, the protocol defined in this document is a pure client-server protocol, assuming the TPP server being the client, i.e. all API calls are initiated by the TPP. In future steps, this protocol might be extended to a server-server protocol, where also the ASPSP initiates API calls towards the TPP.

For the specification the two layers shown in the following figure are distinguished:

![IMAGE](quiver-image-url/311FABFC232F01FF6D580452E0E362EE.jpg =337x197)

At the application layer only the core services will be specified in the first version of the framework. In addition the framework will be prepared such that the interface of an ASPSP may be extended with its own additional corporate specific services (included in the figure as "Extended services"). In future versions of this framework, some extended services will also be part of the standard. This framework documentation will point out extended services where the market need is already identified.

### Character Sets and Notations
The character set is UTF 8 encoded. This specification is only using the basic data elements **"String"**, **"Boolean"**, **"ISODateTime"**, **"ISODate"**, **"UUID"** and **"Integer"** (with a byte length of 32 bits) and ISO based code lists. For codes defined by ISO, a reference to the corresponding ISO standard can be found here: https://www.iso20022.org/payments_messages.page

**Max35Text**, **Max70Text**, **Max140Text** and **Max512Text** are defining strings with a maximum length of **35**, **70**, **140** and **512** characters respectively.

ASPSPs will accept for strings at least the following character set:
```
a b c d ef g h i j k l m n o p q r s t u v w x y z
A B C D E F G H I J K L M N O P Q R S T U VW X Y Z 0 1 23 45 67 89
/-?:().,' +
Space
```

ASPSPs may accept further character sets for text fields like names, addresses, text. Corresponding information will be contained in the ASPSP documentation of the XS2A interface. ASPSPs might convert certain special characters of these further character sets, before forwarding e.g. submitted payment data.

### Transport Layer
The communication between the TPP and the ASPSP is always secured by using a TLS- connection using TLS version 1.2 or higher. For the choice of cipher suite selections, NIST recommendations on the cryptographical strength should be followed. For ASPSPs, further cipher suite requirements of their national IT security agency might apply.

This TLS-connection is set up by the TPP. It is not necessary to set up a new TLS- connection for each transaction, however the ASPSP might terminate an existing TLS- connection if required by its security setting.

The TLS-connection has to be established always including client (i.e. TPP) authentication. For this authentication the TPP has to use a qualified certificate for website authentication. This qualified certificate has to be issued by a qualified trust service provider according to the eIDAS regulation [eIDAS]. The content of the certificate has to be compliant with the requirements of [EBA-RTS]. The certificate of the TPP has to indicate all roles the TPP is authorised to use.


### Location of Message Parameters
The XS2A Interface definition follows the REST service approach. This approach allows to transport message parameters at different levels:
- message parameters as part of the HTTP level (HTTP header)
- message parameters by defining the resource path (URL path information) with
additional query parameters and
- message parameters as part of the HTTP body

The content parameters in the corresponding HTTP body will be encoded either in JSON or in XML syntax. XML syntax is only used where
- an ISO20022 based payment initiation (pain.001 message) with the corresponding payment initiation report (pain.002 message) or
- ISO 20022 based account information message (camt.052, camt.053 or camt.054 message)
is contained.

As an exception, response messages might contain plain text format in account information messages to support MT940, MT941 or MT942 message formats.

The parameters are encoded in
- in spinal-case (small letters) on path level,
- in Spinal-case (starting capital letters) on HTTP header level and
- in lowerCamelCase for query parameters and JSON based content parameters.

The following principle is applied when defining the API:

Message parameters as part of the HTTP header:
- Definition of the content syntax,
- Certificate and Signature Data where needed,
- PSU identification data (the actual data from the online banking frontend or access token),
- Protocol level data like Request Timestamps or Request/Transaction Identifiers

Message parameters as part of the path level:
- All data addressing a resource:
  - Provider identification,
  - Service identification,
  - Payment product identification,
  - Account Information subtype identification,
  - Resource ID

Query Parameters:
- Additional information needed to process the GET request for filtering information,

Message parameters as part of the HTTP body:
- Business data content,
- PSU authentication data,
- Messaging Information
- Hyperlinks to steer the full TPP – ASPSP process

### Signing Messages at Application Layer

The ASPSP may require the TPP to sign request messages.

The signature shall be included in the HTTP header as defined by [signHTTP](https://datatracker.ietf.org/doc/draft-cavage-http-signatures/), chapter 4.

The electronic signature of the TPP has to be based on a qualified certificate for electronic seals. This qualified certificate has to be issued by a qualified trust service provider according to the eIDAS regulation [eIDAS]. The content of the certificate has to be compliant with the requirements of [EBA-RTS]. The certificate of the TPP has to indicate all roles the TPP is authorised to use.

This specification uses on a pure protocol level the following HTTP header in all HTTP requests uniformously for the support of the signature function:

**Request Header**
Attribute | Type | Condition | Descriprion
:-- | :-- | :-- | :--
`Digest` | string | conditional | Is contained if and only if the "Signature" element is contained in the header of the request.
`Signature` | string | conditional | A signature of the request by the TPP on application level.
`Tpp-Signature-Certificate` | string | conditional | The certificate used for signing the request, in base64 encoding. Must be contained if a signature is contained, see above.

### XS2A Interface API Structure

The XS2A Interface is resource oriented. Resources can be addressed under the API endpoints

`https://{provider}/v1/{service}{?query-parameters}`

using additional content parameters {parameters}

where
- {provider} is the host and path of the XS2A API, which is not further mentioned. The host or path may contain release version information of the ASPSP.
- v1 is denoting the final version 1.3.4 of the Berlin Group XS2A interface Implementation Guidelines.

The structure of the request/response is described according to the following categories
- Path: Attributes encoded in the Path, e.g. "payments/sepa-credit-transfers" for {resource}
- Query Parameters: Attributes added to the path after the "?" sign as process steering flags or filtering attributes for GET access methods. Query parameters of type Boolean shall always be used in a form query-parameter=true or query- parameter=false.
- Header: Attributes encoded in the HTTP header of request or response
- Request: Attributes within the content parameter set of the request
- Response: Attributes within the content parameter set of the response, defined in
XML, text or JSON:
  - XML encoding appears only, when camt.052, camt.053 or camt.054 messages (reports, notifications or account statements) or pain.002 payment status messages are transported. pain.002 messages will only be delivered for the GET Status Request, and only in cases where the payment initiation was performed by using pain.001 messages.
  - Text encoding appears only, when MT940, MT941 or MT942 messages (reports, notifications or account statements) are transported.
  - All other response bodies are encoded in JSON.

## Consent Creation flow Sequence Diagram

This diagram describes technical implementation of Consent Creation Flow (initiating fetching of account information process).

![IMAGE](https://i.ibb.co/qyP3Pnn/Screen-Shot-2019-12-02-at-22-13-40.png)

### (1) POST /v1/consents
---
TPP instructs XS2A Interface to initialize the consent creation flow.

According to BerlinGroup Documentation, Request will look like this:

Headers | Condition | Description
:-- |:-: |:--
`X-Request-ID` | Mandatory | _uuid_ of request,
`Signature` | Conditional |[link](https://datatracker.ietf.org/doc/draft-cavage-http-signatures/?include_text=1), standard used to sign headers and body.
`TPP-Signature-Certificate` | Conditional | The certificate used for signing the request, in base64 encoding. Must be contained if `Signature` is contained.
`TPP-Redirect-Preferred` | Optional | true/false
`TPP-Redirect-URI` | Conditional | TPP redirect_uri on which PSU will be redirected after SCA Redirect Authorisation
`PSU-ID` | Optional | Client ID of the PSU in the ASPSP client interface
`Provider-Code` | Mandatory | **Custom**, identifier of requested Provider in Priora PSD2 Compliance

Body Params | Condition |Descriprion
:-- | :-: | :--
`access` | Mandatory, hash | Wrapper for access details for created consent
`access.balances` | Optional, array | List of IBAN's to get balances
`access.transactions` | Optional, array | List of IBAN's to get transactions
`recurringIndicator`| Mandatory, boolean | Recurring access or one-time-request for given accounts
`validUntil` | Mandatory, datetime | Expiration date of consent; E.g. '2019-11-01'
`frequencyPerDay` | Mandatory, integer | This field indicates the requested maximum frequency for an access without PSU involvement per day. For a one-off access, this attribute is set to "1".
`combinedServiceIndicator` | Optional, boolean | If "true" indicates that a payment initiation service will be addressed in the same "session" (combining AIS and PIS).

### (2) create authorisation + consent
---
XS2A Interface creates a Consent record and then it automatically creates an authorisation record linked to the Consent. This is possible in case that ASPSP supports SCA Redirect flow by default. In such case, TPPs are not required to send a separate `POST /api/berlingroup/v1/consents/{ID}/authorisations` request.

Below are represented, in the order of creation, the 2 models which will store the consent and authorisation data.

#### Consent
In BerlinGroup PSD2 XS2A context, consent represents the basic AISP flow service which should be authorised by the PSU. It holds explicit information about accounts/balance/transactions which PSU has offered access to.

Field | Example | Description
:-: | :-: | :--
`id` | **uuid** | unique Consent Identifier
`request_id` | **uuid** | unique identifier of the request initialized by TPP
`client_id` | bigint | reference to the TPP who initialized the consent creation flow
`customer_id` | bigint | reference to the PSU who gave consent. Will be added after PSU is identified
`status` | accepted | describes the current status of the consent
`access` | {"allPsd2" => "allAccounts"} | lists the resources requested to be accessed via API
`events` | **array** | very simmilar to the events of current implementation of Sessions
`recurringIndicator` | true | "true", if the consent is for recurring access to the account data
`frequencyPerDay` | 4 | this field indicates the requested maximum frequency for an access without PSU involvement per day. For a one-off access, this attribute is set to "1"
`validUntil` |  2017-10-30 | the datetime in ISO-Date Format to which consent will be valid

#### Authorisation
In BerlinGroup PSD2 XS2A context, authorisation represents a mandatory process of signing a specific AIS or PIS service on behalf of PSU.

Field | Example | Description
:-: | :-: | :--
`id` | **uuid** | unique Authorisation Identifier
`service_id` | **uuid** | reference to the service to be authorised, be it consent or payment
`request_id` | **uuid** | unique identifier of the request initialized by TPP
`service_type` | consent/payment | type of the requested service to be signed
`status` | started | very simmilar to the events of current implementation of Sessions
`events` | **array** | very simmilar to the events of Sessions

Under question for Authorisation: `TPP-Redirect-Preferred`, `TPP-Redirect-URI` headers.

### (3) response
---
Prior to response, XS2A Interface has to generate the available meta links for checking the status of consent and authorisation

The response should be of the following format:
#### Status:
201

#### Headers:
`X-Request-ID` - request identifier of the request responsible for consent creation.
`ASPSP-SCA-Approach` - **REDIRECT**. Possible values also: **EMBEDDED** or **DECOUPLED**

#### Body:
```json
{
  "consentStatus": "accepted",
  "consentId": "1234-wertiq-983",
  "_links": {
    "status": {
      "href": "/v1/consents/1234-wertiq-983/status"
    },
    "scaStatus": {
      "href": "v1/consents/1234-wertiq-983/authorisations/123auth567"
    }
  }
}
```


### (4) POST /api/priora/v1/tokens/create
---
XS2A Interface queues a worker which sends a request to connector via our current V1 connector API.
One challenge to beat during this stage is the communication via JWT.

The standard structure of request with **mapped values** for V1 connector API is:

```json
{
  "data": {
    "original_request": {
      "client_jwt": <Bearer jwt-for-client_payload>,
      "client_payload": {
        "data": {
          "provider_code": <got from 'Provider-Code' header>,
          "scopes": <from authorisation.service_type value>,
          "consent_period_days": <calculated from 'validUntil' datetime>,
          "force_sca": <'false' value as default>,
          "credentials": {
            "type": <'oauth' value as default for the beginning>,
            "authorization_type": <authorisation_type will be found and selected by XS2A Interface>
          },
          "redirect_url": <got from 'TPP-Redirect-URI' header>"
        },
        "exp": <timestamp 5 minutes from now>
      }
    },
    "session_secret": <authorisation_id>,
    "app_name": < Unknown, maybe from 'TPP-Cerficate' header>,
    "provider_code": <got from 'Provider-Code' header>
  },
  "exp": <timestamp 5 minutes from now>
}
```

### (5) response
---
Connector responds with an empty JSON, 200 status. No changes required from actual implementation.

### (6) Prepare Authorisation
---
Based on the request, Connector stores authorisation data and prepares a URL on which PSU will be redirected. No changes required from actual impelementation.

### (7) POST /api/connectors/v1/sessions/update
---
Connector updates the authorisation by sending the `redirect_url` on which it awaits the PSU to be redirected by TPP.

The structure of request for this request, according to current connector-docs is:

```json
{
  "data": {
    "session_secret": <authorisation uuid>,
    "session_expires_at": "2019-11-01T13:17:36.377Z" <idk what is this for, but fine.>,
    "status": "redirect" <status of the authorisation according to ASPSP.>,
    "extra": {
      "redirect_url": "https://testbank.com/authorisation/:uuid" <the ScaRedirect link>
    }
  },
  "exp": 1572614076 <timestamp of JWT>
}
```


### (8) response
---
Standard XS2A Interface response:

```json
{
  "data": {
  },
  "meta": {
    "time": "2019-11-01T13:12:36.366Z"
  }
}
```


### (9) update authorisation && consent
---
On receiving callback, XS2A Interface performs a check whether the `session_secret` belongs to the existing Session or it represents an Authorisation which belongs to BerlinGroup.

After identifying the Authorisation and its Consent:
- XS2A Interface updates both their statuses to `received`, which means that they were successfuly created and both are technically correct, yet not authorised.
- The consent linked to the authorisation, receives a new meta `_link` field called `scaRedirect`.
- `scaRedirect` hash will have a `href` key with provider `redirect_url`


### (10) POLL? GET /v1/consents/{consentId}
---
TPP has to poll the consent awaiting for status `received`, which will provide the `scaRedirect`, which PSU will use for redirect authorisation.

The format of response will be the following:
#### Status:
200

#### Headers:
`X-Request-ID` - request identifier of the request responsible for consent creation.

#### Body:
```json
{
  "access": {
    "allPsd2": "allAccounts"
  },
  "recurringIndicator": "true",
  "validUntil": "2017-11-01",
  "frequencyPerDay": "4",
  "consentStatus": "received",
  "_links": {
    "scaRedirect": {
      "href": "https://demobank.com/oauth/{authorisation_id}"
    }
  }
}
```


### (11) redirect
---
Once the TPP receives the `scaRedirect` URI, it redirects the PSU on the authorisation page.


### (12) PSU performs authorisation
---
PSU performs SCA Authorisation on bank side...


### (13) POST /api/connectors/v1/sessions/success
---
Upon successful authorisation, Connector sends a Session Success callback to XS2A Interface

The structure of request for this request, according to current connector-docs is:

```json
{
  "data": {
    "session_secret": <authorisation uuid>",
    "user_id": <customer_id linked to consent>,
    "token": <access_token, not necessary>,
    "token_expires_at": <token_expire_date, not necessary>,
    "extra": {
    },
    "refresh_token": <refresh_token, not necessary>
  },
  "exp": 1572614076
}
```

### (14) response
---
Standard XS2A Interface response:

```json
{
  "data": {
  },
  "meta": {
    "time": "2019-11-01T13:12:36.366Z"
  }
}
```

### (15) update authorisation && consent
---
On receiving callback, XS2A Interface performs a check whether the `session_secret` belongs to the existing Session or it represents an Authorisation which belojngs to BerlinGroup.

After identifying Authorisation and its Consent:
- Priora updates Authorisation Status to `psuAuthenticated`
- Priora updates Consent Status to `valid`.
- The Consent linked to the Authorisation, receives a new meta `_link` field called `account`.
- `account` hash will have a `href` key with accounts link `/v1/accounts`


### (16) redirect PSU back
---
After submitting a Session Success callback, Connector redirects PSU back on TPP using the `redirect_url` previously storred when authorisation was created. (4)


### (17) POLL? GET /v1/consents/{consentId}
---
TPP has to poll the consent awaiting for status `valid`, which will provide the meta resource `_link` for further actions.

The format of response will be the following:
#### Status:
200

#### Headers:
`X-Request-ID` - request identifier of the request responsible for consent creation.

#### Body:
```json
{
  "access": {
    "allPsd2": "allAccounts"
  },
  "recurringIndicator": "true",
  "validUntil": "2017-11-01",
  "frequencyPerDay": "4",
  "consentStatus": "valid",
  "_links": {
    "account": {
      "href": "/v1/accounts"
    }
  }
}
```
