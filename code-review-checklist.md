# Code Review Checklist

This is just a base line of some things to look for when doing code reviews.

* [Table of Contents](#table-of-contents)
    * [Backwards Compatibility](#backwards-compatibility)
    * [Coding Conventions](#coding-conventions)
    * [Commit Messages](#commit-messages)
    * [Copyright](#copyright)
    * [Documentation](#documentation)
    * [Don't Repeat Yourself (DRY)](#dont-repeat-yourself-dry)
    * [Migrations](#migrations)
    * [Patch Size &amp; Structure](#patch-size--structure)
    * [Style](#style)
    * [Testing Coverage](#testing-coverage)

### Backwards Compatibility

* Does the change break backwards compatibility from an end user perspective?

> If this is the case, we should be moving API versions.

* Does the change break backwards compatibility with an internal API?

> We should make sure that if a common piece of project code has an API change,
> subsequent patches to update code that uses that API should be proposed. An
> example might be if we changed the `keystone.notification` API, thus changing
> how notifications are sent. This would require several patches to other
> Keystone APIs to consume the change.

### Coding Conventions

* Does the patch follow general project coding conventions?

> Each open source project has it's own conventions. These often vary between
> OpenStack project even. After enough exposure to a project's source, these
> conventions are recognizable.

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

> Bug links can be found in the footer of the commit message. Reviewing the
> linked bug both before reviewing the patch, and after, is a good way to gain
> additional context around what the author is trying to accomplish and ensure
> the patch addresses the issue documented in the bug report.

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

> If so, that can be bad. Only members of the OpenStack Foundation can add
> OpenStack copyrights. Company copyrights should be fine and it's on that
> company to maintain their copyrights, especially if they ship products with
> that code. [Copyright
> Guidelines](https://wiki.openstack.org/wiki/Documentation/Copyright) can
> change at the discretion of the OpenStack Foundation, but it's useful to stay
> up on them.

### Documentation

* Does the change require a change to end-user (deployer/operator) docs?

> If so, check to see if there is a patch that's been proposed to add it. If
> not, then we should be sure to accurately document what is change and how to
> deal with it.

* Does the change require an update to developer documentation?

> Developers need documentation too! This can apply to patches that change tool
> chains, testing automation, development tools, etc. It's harder to be an
> effective contributor when the documentation you're using to on-board is
> out-of-date. We can help mitigate this by ensuring changes to tooling include
> developer documentation.

* If the change includes a public API, are those methods doc string'd?

> Capturing sound information like variable types, purpose, and expected
> behavior in documentation strings is extremely helpful. It helps reviewers
> understand the intent of the code being proposed and it helps with
> maintainability as the code-base changes.

* Is there any confusing logic that would benefit from an inline comment?

> If you've read a piece of code several times and still don't understand
> exactly what it's doing, it could probably benefit from an inline comment
> from the author. If you do understand what is being done, not every other
> reviewer might. Maybe we can use the opportunity to simplify the code into
> something that is more readable.

### Don't Repeat Yourself (DRY)

* Does the patch have redundant code with other parts of the same patch or
  project?

> Duplicate code can be a maintainability nightmare. Let's consolidate and
> share where we can. Maybe the author doesn't know there is a module that does
> exactly what they need. If they do, they might have a good reason for not
> going with that implementation. Does it make sense to find a way to use the
> same code. Who else would be interested in sharing that code?

### Migrations

* Does the patch need a migration of any kind?

> If so, how painful will it be for ops? Can it be improved? We can provider
> migration Advil in multiple ways, like tooling and documentation. Proactively
> documenting a migration change will result in less bugs opened later!

### Patch Size & Structure

* Is the patch greater than 500 LOC?

> Does this patch feel far too complicated to review all at once? Should it be
> broken down into a series of patches? Breaking a monolithic patch into
> several bite-sized patches makes them more reviewable.

### Style

* Is the patch Pythonic?

> Is there a way to make the code more efficient in a Pythonic way?

### Testing Coverage

* Are there unit tests provided in patches where unit tests make sense?

> If the patch consists of code, we should ensure that it is tested.

* Do the tests actually test new code that's being added?

> Don't forget to test the tests! If you pull the patch down locally, can you
> make the tests fail? If you revert the change locally and run the tests, do
> they still pass? If so, then the tests probably need some more work.

* Does the patch introduce test coverage regression?

> Coverage can be checked by running `tox -e cover`. The coverage report can
> then be inspected locally by running `open keystone/index.html`.

* Does the patch require additional testing added to Tempest?

> If we are delivering something that effects how other services interact with
> Keystone, chances are we might need a Tempest test to enforce that behavior.
> An example of this would be Tempests tests and validate operations used with
> Keystone trusts. When in doubt, drop into the `#openstack-qa` channel on
> Freenode and ask the Tempest folks.  Most of these questions can be answered
> by checking out the change locally with `git review -d <patch-number>` and
> inspecting coverage reports.

* Do the tests added have add complex setup?

> With large projects that move fast, the testing footprint grows a lot. It
> becomes very easy for tests to get added with niche setup procedures. If a
> patch has specific setup methods for tests, can they be consolidated into
> something that already does that setup?
