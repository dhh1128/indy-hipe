# 00?? Cooperative Disclosure
- Author: Daniel Hardman <daniel.hardman@gmail.com> 
- Start Date: 2019-02-08
- PR: (leave this empty)

## Summary
[summary]: #summary

Explain how agents at the edge can cooperate to deliberately disclose
limited information to other agents (e.g., routing agents) in the
communication path.

## Motivation
[motivation]: #motivation

We expect messages to be routed to the edge of sovereign domains in
many cases. However, routing agents in the cloud are a logical place
to implement policy--and to implement policy, they may need some
visibility into the semantics of interactions as they unfold. This
creates a tension, as messages they forward can't be maximally opaque.
We need a way to disclose just enough metadata, and we need it to be
interoperable, safe, and well understood.

## Tutorial
[tutorial]: #tutorial

Suppose Bob has a fancy cloud agent, `3`, that routes messages to his mobile
phone, his laptop, and his tablet. He wants to use the routing agent to
filter and triage messages, as follows:

1. Unsolicited messages from unknown senders (e.g., anonymous
love letters) should be held by his routing agent until Bob feels like
retrieving them.
2. Proof requests requests should be delivered to all Bob's devices.
3. Messages that are work-related should be delivered to Bob's phone
and laptop.
4. Personal messages should be delivered to Bob's phone and tablet.

![bob's routing goals](bobs-policy.png)

The default posture in agent-to-agent communication is maximum
privacy, and minimal trust in a routing agent in the cloud that could
be hacked or co-opted by a malicious sysadmin. If this posture isn't
modified, then the `forward` messages received by routing agent `3`
will not contain enough information for most of these policies to be
enforced. (The exception will be policy #1 about anonymous senders;
the use of anoncrypt mode is visible to a forwarder by inspecting
[the `alg` header inside the `protected` section of a wire message](
https://github.com/hyperledger/indy-hipe/tree/master/text/0028-wire-message-format#pack_message-return-value-anoncrypt-mode).)

To implement policies 2-4, agents that send messages to Bob can
include a `disclose` block at the root of `forward` messages that Bob's
routing agent `3` sees. 

```JSON
{
  "@type" : "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/routing/1.0/forward",
  "to"    : "did:sov:1234abcd#4",
  "msg"   : "<pack(AgentMessage,valueOf(did:sov:1234abcd#4), privKey(A.did@A:B#1))>"
  "disclose": {
      "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/notification/1.0/ack"
  }
}
```

This tells the routing agent that, if it were able to
decrypt the `msg` field that it is forwarding, it would see an
A2A message of type `ack`. This is enough for the routing agent to
implement policy #2, since message type is now known.

To address policies #3 and #4, the `disclose` block needs an additional
field, `origin`:

```JSON
"disclose": {
  "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/notification/1.0/ack"
  "origin": "did:sov:gzCbsANjHiqZDAsuTYhgrH"
}
```

This tells the routing agent that the origin of the inner, encrypted
message is a particular DID. If the routing agent has a mapping table
that associates DIDs of Bob's contacts with the labels "work" and
"personal", it can use this information to implement policies #3 and #4.

### Lying

Of course, this mechanism lets a sender lie to Bob's routing
agent. But this is a feature, not a bug. Bob and Alice can pass love
letters and call them `ack` messages, if they like. Whether the routing
agent understands context accurately is not its concern; it is working
properly if it reacts to purported disclosure as Bob instructs it to.

This is not unlike familiar situations in the non-digital world.
Imagine Alice and Bob passing love letters using Bob's butler as a
go-between, and telling the butler that they are corresponding about
a real estate transaction. It is not the butlers' job to understand
the true nature of the transaction; it is his job to deliver
correspondence about real estate according to whatever policy Bob
sets.

If Alice abuses the channel to claim things about her inner message
that prove false, in a way that Bob considers annoying, then Bob
can always sever the relationship, ending Alice's ability to reach
him. This is substantially better than the situation with email and
SMS spam today, where anonymous senders can contact a person at will.

### Cooperating

This protocol is called *cooperative* disclosure because Bob can
negotiate disclosure behaviors with each of his connections. He does
this by announcing disclosure requirements in his DID Doc. He can
create different requirements for each agent, and different
requirements in each relationship.

[TODO: where would we put this, in a DID Doc? Should we instead,
or in addition, invent a message family that communicates
disclosure policy? How fine-grained do we need to get?] 
 

## Reference
[reference]: #reference


## Drawbacks
[drawbacks]: #drawbacks

* Privacy erosion (Sender can reveal something receiver doesn't want.
However, maybe that's okay, because there's no reason sender can't
do that, even if we *don't* specify a field on forward messages.
Is the standardization of disclosure pernicious?) 

## Rationale and alternatives
[alternatives]: #alternatives

- If we don't do this, there will be no way to use cloud/routing agents
for policy enforcement.
- What other designs have been considered and what is the rationale for not
choosing them?
- What is the impact of not doing this?

## Prior art
[prior-art]: #prior-art


## Unresolved questions
[unresolved]: #unresolved-questions

