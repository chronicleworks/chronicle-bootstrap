# Chronicle Bootstrap

This repo serves as a base for setting up your own Chronicle domain.

It contains the structure and `Makefile` which can be used to build docker
images and run a local instance of Chronicle.

It does not contain an example domain, however if you would like to test it,
you can use a domain from the
[Chronicle Examples](https://github.com/chronicleworks/chronicle-examples) repo.

For example, the [manufacturing domain](https://github.com/btpworks/chronicle-examples/blob/main/domains/manufacturing/domain.yaml).

## Chronicle Documentation

Documentation for Chronicle in general may be found [here](https://docs.btp.works/chronicle/).
Example domains may be found [here](https://examples.btp.works).

## Setting up your own domain

This repository follows the same structure as the [Chronicle Examples](https://github.com/btpworks/chronicle-examples)
repository, which you can use for reference.

To get started with Chronicle Bootstrap:

1. Clone this repo, or download it as a zip file from GitHub
   [here](https://github.com/btpworks/chronicle-bootstrap/archive/refs/heads/main.zip).
   *Please note, if you download or copy the repo rather than cloning it,
   you will need to make sure you keep it up to date with the latest Chronicle
   releases in the future.*
1. Add your `domain.yaml` file in its own directory inside the `domains`
   directory. The name of its directory should be the name of your domain. For
   example, if your domain is called `manufacturing`, you would create
   `domains/manufacturing/domain.yaml`.

## Running your domain

You can run your domain locally using the Makefile.
This will use a standalone version of Chronicle which is a single node with a
local database rather than backed by a blockchain. Replace `manufacturing` with
the name of your domain.

```bash
gmake manufacturing-sdl
```

## Understanding the Makefile targets and Docker images

As described above, Chronicle can be built either
[with an in-memory ledger or backed by Sawtooth](#run-a-standalone-node).
These have `inmem` or `stl` in their image name, respectively. Also, it can be
built  either as [a debug or a release
build](#deploy-to-a-chronicle-on-sawtooth-environment). The former suffixes
image names with `-debug`, the latter with `-release`.

The `gmake build` target builds the in-memory release images for every example
domain. For Sawtooth-backed release builds, use `gmake stl-release` instead.

The `run-*` targets build debug versions, other targets typically build
release builds. Debug builds the code much faster for incremental changes but
this advantage is irrelevant when each build is done in a fresh Docker
container. Therefore, the better-optimized release builds are typically
recommended.

To build any specific Docker image, use the form `gmake A-B-C` where,

A
: the name of the domain, e.g., `manufacturing`

B
: either `inmem` or `stl`

C
: either `release` or `debug`

which builds and tags an image named `chronicle-A-B-C`. The previous
`chronicle-A-B` tag names are deprecated.

## Using the GraphQL Client

Chronicle Examples use Chronicle's `serve-graphql` function to provide the
Chronicle GraphQL API. By using a GraphQL client, you can interact with
Chronicle by running GraphQL queries, mutations, and subscriptions.

We recommend using the [Altair GraphQL Client](https://altairgraphql.dev/),
which is available as a free desktop GraphQL IDE or web browser extension.

If you have previously used Chronicle Examples, you can still access the
[GraphQL Playground](https://github.com/graphql/graphql-playground) through your
web browser at <http://127.0.0.1:9982>, unless you are using port forwarding
or an ingress. However we will be deprecating support for the playground in
future releases.

Both of the playground and the Altair GraphQL Client are persistent via cookies,
therefore running the same browser on the same machine will preserves all your
queries and tab positions, simplifying resubmittng them if you are iterating on
an idea for example.

To add a new mutation or query tab, there is a `+` on the right-hand side of the
tab bar. Don't forget to use Shift-R to refresh your client before rerunning an
example.

In the case of the GraphQL Playground, the *SCHEMA* and *DOCS* tabs make it
easy to explore the relationship between your `domain.yaml` configuration and
the resulting strongly-typed Chronicle GraphQL API.

### Subscribing to Events (1)

In order to see what is happening as you run GraphQL mutations and queries, you
can subscribe to async events in one of the tabs. The GraphQL Playround handles
this automatically upgrading your HTTP connection to a websocket end point.
However, other GraphQL clients may ask you to explicitly provide this. Again,
the default is <ws://127.0.0.1:9982/ws>.

```graphql
subscription {
  commitNotifications {
    stage
    txId
    error
    delta
  }
}
```

### Subscribing to Events (2)

If you are using JWT authorization you will need to install a GraphQL client
that supports subscriptions correctly by passing the `Authorization` header.

Neither the Altair GraphQL Client nor the GraphQL Playground support this.
However, we have verified that the CLI
[gql-cli](https://gql.readthedocs.io/en/latest/gql-cli/intro.html)
included in [GQL 3](https://gql.readthedocs.io/en/latest/intro.html) handles
this gracefully. To install this use `pip`:

```bash
pip install "gql[all]"
```

Once `gql-cli` is installed you can use it to run the same subscription. To
faciliate this we have provided a script [subscription.sh](./scripts/subscription.sh).

We recommend that you run this script first, then experiment with mutations and
queries using your preferred GraphQL client.

__NOTE__ If you haven't locked down your environment using JWT, the bearer token
will be ignored.

## Adding a Domain

### Chronicle Definition

Adding a domain to the examples is as simple as adding a new `domain.yaml` file
to a new folder under `domains`.  The folder name will be used as the name of
the docker image.  For example, if you add a `domains/mydomain/domain.yaml`
file, the debug and inmem docker image will be `chronicle-mydomain-inmem:local`.

### User's Guide

The `domain.yaml` definition is typically the smaller part of what there is to
say about the domain's usage. Users will appreciate an accompanying `guide.md`
markdown document structured like those for the other example domains. Take a
look through those domains' guides because they illustrate how to write the
principal sections:

1. Modeling
1. Recording
1. Querying

Briefly explain what your domain is. Then, for the first section, take each
of the domain's most important activities, describe the participating agents
and entities, provide a diagram of how they relate to the activity, then show
how each is modeled in the `domain.yaml`. In this way, you can step through
various aspects of your domain, allowing the reader to accumulate a full
picture gradually. Conclude these by bringing those descriptions together as
the full `domain.yaml`. Note that `yaml` can be specified for the highlighting
in domain definitions.

For producing those diagrams, use [PlantUML](https://plantuml.com/)'s [class
diagrams](https://plantuml.com/class-diagram) with *extension* `--|>` arrows
showing which are agents, entities, and activities, and *directed association*
`-->` arrows for how those relate to each other. Include typed attributes as
fields in the class boxes where appropriate. Notice that the `docs/diagrams/`
folder has two subdirectories; review their contents and follow the same
pattern. Each of your domain agents, entities, and activities gets a
corresponding `include/*.iuml` file and, from your diagrams in `src/*.puml`,
you can `!include` the provided `default.iuml`, `agent.iuml`, `entity.iuml`,
`activity.iuml`, and your extra `*.iuml` for consistency across your diagrams
and those of the other example domains.

Follow the above tour of your domain with the other two sections: provide
example mutations and queries expressed in GraphQL, and show how the responses
should look, to give users some simple stories to try out in the Apollo
Sandbox in their browser. These examples should lead them through the most
important and common uses of your domain, giving them enough starting points
to easily try it out in their own applications. Note that `graphql` for
requests and `json` for responses can be specified for highlighting those
interactions.

For rendering your new guide locally, `docs/` also includes a list of the
Python dependencies required for running `mkdocs serve`. In using it to review
your guide, check that you have explained every aspect of your domain clearly
so the community can draw the greatest benefit from your work.
