# byo-identity

**Not written yet.** This directory is a placeholder: the application is a
v0.3 deliverable (roadmap item 73), and nothing here is code.

## What it will be

The smallest possible proof that none of gorbital's identity code is
required. No gorbital sign-in, no gorbital organisations, no
`internal/modules/auth`, no `internal/modules/orgs` — and still a complete,
tested API with authenticated routes, permissions and audit.

Where the other applications answer "how do I use what gorbital gives me",
this one answers the question that comes before it: *what happens if I don't
want any of it?* An application that carries the framework's identity model
cannot answer that, because it is the thing being tested.

Planned shape:

- an authenticator the application writes, backed by whatever it likes — a
  token the application already issues, a header a gateway sets — wired in
  the same place `modules/auth` would be;
- actors and permissions derived from that, so guards, audit and the Dev
  Portal work unchanged;
- no organisations at all: one tenancy, one kind of caller;
- tests that show a signed-in request, a refused request and an audited
  write, with no gorbital identity module compiled into the binary.

`mobile-backend` is the nearest thing that exists today: it wires
`modules/jwt` by hand against an external identity provider. byo-identity
goes one step further and drops the library's identity modules entirely.

## What is here now

This README. When the application lands, it gets a `go.mod` like every other
application here and a `use ./byo-identity` line in the root
[`go.work`](../go.work).
