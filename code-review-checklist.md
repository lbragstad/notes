# Code Review Checklist

This is just a base line of some things to look for when doing code reviews.

Table of Contents
  * [Backwards Compatibility](#backwards-compatibility)
  * [Coding Conventions](#coding-conventions)
  * [Commit Messages](#commit-messages)
  * [Copyright](#copyright)
  * [Documentation](#documentation)
  * [Don't Repeat Yourself (DRY)](#dont-repeat-yourself-dry)
  * [Exceptions](#exceptions)
  * [Migrations](#migrations)
  * [Patch Size &amp; Structure](#patch-size--structure)
  * [Reviewing Previous Comments](#reviewing-previous-comments)
  * [Style](#style)
  * [Testing Coverage](#testing-coverage)

### Backwards Compatibility

* Does the change break backwards compatibility from an end user perspective?

> If this is the case, we should be moving API versions.

* Does the change break backwards compatibility with an internal API?

> If a common piece of project code has an API change, subsequent patches to
> update the code that uses that API should be proposed. An example might be if
> we changed the `keystone.notification` API, thus changing how notifications
> are sent. This would require several patches to other Keystone APIs to
> consume the change.

### Coding Conventions

* Does the patch follow the project's coding conventions?

> Each open source project has its own coding conventions. These often vary
> between OpenStack projects. After enough exposure to a project's source,
> these conventions are recognizable.

* Does the patch follow OpenStack coding conventions?

> Some conventions are really common and are usually
> [documented](http://docs.openstack.org/developer/hacking/). Understanding
> the common ground, with respect to coding conventions, between projects is
> very useful and helps maintain consistency with the rest of the community.

* Is a new coding convention being introduced?

> This stuff may seem nit picky to comment on but it's more often than not an
> easy fix, and it's easier to fix this up front than propose subsequent patch
> sets. Multiple conventions can be confusing for new developers learning the
> code base.

### Commit Messages

* If the patch is fixing a bug, does it link to a bug?

> Bug links can be found in the footer of the commit message. Carefully reading
> the linked bug report before reviewing the patch, and after, is a good way to
> gain additional context around what the author is proposing.

* If the patch is delivering a feature, does it link to a spec or blueprint?

> This follows the same reasoning as bugs. Reviewing code that is delivering a
> feature without understanding the use case is hard. You can always gain
> context about the proposed feature by reading the blueprint and/or
> specification document. If a blueprint isn't linked in the commit message and
> the change isn't trivial, leave a comment.

* Does the commit message make sense before and after reviewing the patch?

> This might be a good indication that either the commit message or the
> proposed patch needs to be clearer. If something doesn't make sense when
> you've read it several times, it probably needs to be revisited. The "no
> question is dumb" rule applies here because if you're having a hard time
> understand it, chances are someone else is too! Commit messages should be
> detailed enough to describe **why** we should be accepting the proposed
> change.

### Copyright

* Does the patch add an OpenStack Foundation copyright?

> Only members of the OpenStack Foundation can add OpenStack copyrights.
> Company copyrights should be fine and it's usually on that company to
> maintain their copyrights. [Copyright
> Guidelines](https://wiki.openstack.org/wiki/Documentation/Copyright) can
> change at the discretion of the OpenStack Foundation, but it's useful to
> review them regularly.

### Documentation

* Does the change require additional end-user (deployer/operator)
  documentation?

> If so, check to see if there is a patch proposed to add docs for it. If not,
> then one should be proposed to accurately document what is changing and how
> to deal with it.

* Does the change require an update to developer documentation?

> Developers need documentation too! This can apply to patches that change
> developer tools, testing automation, etc. It's harder to be an effective
> contributor when the documentation you're using to on-board is out-of-date.

* If the change includes a public API, does it have doc string'd?

> Capturing sound information like variable types, purpose, and expected
> behavior in documentation strings is extremely helpful. It helps reviewers
> understand the intent of the code being proposed and it helps with
> maintainability as the code-base changes.

* Is there any confusing logic that would benefit from an inline comment?

> If you've read a piece of code several times and you still don't understand
> exactly what it's doing, it could probably benefit from an inline comment.
> Even if you do understand what is being done, it might not be clear to every
> other reviewer.

### Don't Repeat Yourself (DRY)

* Does the patch have redundant code with other parts of the same patch or
  project?

> Duplicate code can be a maintainability nightmare. Let's consolidate and
> share where we can. Maybe the author doesn't know there is another module
> that does exactly what they need. If they do, they might have a good reason
> for not going with that implementation. Does it make sense to find a way to
> use the same code. Who else would be interested in sharing that code? Is this
> something that would benefit other OpenStack project? What about other
> open-source project?

### Exceptions

* Are exceptions handled properly?

> If the patch excepts exceptions, are they handed properly? If you see
> something like
>```
>try:
>    # something
>except Exception:
>    pass
>```
> your Spidey senses should be going off. We should make sure we deal with
> exceptions if we catch them. If there is nothing to deal with, then are we
> logging anything? If there is nothing to log, then is there a comment
> explaining why?

### Migrations

* Does the patch need a migration of any kind?

> If so, how painful will it be for ops? Can it be improved? As developers, we
> can provider migration-Advil in multiple ways, like tooling and
> documentation. Proactively documenting a migration impact will result in less
> bugs opened later!

### Patch Size & Structure

* Is the patch greater than 500 LOC?

> Does this patch feel far too complicated to review all at once? Should it be
> broken down into a series of patches? Breaking a monolithic patch into
> several bite-sized patches makes them more reviewable.

### Reviewing Previous Comments

* Have you commented on this patch before and where your comments answered?

> It's good to go back and see what caught your eye previously. Maybe the
> author, or another reviewer, responded to your comment. It's also important
> to make sure the point being raised maintains integrity throughout the review
> cycle. An example being:
>
> * you made a comment on patch set 2
> * the author addressed your comment in patch set 3
> * tons of things change between patch set 3 and 45 and your comment from
>   patch set 2 is no long valid or regresses on accident
> * patch merges
>
> We can prevent that from happening by reviewing previous comments.

* What have other reviewers commented on?

> This is a great way to learn new tips and tricks from other developers!

### Style

* Is the patch Pythonic?

> Is there a way to make the code more efficient in a Pythonic way?

### Testing Coverage

* Are there unit tests provided in patches where unit tests make sense?

> If the patch consists of code, we should ensure that it is tested.

* Do the tests actually test the code that's being added?

> Don't forget to test the tests! If you pull the patch down locally, can you
> make the tests fail? If you revert the change locally and run the tests, do
> they still pass? If so, then the tests probably need some love.

* Does the patch introduce test coverage regression?

> Coverage can be checked by running `tox -e cover`. The coverage report can
> then be inspected locally (`open keystone/cover/index.html`).

* Does the patch require additional testing added to Tempest?

> If we are delivering something that affects how other services interact with
> Keystone, chances are we might need a Tempest test to enforce that behavior.
> An example of this would be Tempests tests and validate operations used with
> Keystone trusts. When in doubt, drop into the `#openstack-qa` channel on
> Freenode and ask the Tempest folks.

* Do the tests added add unnecessary complex setup?

> With large projects that move fast, the testing footprint grows a lot. It
> becomes very easy for tests to get added with niche setup procedures. If a
> patch has specific setup methods for tests, can they be consolidated into
> something that already does that setup? Can the setup logic be pulled out
> into a testing mixin and shared?
