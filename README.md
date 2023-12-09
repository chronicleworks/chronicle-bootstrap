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

To get started with Chronicle Bootstrap:

1. Clone this repo, or download it as a zip file from GitHub
   [here](https://github.com/chronicleworks/chronicle-bootstrap/archive/refs/heads/main.zip).
   *Please note, if you download or copy the repo rather than cloning it,
   you will need to make sure you keep it up to date with the latest Chronicle
   releases in the future.*
1. Add your `domain.yaml` file in its own directory inside the `domains`
   directory. The name of its directory should be the name of your domain. For
   example, if your domain is called `manufacturing`, you would create
   `domains/manufacturing/domain.yaml`.

From there, we suggest using the following sections of the Chronicle Examples `README`
as a guide:

- [Prerequisites](https://github.com/chronicleworks/chronicle-examples/blob/main/README.md#prerequisites)
- [Building a Domain](https://github.com/chronicleworks/chronicle-examples/blob/main/README.md#build-a-domain)
- [Running a Standalone Node](https://github.com/chronicleworks/chronicle-examples/blob/main/README.md#run-a-standalone-node)
- [Deploying to a Chronicle-on-Sawtooth Environment](https://github.com/chronicleworks/chronicle-examples/blob/main/README.md#deploy-to-a-chronicle-on-sawtooth-environment)
- [Generating a GraphQL Schema](https://github.com/chronicleworks/chronicle-examples/blob/main/README.md#generate-the-graphql-schema)
