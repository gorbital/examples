# gorbital examples

Runnable applications built on [gorbital](https://github.com/gorbital/gorbital).
Every code block on the library's **Build an app** and **Examples** pages is
included from a file here by marker, so the prose cannot drift from code that
compiles and passes its tests.

These applications are documentation. The library's own `examples/minimal`,
`examples/full-single`, `examples/full-multi` and `examples/v0.1/*` are not:
they are the source the `orb` CLI's templates are generated from, and they
stay in the library repository ([ADR-0093](https://github.com/gorbital/gorbital/blob/main/docs/adr/0093-the-examples-repository.md)).

## The applications

| App | What it shows | Builds against |
|---|---|---|
| [`plateful`](plateful/) | The flagship, and the app the tutorial chapters teach: a restaurant delivery platform where a restaurant *is* an organisation, with platform staff, restaurant staff, and customers and couriers who belong to no organisation — eight modules, an order state machine, a transaction across four tables, a custom guard, jobs, file storage and outbound webhooks | gorbital v0.3.0 |
| [`byo-identity`](byo-identity/) | **Not written yet** — a v0.3 deliverable. The smallest proof that none of gorbital's identity code is required: no gorbital sign-in, no gorbital organisations | gorbital v0.3.0 |
| [`mobile-backend`](mobile-backend/) | `modules/jwt` wired by hand as the app's authenticator: an external identity provider's claims become actors and permissions, scoped per user, tested against a local JWKS | gorbital v0.3.0 |
| [`payments`](payments/) | Receiving payment webhooks: `guard.Webhook` with a `webhook.NewStandard` verifier, idempotency keyed on the provider's event ID, and a job enqueued with `jobs.Client.InsertTx` inside the write's transaction | gorbital v0.3.0 |
| [`invoicing`](invoicing/) | Multi-tenant invoicing: `orgshttp`, a module generated with `orb gen module --org`, and a row-level security migration tested as a database role without bypass. **Candidate for retirement** — see below | gorbital v0.3.0 |
| [`admin-tool`](admin-tool/) | An internal admin tool: the built-in `/ops` and flags modules, a module's runtime setting, a client flag, retention and a named rate limiter. **Candidate for retirement** — see below | gorbital v0.3.0 |

Invoicing and admin-tool overlap Plateful and should be folded into it or
retired. That is an editorial change to a dozen documentation pages and a
judgement about what each chapter teaches, so it was deliberately kept out of
the move that created this repository and gets its own decision (ADR-0093,
decision 9). Until then they are built and tested like the rest.

Shelfie, the reading-tracker API the Examples chapters teach, is **not**
here. It came across in the move and went back: the library's CLI compares
`orb gen module`'s output with its shelves and clubbooks modules file by
file, and three more of the CLI's test files copy the whole application, so
it is a test fixture and lives in the library repository at
`examples/shelfie`.

## Running an application

Each application is its own Go module. With the `orb` CLI:

```bash
cd plateful
orb dev                    # the database, the migrations, the API and the Dev Portal
```

Without it:

```bash
cd plateful
cp .env.example .env       # then set AUTH_ENCRYPTION_KEYS: echo "k1:$(openssl rand -base64 32)"
docker compose up -d --wait
set -a; . ./.env; set +a
go run ./cmd/api migrate && go run ./cmd/api seed && go run ./cmd/api
```

Each application's own README says what it does and what it needs. Tests want
PostgreSQL, and some want Mailpit, exactly as the library's do.

## Building against the library

Every application requires the published `gorbital.dev` modules at **v0.3.0**.
None of them replaces a module with a relative path any more, which is the
point: they prove that the released library works, not that one checkout does.

**v0.3.0 is not published yet.** Until it is, nothing here builds from the
module proxy alone, and the root [`go.work`](go.work) points every application
at a checkout of the library beside this one:

```bash
git clone https://github.com/gorbital/gorbital ../gorbital
```

Without that directory every `go` command in this repository fails with
`replacement directory ../gorbital does not exist`.

A `go work use ../gorbital …` line is **not** enough, and this is worth
knowing: a `use` line makes the library a main module of the workspace, but
the go command still resolves the version each `go.mod` requires — v0.3.0 —
from the proxy while it loads the module graph, and fails before it builds
anything. `go.work` carries `replace` directives for that reason. The day
v0.3.0 is published, delete the `replace` block and the applications build
against the release with no local checkout.

To develop an application against the library, edit both trees and build: the
workspace already points at the checkout. To point it somewhere else, change
`../gorbital` in `go.work`, or run
`go work edit -replace gorbital.dev=<path>` for each module.

## Checks

CI runs, for every application:

```bash
gofmt -l .           # must print nothing
go vet ./...
go build ./...
go test -race ./...  # against Docker PostgreSQL and Mailpit, like the library
```

on every pull request, and again nightly against the library's `main`, so a
library change that breaks an application is found within a day rather than at
the next release.

An application's `api/openapi.json` is not regenerated by CI: after changing a
route, run `go run ./cmd/api openapi --dir api` in the application, which its
own `TestOpenAPIIsCurrent` checks. An application created by `orb new` also
keeps `internal/modules/surface_test.go` and `api/surface.json`, the record of
its error codes, permissions, settings, jobs and flags; after adding any of
those, run `go test ./internal/modules -run TestPublicSurface -update` and
commit the file.

## Including code in a page

Mark a region in any source file with a name that is unique within the
application:

```go
// docs:start create-book
func (h *Handlers) CreateBook(ctx context.Context, in *CreateBookInput) (*BookOutput, error) {
	...
}
// docs:end create-book
```

A documentation page includes the region by application, file and name, and
the lines between the markers are shown without the markers. The library's
`docscheck` resolves those markers against a checkout of this repository at
the tag pinned in its `docs/examples.json`, and fails when a file or a marker
has gone — so renaming or deleting a region breaks a build rather than a page.
This repository is tagged in lockstep with the library: `v0.3.0` here is the
counterpart of the library's `v0.3.0`.

## Versions and layout

Top-level application directories, not `apps/`: every documentation link and
README path is one segment shorter, and future non-application content gets
its own clearly named directory — `clients/` for generated SDK samples,
`deploy/` for deployment recipes.

The history of every file goes back to the library repository, where these
applications were written: this repository was created with
`git subtree split --prefix=examples/apps`.

## Licence

Apache 2.0, the same as the library. See [LICENSE](LICENSE), [NOTICE](NOTICE)
and the [code of conduct](CODE_OF_CONDUCT.md).
