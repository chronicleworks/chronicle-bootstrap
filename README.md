# Chronicle Bootstrap

This repo serves as a base for setting up your own Chronicle domain.

It contains the structure and `Makefile` which can be used to build docker
images and run a local instance of Chronicle.

It does not contain an example domain, however if you would like to test it,
you can use a domain from the
[Chronicle Examples](https://github.com/chronicleworks/chronicle-examples) repo.

For example, for a debug build of the manufacturing domain,

```bash
gmake manufacturing-stl-debug
```
or a release build for the same domain,
```
gmake manufacturing-stl-release
```

As above, the name of any of the other listed domains may be substituted for
`manufacturing`.

After the build, running `docker image ls` should show the built image that
can then be pushed to the appropriate registry for installation.

By default, the images are given tags like, say,
`chronicle-manufacturing-stl-release:local`. A value other than `local` can
be set in the `ISOLATION_ID` environment variable prior to build.

### Set Environment Variables

Additional environment variables can be set that are recogized by the running
Chronicle process. You may list these in the `docker/chronicle-environment`
file which is initially empty. To instead read them from a different file,
set its location in the `DOCKER_COMPOSE_ENV` environment variable.

For example, to have Chronicle require all API requests to be
authenticated, you could write your authentication provider's
[OIDC endpoints](https://docs.chronicle.works/auth/) into
`docker/chronicle-environment` thus,

```properties
REQUIRE_AUTH=1
JWKS_URI=https://id.example.com:80/.well-known/jwks.json
USERINFO_URI=https://id.example.com:80/userinfo
```

taking the variable names from `chronicle serve-api --help`. Then, after you
use `gmake run-my-domain` or similar, the running Chronicle will use the
specified authentication provider to verify incoming requests.

## Setting up your own domain

This repository follows the same structure as the [Chronicle Examples](https://github.com/chronicleworks/chronicle-examples)
repository, which you can use for reference.

To build any specific Docker image, use the form `gmake A-B-C` where,

A
: the name of the domain, e.g., `manufacturing`

B
: either `inmem` or `stl`

C
: either `release` or `debug`

which builds and tags an image named `chronicle-A-B-C`. The previous
`chronicle-A-B` tag names are deprecated.

## GraphQL Client


Chronicle Examples use Chronicle's `serve-graphql` function to provide the
Chronicle GraphQL API. By using a GraphQL client, you can interact with Chronicle
by running GraphQL queries, mutations, and subscriptions.

We recommend using the [Altair GraphQL Client](https://altairgraphql.dev/),
which is available as a free desktop GraphQL IDE or web browser extension.

If you have previously used Chronicle Examples, you can still access the
[GraphQL Playground](https://github.com/graphql/graphql-playground) through your
web browser at <http://127.0.0.1:9982>, however we will be deprecating support
for GraphQL Playground in future releases.

Both of these GraphQL clients are persistent via cookies, so running the same
browser on the same machine will remember all your queries and tab positions.

To add a new mutation or query tab, there is a `+` on the right-hand side of the
tab bar.

Once you get to this point, you are ready to explore the example. To do this,
refer to the relevant guide.

### Notes

If you are using Chronicle on default settings, point the GraphQL client to
<http://127.0.0.1:9982>.


In the case of the GraphQL Playground the *SCHEMA* and *DOCS* tabs are useful
for showing the relationship between your `domain.yaml` configuration and the
resulting Chronicle API.

__NOTE__ Use Shift-R to refresh the playground before rerunning your example.

### Subscribing to Events (1)

Finally, to see what is happening as you run GraphQL mutations and queries, you
can subscribe to async events in one of the tabs. The GraphQL Playround handles
this automatically but other GraphQL clients may ask you to explicitly provide
the websocket end point. The default is <ws://localhost:9982/ws>.

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


__NOTE__ that if you are using JWT authorization you will need to install the
a GraphQL client that supports subscriptions in this scenario. We have verified
that [GQL 3](https://gql.readthedocs.io/en/latest/intro.html) works.

Once installed you can run the same subscription using `gql-cli`. To faciliate
this we provide a script [subscription.sh](./scripts/subscription.sh). Simply
run this and respond to the prompts. If you haven't locked down your
environment using JWT the bearer token will be ignored.

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
