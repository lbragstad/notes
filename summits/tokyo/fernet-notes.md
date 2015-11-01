# Notes and Questions on Fernet

Collected throughout Thursday, October 29th, 2015

## Participants

  - Alexander Makarov (Mirantis)
  - David Stanek (Rackspace)
  - Kevin Carter (Rackspace)
  - Lance Bragstad (Rackspace)

## Notes

Since the landing of the `fernet` token provider, the Fernet implementation,
and use, in keystone has raised plenty of questions. The following questions
have been gathered from the OpenStack Tokyo Summit. I'd like to be able to
consolidate these questions, and answers, into formal keystone documentation.

### What is the recommended way to rotate and distribute keys?

You will only need to distribute keys when you have multiple keystone nodes in
your deployment. There is a rotation mechanism built into `keystone-manage
fernet_rotate`. This rotation mechanism does not do any sort of distribution of
keys. The distribution of keys is best handled by configuration management.

### Must you run your rotation and distribution from the same node?

Nope, as long as all nodes sync state at least once, each node should be able
to rotate and push keys to all other nodes in the cluster.

### What do I need to do to add new keystone nodes to my deployment?

Treat your keys like super secret configuration files. Before a node is allowed
to join an existing cluster, issuing and validating tokens, it should sync key
repository state with the rest of the cluster.

This is done through distribution from an existing node in the cluster.

### How should I approach key distribution?

You could build your key distribution into your deployment's exisiting
configuration management system. In all reality, something as simple as `rsync`
is a viable option.

Since key rotation is a single operation, it's easiest to do this on a single
node and verify the rotation took place properly. The fact that you have a
staged key allows you the ability to verify the rotation before syncing state
to the other nodes. Key distribution should be something that runs until it
succeeds. If key distribution does succeed and you run a subsequent
distribution, it shouldn't have any ramifications. The following operations
should help illustrate:

```
Pick a keystone node in the current mesh
Rotate Fernet keys
  - Did it fail?
    - If yes, investigate issues with that particular box. Fernet tokens are
      small and the operation for adding a new key, promoting the staged key,
      and demoting the current primary are trivial. There shouldn't be much
      room for error in this operation.
    - If no, the key rotation happened successfully. You should be good to
      distribute the new key set.
Distribute the new key repository
  - Did is fail?
    - If yes, try distributing again. The nodes that have not received the
      newest key set should still be able to validate tokens because they have
      the old staged key, which is now the new primary key in the new key set.
    - If no, you're good to go. You've successfully rotated a new key into the
      mix and pushed the new key set into the mesh.
```

### How long do you keep your keys around?

Fernet tokens are only secure as long as you rotate the keys creating them.
Since the penalty of rotation is low, an operator could err on the side of
security and rotate weekly, daily, even hourly.

### So, how does a staged key help me again?

The staged key is special. Fernet keys have a natural life-cycle. Each key
starts as a staged key, is promoted to be the primary key, and then demoted to
the secondary key. Only the primary key can actually create new keystone
tokens. The staged, primary, and secondary keys can all validate tokens. The
staged key is special given the order of events and the attributes of each key.
The staged key is the only key in the repository that **hasn't** had a chance
to create any tokens yet, but it's still allowed to decrypt tokens. This gives
operators the chance to perform a key rotation on one keystone node, and
distribute the new key set over a span of time. This doesn't require the
distribution to take place in an ultra short period of time for fear of not
being able to validate tokens created by a different node in the cluster.

### Where do I put my keys?

Your `key_repository` will tell keystone where to look for Fernet keys. This
could be anywhere, really. Currently, keystone supports a file-backed
repository. It's not to say that an interface could pull keys out of another
storage system. This is similar to the "fast", "cheap", or "good" argument,
pick two!

Storing your keys on disk, in `/etc/keystone/fernet-keys/` for example, has the
advantages of being fast and scalable. Each keystone node has its own local
repository of Fernet keys. The downside of this approach is that it requires
coordination when doing a rotation. But, this coordination can be orchestrated
using configuration management like `ansible`, `puppet`, `chef`, etc. Also, the
timing between key rotation and distribution doesn't require tight coupling
because of the usage of a staged key.

file-backed:
  + fast (+)
  + scalable (+)
  + requires coordination (-)

Storing keys in a database and passing a connection string as the
`key_repository` location has the ability to mitigate the pain of distribution.
If all your keystone nodes are reading keys from the same location, once you
rotated, you've also distributed the new key set. The downside would be that
retrieving keys can be subject to database latency. It is also requires a
database connection.

database:
  + distribution can be handled auto-magically (+)
  + requires a database connection (-)
  + possibly slow (-)

### Do Fernet tokens still expire?

Yep, Fernet tokens can expire just like keystone's other token formats.

### Why Fernet over UUID?

Even though Fernet tokens operate very similar to UUID tokens, they do not
require persistence. Meaning, the database no longer suffers bloat as a
side effect of authentication.

### Why Fernet over PKI or PKIZ?

The arguments for using Fernet over PKI and PKIZ remain mostly the same as
UUID. PKI and PKIZ tokens still require persistent storage. However, Fernet
tokens can't be validated by the client. If you're leveraging distributed
signing for PKI and PKIZ tokens, you won't be able to continue that flow with
Fernet, simply because keystone doesn't share it's key repository with clients.

### Performance

Fernet tokens consist of only the necessary bits needed to rebuild
authorization context. These bits are kept to a minimum since it keeps the
token length smaller, and more manageable. A side effect is that keystone must
rebuild the user's catalog when it validates a token. Rebuilding context
requires more cycles, and more backend reads. This didn't occur with UUID
tokens since the authorization context was cached in the token backend on
creation and used when validating the token, meaning it didn't have to be
reconstructed by hand.

Keystone has several patches in-flight to improve the performance of
constructing a user's catalog. The current patches proposed aren't specific to
Fernet and will actually benefit most situations requiring a user's catalog or
retrieving their role assignments on a project.

### What happens when I need to revoke all keys?

```
ZOMG! I must revoke all tokens now!
- Operator
```

Unfortunately, this happens. Fortunately, this is easy to fix with Fernet. In
the event an operator needs to take the "scorched earth" path, all they need to
do is remove their current key repository, create a new one, and redistribute
it.

```
$ rm -rf /etc/keystone/fernet_keys
$ keystone-manage fernet_rotate
```

The distribution of the keys can be left to the deployer, depending on their
configuration management system.

All pre-existing tokens being validating against keystone will be unauthorized,
but that was probably the desired effect when you decided to take the "scorched
earth" path. This will force all clients to get a new token, using new keys in
the key repository.
