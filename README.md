```
/**
 * @author: KentProjects <developer@kentprojects.com>
 * @license: Copyright KentProjects
 * @link: http://kentprojects.com
 */
```

# Queue

### A simple system for queuing large numbers of emails and sending them at a reasonable rate!

Here lies a simple queue designed to hold a large number of emails, and slowly run through them, executing them one by
one in a sensible order in a sensible time.

----

## Specification

Since we are going to be sending a large number of emails to keep people up-to-date with their project, we need a simple
First-In-First-Out queue system to buffer these emails, otherwise we will end up spamming our email provider / SMTP
service, which won't be good!

### REST API

This system doesn't require a frontend, but should offer a REST API to add items to the queue and to check the queue's
state.

So, a simple `GET` request returning information on the queue size:

```http
GET / HTTP/1.1
Token: 00a3b76189569a6119e834a2eae5e8cb
```

This should return a JSON object containing information on the queue.

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

```json
{
  "size": 432,
  "last_join": "3 minutes ago",
  "last_join_timestamp": 1420471163
}
```

And a simple `POST` request endpoint should exist, allowing information to be added to the queue.

```http
POST / HTTP/1.1
Content-Length: 512
Token: 00a3b76189569a6119e834a2eae5e8cb
```

```json
{
  "handler": "postmark",
  "data": {
    "From": "ProjectBot <bot@kentprojects.com>",
    "To": "James Dryden <james.dryden@kentprojects.com>",
    "Bcc": "bot-backup@kentprojects.com",
    "Subject": "Matt House has invited you to his group on KentProjects",
    "TextBody": "Hello James!\nMatt House has invited you to his group on KentProjects...",
    "ReplyTo": "reply@example.com"
  }
}
```

This should return a simple JSON object confirming the item was added to the queue.

```http
HTTP/1.1 201 Created
Content-Type: application/json
```

```json
{
  "current_size": 432,
  "last_join": "12 minutes ago",
  "last_join_timestamp": 1420471999,
  "last_sent": "1 minute ago",
  "last_sent_timestamp": 1420472648
  "sent_today": 2701,
  "sent_yesterday": 11721
}
```

The REST API should also provide some form of authentication, so it can be run on a public-facing port without the worry
of outside users interfering with it. In the examples above, I have used a `Token` header, because that seemed better
than having a `GET` / `POST` parameter containing a token, however this is entirely up to you. As long as it makes sense
and the keys can be changed (ideally by use of an `.ini` file) then that's fine.

### Handlers

If you noticed in the `POST` request example, there's a field called `handler` and a field called `data`. The `handler`
field represents a specific action that the queue will execute when this item is first in the queue, and the `data` is a
simple JSON-encoded object that will be passed on to the `handler`.

This queue system should be written in such a style that new handlers can be written with little or no difficulty.

#### Example Handler

An example `handler` is `postmark`. Postmark offer a simple API to assist in sending transactional emails, and [their
online documentation](http://developer.postmarkapp.com/developer-api-email.html) details how their API receives JSON-
encoded POST data and send the email onwards! In this case, the `postmark` handler would simply need to send the `data`
in a POST request to Postmark's API, handling all authentication and errors.

### Development

This queue system can be written in any server-side language required, as it will run on it's own independent server
that will require configuring either way. You should use `vagrant` to test the queue.

### Shutdown

Ideally, the queue should be able to maintain it's state during shutdown / reboot operations, for example if we deploy a
new version of the queue it should be able to be shutdown, updated, and start up without any previously queued items
being lost in the process (items attempted to be queued via the REST API can be lost, since we expect whilst the queue 
is being updated the REST API will be unavailable).

----

## Thoughts

This queue might benefit from storing it's data in a document-driven database like MongoDB or even a cache platform like
[Redis](http://redis.io). If all else failed we could use something like [Kue](https://github.com/learnboost/kue) and
run it on a handmade NodeJS server, but I'd rather we built something that allows us to extend the queue as we see fit!

----

### Good luck!

This message will (probably not) self-destruct in five seconds.