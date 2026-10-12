# blueprints

The **core catalog** for [paisans-stack](https://github.com/Paisans-Software/paisans-stack): the first party blueprints every deployment can use.

A blueprint is how paisans-stack runs one application. It is a directory holding a `blueprint.yaml`, which says what the toolkit needs to know about the application (its containers and images, its port, its health route, how it signs people in, what the gate leaves to it, its secrets), and the templates the toolkit renders for it. The design is `docs/specs/2026-10-12-blueprints-and-catalogs.md` in paisans-stack.

```
catalog.yaml
blueprints/
  element/       Element, the Matrix web client
  mbin/          Mbin, forums and social (the paisans fork)
  outline/       Outline, the wiki
  synapse/       Synapse with Matrix Authentication Service
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

Each paisans-stack release records the commit of this repository it was tested against and carries a copy of it, under `internal/blueprints/core/`. A deployment renders that copy, so a given toolkit renders the same blueprints every time, with or without network access.

A blueprint here changes in two moves: a pull request here, then a paisans-stack pull request that moves its pin to the new commit (`scripts/sync-core-catalog.sh` there). The toolkit's tests, golden render tree included, run against the new copy before anything uses it.

## What is not here

The identity provider (`pocket-id`), the gate (`oauth2-proxy`), the invite service (`invite`) and the monitor (`uptime`) are reserved: they are built into paisans-stack with the Go that drives them, and no catalog provides them, this one included.

## Writing a blueprint

`blueprint.yaml` is read by `internal/kinds.Parse` in paisans-stack, which documents every key and refuses any it does not know. Each value that comes from an application's source is cited beside it in a comment, with the tag it was read at, because a wrong one fails at the worst moment: a callback that does not match fails at the end of a member's first sign in.
