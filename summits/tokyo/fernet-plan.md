# The Future of Fernet

During the OpenStack Mitaka design summit, we had several discussions on
getting Fernet to be set as keystone's default token provider. The goal of this
post is to document that path and see what it looks like for keystone and
related projects.


### What is needed to get Fernet set as the default token provider in Keystone?

We proposed a [patch](https://review.openstack.org/#/c/195780/) that sets the
default token provider in Devstack to Fernet. This exposed gaps in our
implementation that needed to be addressed. Most of those bugs could be ignored
for this conversation as they were relatively minor fixes, internal to
keystone.

#### Patch Tempest to wait one second on password changes.

Ok, hold up. With the introduction of fernet, we also found a special case with
how fernet interacts with revocation events in certain environments. For
example, when running keystone unit tests, revocation events have the ability
to store sub-second precision because unit tests are backed by sqlite. The
sqlite implementation allows the `DATETIME` sql format to store sub-seconds in
the format. When playing with things using MySQL, that sub-second precision is
lost on some versions of MySQL. I believe subsecond
[support](http://dev.mysql.com/doc/refman/5.7/en/fractional-seconds.html) was
added in MySQL 5.7, but don't quote me. Regardless, at the time of this
writing, we can't enforce a suitable version of MySQL to be installed in
upstream gate tests.

##### So, why is this a problem?

Let's say that you enter the threshold of a new second and create a Fernet
token. The token creation of that token will be rounded down to the beginning
of that second. Now, let's change our password in that same second. The
revocation event's `issued_at` time will be truncated once it enters the data
layer. If we go get a new token, still within that same second, we will have
two tokens and a revocation event, all created at the same at time. This is all
due to truncation. When we attempt to validate our latest token it will have
the same creation time as the `issued_at` time of the revocation event,
resulting in a 404 or 401, despite the fact that it was actually created after
the password was changed.

There are several cases like this as you mix and match different combinations
of a revocation backend that do, or do not, support sub-second precision with
Fernet tokens. We've documented some of these cases in keystone [unit
tests](https://review.openstack.org/#/c/227995/).

The hack-tastic fix to get around this initially is to
[patch](https://review.openstack.org/#/c/231191/) Tempest. The plus is that
there hasn't been any operator feedback saying this is an "important" case. The
right long term solution is to separate keystone's reliance on the `DATETIME`
format in SQL, and address the lack of sub-second precision in the Fernet
specification.

#### Patch keystone to remove all `DATETIME` MySQL types.

Since there is such inconsistency in various `DATETIME` formats within SQL, why
not remove it and replace it with some else? One way we could do this would be
to replace `DATETIME` with `INT` and store timestamps instead. This would allow
us to get sub-second precision.

This would require a database migration as well as a layer that translates
timestamps to the time strings that keystone expects.

#### Add sub-second precision to Fernet tokens.

Now we approach the second part of the problem, and add sub-second precision to
Fernet. Since keystone doesn't actually control the [Fernet
specification](https://github.com/fernet/spec/blob/master/Spec.md), we'd have
to do all this upstream. This would require changing the Fernet specification
as well as bumping the version of `cryptography` required by keystone after the
implementation is delivered.

We could also look into carrying the microseconds somewhere in the Fernet
payload. This would be a hacky fix, but it would work until we can actually get
feedback from the upstream Fernet specification maintainers.

#### Revert patch to Tempest to wait one second on password changes.

Since we have a data layer that understands things in sub-second format and
tokens that contain sub-second precision, we should be able to run the use case
above in less than a second. So, remove the `time.sleep(1)` from the Tempest
tests!

### What's next?

#### Code Removal

We have Fernet as the default token provider. Now we can deprecate the PKI and
PKIZ token providers, possibly the UUID provider if we're feeling spicy. This
would be cool because we would be on the path to removing a ton of code,
consolidating test cases, etc.

#### Continued Performance

We already have some patches in place to speed up token creation and validation
as a result of introducing Fernet. These patches include caching of the
[catalog](https://review.openstack.org/#/c/215212/) and [role
assignments](https://review.openstack.org/#/c/215715/). I'd like to continue
this pattern in the future. Note that the above fixes would actually improve
performance outside of the Fernet-specific cases.
