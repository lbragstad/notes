# Federation at Pod

Date: YYYY-MM-DD HH:MM

## Moderators/Presenters

  - Adam Young (RedHat)
  - David Stanek (Rackspace)
  - Dolph Mathews (Rackspace)
  - George Silvis (MOC)
  - Jamie Lennox (IBM)
  - Lance Bragstad (Rackspace)

## Notes

Discussion about shadow user table for federation.

what problems does it solve?
- for fernet federated users, write them to the shadow user table
  - user id
  - idp id
  - groups ids
  - everything that is dynamic in the federated fernet payload needs to be
    persisted in a shadow user table

what are the benefits of this?
- is shortens the fernet payload to be a sane length for all federated use
  cases
- it makes it a regular user again, assign roles to federated users directly
- it allows you to support federated users easier

how do you manage the same user with multiple creds from different identity
providers?

TODO: talk to marek about how/why we need idp_id and protocol persisted in the
fernet token format

what would be the issues?
- your back to persisting user information
  - leads to a cleanup issue
  - jamielennox doesn't think this will be problem because you should be
    cleaning up on project deletion, not user deletion

what are the needs?
- authentication can't be tied to the fact they are stored in the backend
- not allowed to store a password


## Questions

Q = Question
A = Answer
S = Suggestion

Q:

A:


Q:

A:
