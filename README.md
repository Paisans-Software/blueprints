# blueprints

The **core catalog** for [paisans-stack](https://github.com/Paisans-Software/paisans-stack): the first party blueprints every deployment can use.

A blueprint is how paisans-stack runs one application. It is a directory holding a `blueprint.yaml`, which says what the toolkit needs to know about the application (its containers and images, its port, its health route, how it signs people in, what the gate leaves to it, its secrets), and the templates the toolkit renders for it. The design is `docs/specs/2026-10-12-blueprints-and-catalogs.md` in paisans-stack.

```
catalog.yaml
foundation.yaml   the toolkit's own images
blueprints/
  element/       Element, the Matrix web client
  mbin/          Mbin, forums and social (the paisans fork)
  synapse/       Synapse with Matrix Authentication Service
  wikijs/        Wiki.js, the wiki, with its bootstrap
  writefreely/   WriteFreely, the blog (the writefreely-wisp fork)
```

A deployment names one in `paisans-stack.yaml`:

```yaml
apps:
  talk:
    blueprint: mbin
```

An unqualified name always means this catalog.

## How the toolkit uses it

paisans-stack fetches the newest release of this repository it can read, every time it plans, unless a deployment pins one in `catalogs`. A release is a tag `vMAJOR.MINOR.PATCH`; a pre-release is never chosen. A release whose files an older toolkit cannot read is skipped by that toolkit, which uses the newest one it can.

Every image here is a **track**: a repository and a tag pattern in which each `*` matches one run of digits (`ghcr.io/pocket-id/pocket-id:v2.*.*`). `paisans apply` runs the newest tag matching the whole pattern, pinned by its digest, so a patch or a minor release reaches a deployment on its next apply with nobody editing anything. A pattern stops where an upgrade stops being routine; moving it is a release of this catalog.

`foundation.yaml` names the images of the services whose templates are the toolkit's own: the infrastructure stack (etcd, Spilo, HAProxy, Garage, the gateway's Caddy, the heartbeat) and the reserved blueprints below.

## What is not here

The identity provider (`pocket-id`), the gate (`oauth2-proxy`), the account service (`account`) and the monitor (`uptime`) are reserved: they are built into paisans-stack with the Go that drives them, and no catalog provides them, this one included.

## Writing a blueprint

`blueprint.yaml` is read by `internal/blueprints.Parse` in paisans-stack, which documents every key and refuses any it does not know. Each value that comes from an application's source is cited beside it in a comment, with the tag it was read at, because a wrong one fails at the worst moment: a callback that does not match fails at the end of a member's first sign in.

A blueprint that other apps link to says what it offers under `provides`, one CEL expression per value (Synapse's `matrix-homeserver` offers `base_url` and `server_name`), and one that links to it names the role under `consumes` with the setting that names its provider. paisans-stack knows no role itself.

`catalog.yaml`'s `wizard` is the menu `paisans create` offers, in order. Adding a blueprint there is how a new deployment is offered it.
