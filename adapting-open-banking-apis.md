## Adapting an Open Banking API Implementation
The following sections describe the *recommended* upgrade scheme.  Note:
no changes to the Open Banking API itself is needed.
### 1. Deploy a Locally Trusted Certificate
Since traditional TTP services **MUST NOT** be able using the new mode,
a specific locally trusted certificate for TLS client-certificate authentication
**MUST** be deployed and recognized by the Open Banking API implementation.
### 2. Update OAuth2 "authorize" Method
In the new mode (as recognized by \#1) OAuth2 authorization normally only happens
during enrollment of virtual payment cards.
To enable frictionless `access_token` upgrades,
virtual payment cards are not minted with built-in access token information but rather
hold card identifiers (like serial numbers), which in turn are linked to a 
database table holding the currently valid `access_token` for each specific user.
This linking requires a user *identity token* to function.
Such tokens **MUST** adhere to the follwing rules:
- Be static over the period the user is associated with the bank
- Be unique per user and never be reused
- Be represented as an ASCII string

The *recommended* way is supplying the identity token as a part of the
OAuth2 authorization response using an extension property called `identity_token`:
```json
{
  "access_token": "619763e4-cf77-4d2f-838e-1f6c6b634040",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "da1bdd53-bed9-4cb7-9c62-0bbe0356d90b",
  "scope": "xyz",
  "identity_token": "479262777"
}
```
The `identity_token` is also needed for virtual payment card administation and logging.

Although strictly put not necessary, personalization of the visual part
of virtual payment cards would benefit by also getting the user's *human name* in
the authorization response.  If supplied it
**MUST** be a string of UTF-8 characters supplied in the extension
property `human_name`.
### 3. Suppress SCA Requests
Since the proposed "Wallet" scheme performs SCA (Strong Customer Authentication)
in a similar way to the EMV standard, the Open Banking API **MUST NOT**
ask the user for additional authentications.
### 4. Make `refresh_token` Life-time Unlimited
In the new mode, `refresh_token` may be renewed as usual but they **MUST NOT** expire.
### 5. Grant Consent Requests by Default
Although "consent" requests must still be verified for technical correctness,
they **MUST** be granted by default since *account data is never to be shared with external entities*
(a locally installed and trusted "Wallet" service is effectively like an extension to the on-line bank).

Note: skipping consent requests is not an option since that would break Open Banking APIs.
### 6. Optional: Reuse the On-line Bank Login
In a fully integrated solution the virtual payment card enrollment service would
preferably reuse the regular on-line bank login.  That is, the service would
simply appear as an additional choice in the on-line bank.
&nbsp;

&nbsp;

&nbsp;

Version 0.2, 2019-12-07
