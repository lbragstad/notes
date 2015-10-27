# Deep Dive into Keystone Tokens and Lessons Learned

Date: 2015-10-27 11:15

## Presenters

  - Priti Desai (IBM)
  - Brad Pokorny (Symantec)

## Notes

- what is the best format?
  - uuid: simplest, version 4 tokens
    - all information is stored in the token backend
    - when the token is given to a service, the service has to to back to
      keystone to validate it
    - on validate the information is pulled from the backend
    - revoking a uuid token requires removing the token from the persistent
      store
      - leverages revocation events to revoke a token, creating a new event for
        the token that needs to be revoked
    - but what happens in muliple data centers?
      - when using the same uuid token in a different data center, you'll need
        to wait until the token is replicated, if you have replication set up
      - if replication isn't set up, you need to get a new token for each
        region
      - bearer token
    - pros:
      - easy to set up
      - copy-pastable
    - cons:
      - must be persisted
      - multi-region requires backend replication

  - pki/z: crytographically signed, x509 standards, CMS, pkiz is compressed
    - requires:
      - signing keys
      - signing certificate
      - certificate authority
    - all certs have to be given to keystone via a configuration file
      (batteries required)
    - everything about the auth context is stored as json and signed
    - made url-safe
    - tokens are still persisted in the backend
    - much longer in length
    - what happens on validation against keystone server?
      - keystone generates a hash and validates the integrity of the token
      - very similar to the uuid validation flow
    - how to i revoke a pki token?
      - revocation lists have to be leveraged, just like uuid
    - what about multi-data centers?
      - the signing keys have to be distributed against multiple keystone nodes
        in order to validate those tokens
      - works across multiple datacenters
      - must have token backend replicated
      - you have to assume the cost of replication
      - they are **real** big!
      - still a bearer token
    - pros:
      - multi-region is possible
    - cons:
      - must be persisted
      - lot of setup required

  - fernet: cryptographic authentication, symmetric key encryption
    - requires a key repository to create and validate tokens
    - there are different types of signing keys
    - primary, secondary, and staged keys
    - what is in the token?
      - token version, timestamp, iv, cipher text, hmac
    - no need to store it anywhere, everything needed to validate the token is
      **in** the token
    - requires a round trip to the keystone server to validate
    - requires revocation events in order to revoke the token
      - this can cause performance issues!
      - token validation time goes up
        - this isn't good
    - better support in multiple datacenters
    - still a bearer token
    - pros:
      - no persistence
      - reasonable in size
      - multiple data centers
    - cons:
      - token validation impacted because everything has to be rebuilt

  - what is the best choice for multiple data-centers?
    - fernet; but you have to watch the revocation events and validate times

- from a horizon perspective
  - tokens are logged for each user
    - if horion is compromised, the token is compromised
  - tries to do as much re-use as possible, stored in sessions
  - boot back to login bug
    - user is booted back to login after entering valid creds
    - issue with cookie token storage
    - use memcache instead
  - use token hashing
  - service regions
    - uuid, pki and pkiz don't work in multi-region deployments, without
      backend replication
    - fernet tokens work in multi-region deployments

- fernet tokens and horizon
  - works out of the box on liberty and master
  - needs patches for kilo

- will fernet solve all the problems?
  - smaller and non-persistence is a plus
  - there are performance issues
    - we have patches up for review to increase the performance of fernet
      tokens
      - [role assignment caching](https://review.openstack.org/#/c/215715/)
      - [catalog caching](https://review.openstack.org/#/c/215212/)
  - seamless authentication across regions


## Questions

Q = Question
A = Answer
S = Suggestion

Q: If the token is not persisted, how is the token tied to the user?

A: the payload/metadata about the token is "persisted" in the token. This is so
the keystone service can find since it has the ability to decrypt the tokens.

S: UUID tokens can be used with federation and SAML IdP and replication.

Q: If you're doing a key rotation, how do you sync across datacenters?

A: Use a configuration management tool, like ansible.

Q: Can we use kubernetes to use keystone for authentication?

A: Unsure

Q: Can you use fernet and trusts

A: Yep

Q: Can I validate tokens with different identity backends in different regions?

A: Nope, not unless you replicate your keystone backend across regions. The
keystone server in the other region needs to be able to understand the data
packed in the token.

Q: Is there a mechanism to clean up the revocation events?

A: Yes, you can setup a separate job to delete old revocation events.

Look into adding documentation around key rotation and validation across
regions with replication.
