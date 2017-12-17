---
title: "Home"
type: "post"
author: "Jeswin Kumar<jeswinpk@agilehead.com>"
---

<div style="background:RGBA(255,0,0,0.1); padding: 1em; margin-bottom: 2em; border-radius: 4px">
  PS: Examples and Download links are still work in progress.
  They don't work yet.
</div>

# NullAuth Specification 0.2 Draft

Authentication mechanisms today have multiple weaknesses and shortcomings. To name a few:

1. You have to trust a third-party with safe-keeping your password.

2. Google and FaceBook are gatekeepers (via Google/Facebook Login) to apps we use everyday.

3. You have to remember multiple passwords.

4. Humans are not good at remembering good passwords.

NullAuth is an authentication scheme which uses Public Key Cryptography to alleviate some of these problems. In the first phase, NullAuth with target apps with technically proficient users, and we hope that good tooling will eventually make it accessible to non-technical users as well. NullAuth is designed to be simple to understand and implement from scratch, and is based on existing, proven methods. The approach outlined here has been previously discussed on the internet, and the main intent is to turn those conversations into an easy-to-understand spec.

Sample projects which implement NullAuth are available under [Examples](https://www.nullauth.org/examples).
To simplify complexity of end-users, [browser extensions](https://www.nullauth.org/downloads) are available for Firefox and Chrome.
Contributions for other browsers are welcome.

## Pre-requisites for using NullAuth for your app

* Generate an RSA 2048-bit key pair for your domain.

* The domain's public key should be available at www.example.com/.well-known/nullauth/public.key

* The website should expose a single url at which it will receive nullauth "commands" (eg: https://www.example.com/nullauth). NullAuth does not mandate what the url should be.

* NullAuth assumes HTTPS everywhere.

## About NullAuth Commands

NullAuth Commands take the general form:

```javascript
command with key1=val1, key2=val2, key3=val3;signature

//example
Login with username=jeswin@nullauth.org, timestamp=1513348513265;SIGNATURE_STRING
```

If any of the values contain a comma, it needs to be escaped with a backslash '\'.

## Flow 1: Creating an account for a user

* User creates an RSA 2048-bit key pair (or uses an existing one) locally. For now, NullAuth [Browser Extensions](https://www.nullauth.org/downloads) make this easier.
  Otherwise users may generate it from the command line or use [NullAuth KeyGen](https://www.nullauth.org/keygen).

* Safekeeping the key pair is the user's responsibility. Eventually, we'll have tools for this.

* Create a form that takes the username, device name, public key and other details needed for creating the account. Device name is needed so that the user can intuitively revoke keys later.

* Server creates an account with the specified username and stores the public key against it

## Flow 2: Login

* The login challenge text takes this form:

```javascript
Login with username=(username@website), timestamp=(utc_milliseconds)

//example
Login with username=jeswin@example.com, timestsamp=1513348513265
```

* The domain (or website) needs to append a signature after a semicolon. Signature is equivalent to hashing the above challenge with SHA256, and encrypting the hash with the domain's private key.

```javascript
Login with username=(username@website), timestamp=(utc_milliseconds);APP_SIGNATURE_STRING
```

* User verifies the app signature, signs challenge with his or her private key, and sends it to the server.

* Server verifies the username, and if the utcmilliseconds is recent enough returns a session token and optional expiry. If expiry is not set, it is left to the app.

```javascript
(expiry = valid_till_time), (token = token);
```

* The session token is used for subsequent access.

## Flow 3: User updates public key

This can be done in two ways.

* Just provide a UI to logged in users for submitting a new public key. Outside this spec.

* Call an app's nullauth url with the following command. This is useful for password or key managers.

```javascript
Update key with username=(username@website), timestamp=(utc_milliseconds), new_public_key=(new_public_key);USER_SIGNATURE_STRING
```

## Flow 4: Access Delegation (like OAuth)

This addresses usecases currently handled by OAuth. Eg: User has an account on docs.example.com (aka provider), and some data stored there. User wants to allow another app publisher.example.com (aka consumer) to access (read and modify) his or her data on docs.example.com.

* An access delegation challenge text takes the form:

```javascript
Grant external access with username=(username@provider), permissions=(comma_separated_permissions), consumer=(consumer), timestamp=(utc_milliseconds)

//example
Grant external access with username=jeswin@docs.example.com, permissions=read,contacts, consumer=publisher.example.com, timestamp=1513348513265
```

* publisher.example.com signs the above message and sends it to the user

```javascript
Grant external access with username=(username@provider), permissions=(comma_separated_permissions), consumer=(consumer), timestamp=(utc_milliseconds);CONSUMER_SIGNATURE_STRING
```

* User verifies if the signature matches that of the consumer (publisher.example.com), signs it, and sends it to publisher.example.com

* publisher.example.com sends signed message to docs.example.com and receives a token.

* The token can be used to access data until an expiry decided by the provider app (docs.example.com).


## Flow 5: Access from Multiple Devices

To access an account from a new device, you need generate a new key pair on the device and add it to the account.
There are two ways to do this.

* The app can provide a way for an already logged-in user to add a new public key and device name

* Call an app's nullauth url with the following command. This is useful for password or key managers.

```javascript
Add key with username=(username@website), device=(device_name), timestamp=(utc_milliseconds), public_key=(new_public_key);USER_SIGNATURE_STRING
```

## Flow 6: Revoking access from a device (Programmatically)

* Call an app's nullauth url with the following command. This is useful for password or key managers.

```javascript
Revoke key with username=(username@website), device=(device_name), timestamp=(utc_milliseconds);USER_SIGNATURE_STRING
```

## Tools for End Users

* We'll create browser extensions for non-technical users to try NullAuth.

Tools and extensions should refuse to sign if the challenge's signature cannot be verified by the consumer's public key.

Consider the following challenge:

```javascript
Grant access with username=(username@providerapp), permissions=(comma_separated_permissions), consumer=(consumerapp), timestamp=(utc_milliseconds);CONSUMER_SIGNATURE_STRING
```

If the consumer-domain's public key cannot verify the above challege, the tool should refuse to sign it.

#### Inputs from:

* dchestnykh in [Reddit thread on /r/crypto](https://www.reddit.com/r/crypto/comments/7k0uib/nullauth_a_proposal_for_decentralizing/)

* Anshum Garg
