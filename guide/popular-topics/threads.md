# Threads

Threads can be thought of as temporary sub-channels inside an existing channel, to help better organize conversation in a busy channel.

## Thread related gateway events

::: tip
You can use the <DocsLink path="class/ThreadChannel?scrollTo=isThread" type="method" /> type guard to make sure a channel is a <DocsLink path="class/ThreadChannel" />!
:::

Threads introduce a number of new gateway events, which are listed below:

- <DocsLink path="class/Client?scrollTo=e-threadCreate" />: Emitted whenever a thread is created or when the client user is added to a thread.
- <DocsLink path="class/Client?scrollTo=e-threadDelete" />: Emitted whenever a thread is deleted.
- <DocsLink path="class/Client?scrollTo=e-threadUpdate" />: Emitted whenever a thread is updated (e.g. name change, archive state change, locked state change).
- <DocsLink path="class/Client?scrollTo=e-threadListSync" />: Emitted whenever the client user gains access to a text or news channel that contains threads.
- <DocsLink path="class/Client?scrollTo=e-threadMembersUpdate" />: Emitted whenever members are added or removed from a thread. Requires `GUILD_MEMBERS` privileged intent.
- <DocsLink path="class/Client?scrollTo=e-threadMemberUpdate" />: Emitted whenever the client user's thread member is updated.

## Creating and deleting threads

Threads are created and deleted using the <DocsLink path="class/ThreadManager" /> of a text or news channel.
To create a thread you call the <DocsLink path="class/ThreadManager?scrollTo=create" type="method" /> method:

<!-- eslint-skip -->

```js
const thread = await channel.threads.create({
	name: 'food-talk',
	autoArchiveDuration: 60,
	reason: 'Needed a separate thread for food',
});

console.log(`Created thread: ${thread.name}`);
```

To delete a thread, use the <DocsLink path="class/ThreadChannel?scrollTo=delete" type="method" /> method:

<!-- eslint-skip -->

```js
const thread = channel.threads.cache.find(x => x.name === 'food-talk');
await thread.delete();
```

## Joining and leaving threads

To join your client to a ThreadChannel, use the <DocsLink path="class/ThreadChannel?scrollTo=join" type="method" /> method:

<!-- eslint-skip -->

```js
const thread = channel.threads.cache.find(x => x.name === 'food-talk');
if (thread.joinable) await thread.join();
```

And to leave one, use <DocsLink path="class/ThreadChannel?scrollTo=leave" type="method" />;

<!-- eslint-skip -->

```js
const thread = channel.threads.cache.find(x => x.name === 'food-talk');
await thread.leave();
```

## Archiving, unarchiving, and locking threads

A thread can be either active or archived. Changing a thread from archived to active is referred to as unarchiving the thread. Threads that have `locked` set to true can only be unarchived by a member with the `MANAGE_THREADS` permission.

Threads are automatically archived after inactivity. "Activity" is defined as sending a message, unarchiving a thread, or changing the auto-archive time.

To archive or unarchive a thread, use the <DocsLink path="class/ThreadChannel?scrollTo=setArchived" type="method" /> method and pass in a boolean parameter:

<!-- eslint-skip -->

```js
const thread = channel.threads.cache.find(x => x.name === 'food-talk');
await thread.setArchived(true); // archived
await thread.setArchived(false); // unarchived
```


This same principle applies to locking and unlocking a thread via the <DocsLink path="class/ThreadChannel?scrollTo=setLocked" type="method" /> method:

<!-- eslint-skip -->

```js 
const thread = channel.threads.cache.find(x => x.name === 'food-talk');
await thread.setLocked(true); // locked
await thread.setLocked(false); // unlocked
```

::: warning
Archived threads can't be locked!
:::

## Public and private threads

Public threads are viewable by everyone who can view the parent channel of the thread. Public threads can be created with the <DocsLink path="class/ThreadManager?scrollTo=create" type="method" /> method.

<!-- eslint-skip -->

```js {4}
const thread = await channel.threads.create({
	name: 'food-talk',
	autoArchiveDuration: 60,
	reason: 'Needed a separate thread for food',
});

console.log(`Created thread: ${thread.name}`);
```

They can also be created from an existing message with the <DocsLink path="class/Message?scrollTo=startThread" type="method" /> method, but will be "orphaned" if that message is deleted.

<!-- eslint-skip -->

```js {4}
const thread = await message.startThread({
	name: 'food-talk',
	autoArchiveDuration: 60,
	reason: 'Needed a separate thread for food',
});

console.log(`Created thread: ${thread.name}`);
```

The created thread and the message it originated from will share the same ID. The type of thread created matches the parent channel's type.

Private threads behave similar to Group DMs, but in a Guild. Private threads can only be created on text channels.

To create a private thread, use <DocsLink path="class/ThreadManager?scrollTo=create" type="method" /> and pass in `GUILD_PRIVATE_THREAD` as the `type`:

<!-- eslint-skip -->

```js {4}
const thread = await channel.threads.create({
	name: 'mod-talk',
	autoArchiveDuration: 60,
	type: 'GUILD_PRIVATE_THREAD',
	reason: 'Needed a separate thread for moderation',
});

console.log(`Created thread: ${thread.name}`);
```

## Adding and removing members

You can add and remove members to and from a thread channel.

To add a member to a thread, use the <DocsLink path="class/ThreadMemberManager?scrollTo=add" type="method" /> method:

<!-- eslint-skip -->

```js
const thread = channel.threads.cache.find(x => x.name === 'food-talk');
await thread.members.add('140214425276776449');
```

And to remove a member from a thread, use <DocsLink path="class/ThreadMemberManager?scrollTo=remove" type="method" />:

<!-- eslint-skip -->

```js
const thread = channel.threads.cache.find(x => x.name === 'food-talk');
await thread.members.remove('140214425276776449');
```

## Sending messages to threads with webhooks

It is possible for a webhook built on the parent channel to send messages to the channel's threads. For the purpose of this example, it is assumed a single webhook already exists for that channel. If you wish to learn more about webhooks, see our [webhook guide](/popular-topics/webhooks.md).

```js
const webhooks = await channel.fetchWebhooks();
const webhook = webhooks.first();

await webhook.send({
	content: 'Look ma! I\'m in a thread!',
	threadId: '123456789012345678',
});
```

And that's it! Now you know all there is to know on working with threads using discord.js!
