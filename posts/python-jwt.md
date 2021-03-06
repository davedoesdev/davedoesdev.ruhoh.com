---
title: "JSON Web Tokens in Python"
date: '2013-10-12'
description: 'python-jwt: A Python module for generating and verifying JSON Web Tokens'
categories:
tags: [python, crypto, JWS, JWT, python-jws, python-jwt]
---

My [previous article](/json-web-signatures-on-node-js) introduced a Node.js
module, [node-jsjws](https://github.com/davedoesdev/node-jsjws), for performant
generation and verification of [JSON Web Signatures](http://tools.ietf.org/html/draft-ietf-jose-json-web-signature-13) and [JSON Web Tokens](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html).

Brian J Brennan's [python-jws](https://github.com/brianloveswords/python-jws) is a
nice module for generating and verifying JSON Web Signatures in Python. I've
already written some unit tests for __node-jsjws__ which show that the JSON Web
Signatures it generates can be verified by __python-jws__ and vice versa.

Note that I had to make [some minor changes](https://github.com/brianloveswords/python-jws/pull/5) to __python-jws__ in order to add
support for the [RSASSA-PSS](http://tools.ietf.org/html/draft-ietf-jose-json-web-algorithms-13#section-3.5) signature algorithms (__PS256__, __PS384__ and
__PS512__).

Interoperability between __node-jsjws__ and __python-jws__ is useful because it
means a Web site written in Python can send a JSON Web Signature to another site running on Node.js, for example.

I wanted to be able to do the same with JSON Web Tokens: send a token from a site running on Google App Engine, for example, to a service running on Node.js.

# Introducing [python-jwt](https://github.com/davedoesdev/python-jwt)

__python-jws__ does have a JWT example, [minijwt](https://github.com/brianloveswords/python-jws/blob/master/examples/minijwt.py), but as its name suggests it's a limited implementation of JSON Web Tokens.

I've added the following things to the JWT header and turned __minijwt__ into a standalone module, __python-jwt__:

- Expiry date and time of the token (__exp__).
- Date and time at which the token was generated (__iat__).
- Date and time from which the token is generated (__nbf__).
- A unique identifier for the token (__jti__).

__exp__, __iat__ and __nbf__ are checked against the current time when a token
is verified.

I also added support for the [none](http://tools.ietf.org/html/draft-ietf-jose-json-web-algorithms-14#section-3.6) signature algorithm (i.e. an empty signature).

## Example

Here's a simple example using a key generated by [PyCrypto](https://www.dlitz.net/software/pycrypto/):

```python
import jwt, Crypto.PublicKey.RSA as RSA, datetime
key = RSA.generate(2048)
payload = { 'foo': 'bar', 'wup': 90 };
token = jwt.generate_jwt(payload, key, 'PS256', datetime.timedelta(minutes=5))
header, claims = jwt.verify_jwt(token, key)
for k in payload: assert claims[k] == payload[k]
```

The expiry time of the token is set to 5 minutes.

The [API documentation](http://rawgit.davedoesdev.com/davedoesdev/python-jwt/master/docs/_build/html/index.html) is linked to from the [python-jwt homepage](https://github.com/davedoesdev/python-jwt). __python-jwt__ comes with
a full set of unit tests (including interoperability with __node-jsjws__) and some
benchmarks.

I've decided not to compare benchmark results with __node-jsjws__ because I
don't want to get into comparing Node.js with Python.
