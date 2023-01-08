---
title: Notes from reading The Little Book of OAuth 2.0 RFCs
noindex: true
---

# Notes from reading [The Little Book of OAuth 2.0 RFCs](https://oauth.net/books/#little-book-of-rfcs)

## Security

I didn’t read the two documents of security recommendations. I started reading the first one and realised that in order to understand the attacks being described, I needed a better grasp of:

- The front channel vulnerabilities that need to be worked around – CSRF attacks, s
- The security mechanisms provided by a user agent – same origin policy and CORS (see [Fetch spec](https://fetch.spec.whatwg.org))
- Some of OAuth’s security features such as `state` parameter and redirect URLs and how they’re meant to be used and checked.

So that’s something I’ll come back to.

## Other things

- Look at mentioned RFCs eg SAML, or RFC 6125 (how to use X.509 certificates with TLS).
- Lots of stuff in OAuth 2 is about protecting stuff that goes in the front channel (e.g. URLs) (which, as more and more extensions are added, becomes more stuff) - see Transactional Authorization talk at Identiverse (making the transactions implicit in OAuth 2 implicit). These days it’s known as GNAP - Grant Negotiation Authorization Protocol.
- A lot of the authorisation server metadata RFC 8414 refers to specs that aren’t in the book and to JWT specs.
- RFC 5646 tags for identifying languages (BCP 47)
- All the IANA registers that these specs create – interested to learn more about IANA and ICANN.
- OpenID Connect I’m particularly interested in learning about because it’s what [GOV.UK One Login](https://www.sign-in.service.gov.uk/) uses .
