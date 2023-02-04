---
title: How to read the Ably client library specification
noindex: true
---

# How to read the Ably client library specification

Working with spec version 6a1d650 (I think [this](https://sdk.ably.com/builds/ably/specification/pull/123/features/) is the nearest published one that's still available.)

- What are its responsibilities
- What kind of state does it manage and what causes it to be mutated
  - e.g. stuff from backend, transport events, interactions with public API. I guess more broadly, what are the external inputs to the library?
  - outputs other than stuff via public API
    - e.g. calls to system push notifcation methods in RSH3a2d
  - bear in mind these things are related, e.g. according to RTN15h the server can send a DISCONNECTED message which means the transport will be closed
  - OS network/connectivity change events (RTN20)
  - OS push notification token success and failure events (RSH8g, RSH8h)
- How it responds to its own state changes
  - e.g. RTN20a says that RTN15 says that transitioning to the DISCONNECTED state "automatically trigger the client library to attempt to disconnect"
- What assumptions do we make about the transport?
- How is the transport modelled? What kind of errors can it raise?
- How is this state exposed in public API
- How its behaviours vary depending on its state
- What things does it need to synchronise with the server
- What does it use as an indicator of the success or failure of various operations
- How does it respond to external events
- What kind of retry / recovery behaviour exists
- Consistency checks
- Behaviours of the backend it mentions / hints at
- How the different `clientId` in the different places it can be specified (auth token, options, ...) affect the behaviour of the library (RSA8f, ...)
- Are there some common definitions it would be useful to have once?
- The motivations for some of its behaviours
- Can we make it clearer from the spec points which ones depend on state and which don’t?
- Weird things like a `ConnectionStateChange` to `CONNECTED` with a non-nil error `reason` (I remember Paddy saying this one happened when it failed to restore a connection but that didn't mean something had gone wrong)
- Think about implications of threading plus the internal things that use its own public methods
  - e.g. does it make sense to say "when this public method is called, take this action depending on the current state" if you can call a method from any thread?
  - or e.g. RTL17 "No messages should be passed to subscribers if the channel is in any state other than ATTACHED".
- The _why_ behind all this - especially things like state changes
- Making sure that if something does something that then takes advantage of some behaviour (e.g. if bad thing happens, put connection into state x, then the connection going into state x will automatically trigger some recovery behaviour), then this thinking is known
- Interactions between components

Gonna start scanning spec for some of these things. No intention of being comprehensive.

The thing about the spec is that all of these kinds of things are all interspersed. I want to focus on one thing at a time (e.g. all of the state). That's how I’ll try and attack this.

## State

Here’s a list (that I’ve inferred from the spec) of all the state that the library manages.

- I wonder if we can categorise these
- What kind of invariants or relationships between these states can we find? e.g x is only non-null when y is one of these values
  - These are really useful things to be able to write in documentation, they help people reason about the state of the system and we can also make assertions about them
  - What things can be factored out, what things are specific to a certain e.g. channel state?

### REST

- successful fallback host (RSC15f)
  - cleared after some timer
- storing the auth token in use (implied by RSA4b?, RSA7a, RSA12)
- storing whether to use token auth for all future requests (RSA10)
- the current `clientId`
- whether the client is identified (RSA7a)
- the `ConnectionDetails#clientId` provided by the backend after a connection to Ably
- server time offset from local clock (RSA10k)
- the `AuthOptions` and `TokenParams` arguments of `Auth#authorize` (RSA10g)
- the current list of channels (RSN2)
- the current `ChannelOptions` for a channel (RSL7)

### Realtime

The above (in some places there's a corresponding realtime spec point that kinda says the same thing, haven't mentioned those), plus:

- connection state (e.g. RTN4d)
- `ConnectionDetails` received in `CONNECTED` ProtocolMessage (RTN21)
- messages awaiting ACK or NACK (RTN7a)
  - ProtocolMessages awaiting ACK or NACK (RTN19a)
- current `msgSerial` (RTN7b says "unique", not sure within what scope, guess connection. Described in RTN10b as "library-internal `msgSerial`")
- `Connection#serial` updated in response to `ProtocolMessage` received from Ably
- `Connection#id`
- `Connection#key`
- random `id` of `ProtocolMessage` sent for `Connection#ping`
- timer for connection retry (RTN14d)
- time spent in connection state for checking whether it exceeded `connectionStateTtl` (RTN14d, RTN15g)
- whatever the "local connection state" that RTN15g says should be cleared is
- the `ProtocolMessage#connectionKey` from the most recent `CONNECTED ProtocolMessage` received (for resume, RTN15b)
- the latest `connectionSerial` received on the connection (RTN16b)
- `Connection#errorReason` (e.g. set by RTN16e. A full list of setters in RTN25)
- the random order in which we’re going to try fallback hosts (RTN17c)
- which fallback host we're connected to (RTN17e)
- time when transport last received indication of activity (RTN23a)
- per channel:
  - presence actions that are queued for send on that channel (RTL11, RTP16b)
  - information about the `#attach` operation that should be performed after the channel leaves `DETACHING` or `ATTACHING` state (RTL4h)
  - information about the `#detach` operation that should be performed after the channel leaves `DETACHING` or `ATTACHING` state (RTL5i)
  - time when we sent `ATTACH ProtocolMessage` (for checking that `ATTACHED` is received within default realtime request timeout, RTL4f)
  - time when we sent `DETACH ProtocolMessage` (for checking that `DETACHED` is received within default realtime request timeout, RTL5f)
  - information about whether the channel has previously been attached or has been explicitly detached since last time it was attached (for knowing whether it’s a _clean attach_, RTL4j1)
  - the channel’s "previous state" for transitioning back to if `#detach` request fails (RTL5f)
  - ACK/NACK/... callbacks for `#publish` (RTL6b)
  - attach callbacks for `#subscribe` (RTL7c)
  - the `RealtimeChannel#properties.attachSerial` received in the `ATTACHED ProtocolMessage` (RTL10b, RTL15a) - for only fetching history messages since before channel was attached
  - a flag to control automatic re-attach attempts after channel receives server initiated `DETACHED` message (RTL13b, RTL13c)
  - `RealtimeChannel#errorReason` (e.g. RTL14)
  - stored channel options (RTL16)
  - attach callbacks for `#setOptions` (RTL16a)
  - last message (specifically, id and payload) received on a channel, for vcdiff decoding (RTL19, RTL20)
  - current retry number (for incremental backoff) (RTB1)
- per channel’s `RealtimePresence` instance:
  - a `PresenceMap` – "used to maintain a list of members presence on a channel ... broadly ... a map of memberKeys to presence messages" (RTP2)
  - the current sync sequence identifier (RTP18a)
  - (during sync) list of members that have not been added or updated in the `PresenceMap` during the sync process (RTP19)
  - a second (private) `PresenceMap` containing only members that match the current `connectionId` (RTP17)
  - queued presence messages (mentioned in `RTP5*`, may have already been covered?)
  - attach callbacks for `#subscribe` (RTP6c)
  - callbacks for ACK / NACK of `#enter` / `#update` / `#leave` (and their `*Client` variants)
  - callbacks for `#get` until `SYNC` happens if `waitForSync` (RTP11c)
  - `#syncComplete` - whether the initial `SYNC` has completed (RTP13)
- connection-wide message queue for `#publish` (RTL6c2)
- current retry number (for incremental backoff) (RTB1)
- `PushAdmin`:
  - whether the client has been activated as a push target device (for deciding whether to include push device authentication in RSH1b / RSH1c) - I think this might just be part of `LocalDevice`’s state
- Push activation state machine (with lifetime the _app_, not the process):
  - The current enum-valued state
  - Things like RSH3b4a imply that it needs to store the callback that was passed to `Push#activate`. (Presumably that isn’t meant to be persisted.)
  - Things like RSH3b3a imply that it needs to store the `registerCallback` that was passed to `Push#activate`. (Presumably that isn’t meant to be persisted.)
  - Things like RSH3g2b imply that it needs to store the callback that was passed to `Push#deactivate`. (Presumably that isn’t meant to be persisted.)
  - RSH3e’s "if/unless the machine is in state ... as a result of a `CalledActivate` event", is there some extra state needed to know that?
  - The event queue defined by RSH4 for handling the case where there isn’t a transition from the current state
- `LocalDevice` (from RSH8a it sounds like only some of its properties have lifetime the same as the app):
  - `id` (persisted, RSH8b)
  - `deviceSecret` (persisted, RSH8b)
  - `clientId` (persisted, RSH8a)
  - `deviceIdentityToken` (persisted, RSH8a)
  - `push: DevicePushDetails` (see IDL)
    - `errorReason` - not clear where it comes from
    - `state`- not clear where it comes from
    - `recipient` (persisted, RSH8a)
  - `metadata` - not clear where it comes from
  - `formFactor` - not clear where it comes from
  - `platform` - not clear where it comes from

For connection / channel / presence:

- listeners for `subscribe` / `EventEmitter` in general `(RTE*)`

OK, this list is done.

(Are any of these the same thing, e.g. `Connection#serial` and "the latest `connectionSerial` received on the connection"?)

What model of a state machine does the push activation one fit into? It has more than one piece of state, by the looks of it. And has stuff like an event queue for no matching transitions.

## Retries

These are just things I noticed whilst looking at state, not comprehensive

I guess these imply some kind of state

- renewing tokens (RSA4b, RTN14b)
- recoverable connection attempt error (RTN14d)

## Thoughts on structure

- So you have a transport, which roughly speaking sends and receives protocol messages? And can be closed and has some events like being closed by the server
- And a connection, which does what? Is it responsible for e.g. RTN7, tracking ACK and NACK?
- Who can influence what? A connection can influence a channel, but can a channel influence a connection? I think it would be good to be able to describe the behaviour of the connection in detail and then bolt channels on top of that, is that possible?
- A connection has an identity, right? Because it can be replaced by the server? Is that something that a channel needs to be concerned with? Because it needs to do extra work if the connection changes?
- I think that the top-level `Connection` is really then a wrapper for a sequence of connections, and has some sort of dispatching logic. So the word "connection" might be overloaded.
- There's some link between the connection recovery/resume and the channel state resume (e.g. see TH4 which emits a flag with this info).
- I think that having a good understanding of the connection (auth etc)

## What does the `Connection` do?

I think that there are some layers going on, even though the spec doesn't make it clear, and it would be good to think about what each layer adds on top of the previous.

Can 1-3 each be broken and re-created transparently? TCP connection definitely.

1. TCP connection
2. Transport
3. Connection
4. Channel

   Not sure if this counts as another layer in terms of transport.

   Also note that the channel is not invisible to the connection – e.g. RTN11d says that the connection can reset state on the channel when `connect` is called

   Also note that RTN11d implies that if the connection is in a given state, then the channel is already in the corresponding state? which, depending on threading, might not be true

There probably should be a layer in between connection and channel that handles connection events that affect all channels, instead of directly fanning out to all channels

Let's scan through the `Connection` and see roughly what kind of things it does on top of the transport (I haven't defined exactly what the transport does yet though):

- has a bunch of states: `INITIALIZED`, `CONNECTING`, `CONNECTED`, `DISCONNECTED`, `SUSPENDED`, `CLOSING`, `CLOSED`, or `FAILED` – more than the transport
- its states are a composite of the transport state and received `ProtocolMessage` – e.g. RTN6 which defines the `CONNECTED` state (responding to `CONNECTED` ProtocolMessage from Ably)
- responsible for making sure a sent `Presence` or `Message` `ProtocolMessage` was delivered to and accepted by Ably, by waiting for `ACK` or `NACK` (RTN7a)
- responsible for failing these sent messages when the connection enters certain states and the message is not yet acknowledged (RTN7c)
- responds to `AUTH` `ProtocolMessage` from Ably (forced re-auth, RTN22)
- responding to `DISCONNECTED` `ProtocolMessage` from Ably (not sure exactly what this means, is that a "you need to reconnect" e.g. RTN15h)
- is able to cancel an ongoing retry process in order to perform an action immediately (RTN11c, `connect` called when `DISCONNECTED` or `SUSPENDED`)
- waits for confirmation from Ably on user-initiated close (waits for `CLOSE` `ProtocolMessage`)
- responds to abrupt closures in transport during close (RTP12c)
- retries on token error (RTN14b)
- offers an `errorReason` property which I’m not sure how it's meant to be used by consumers
- how to properly track state of things like a single retry? RTN15h
- responding to `ERROR` `ProtocolMessage` from Ably (RTN15i)
- resuming a connection and recovering connection state (by holding on to information from the last `CONNECTED` `ProtocolMessage` received)
- how do failed resumes work? is there a process of trying to create a connection and seeing whether you end up with a new one or not?
- how does Ably know which messages the client actually received? is that what one of those serials is for?
- handling Ably’s response to a resume request (RTN15c)
  - all channels still attached, with / without all backlog messages available
  - new connection established
- re-initiating channel attaches if resume request failed (RTN15c3)
- re-sending un-acknowledged messages on a resumed connection with a new transport (RTN15f)
- responding to OS connectivity events (RTN20)
- trying callback hosts if necessary when connecting (RTN17)
- re-sending `ProtocolMessage` awaiting acknowledgement when transport disconnected
- re-sending `ATTACH` or `DETACH` for channel in `ATTACHING` or `DETACHING` state when transport disconnected
- handling a `CONNECTED` message received from Ably at any time (why? does this mess with our understanding of what a connection is?) – RTN24

TODO when can the connection’s ID change? This is what I wanted to know to flesh out "a connection might actually be multiple connections"

- RTN11b says that if the state is `CLOSING` and `connect` is called, then "the client should make a new connection with a new transport instance" and ignore stuff received on the old connection. Not clear exactly what a “new connection” means here – does it mean it should replace the `Connection` instance?
- Like, I think that certainly if a connection ends up in `CLOSED` and you `connect` again, that’s a “new connection” whatever that means
- RTN15g explains when to discard the local connection state (after disconnected for more than `connectionStateTtl`)

## The transport

- RTN12c says it is something that can be "abruptly closed"

## Thoughts on implementation

- Again, lots of good data structures that allow us to introspect exactly what's going on at any given time. Explicit state as opposed to call stack.
- Lots of good information being saved when one class asks another class to do something. E.g. when the channel asks the connection to do something, it should get a receipt so that the operations can be correlated. Not sure how this would play with the Swift async stuff.
- The stuff where the channel checks the connection’s state before making a decision about how to proceed – this seems like it could be fraught with synchronisation issues. See what RTL6c2 says about this. It’s also a bit confusing because the channel state is dependent on the connection state
- In general, how to implement the operations that are dependent on the current state even of the receiver (e.g. RTL7c which says that subscribe should attach if the channel is `INITIALIZED`)
