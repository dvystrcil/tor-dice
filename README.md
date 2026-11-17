# tor-dice

Deploy charts for the [TOR dice roller](https://github.com/dvystrcil/tor-dice-docker) (the actual code lives in `tor-dice-docker`). Argo-managed.

Tracked: [dvystrcil/homelab#262](https://github.com/dvystrcil/homelab/issues/262).

## Layout

```
.
├── base/
│   ├── namespace.yaml           tor-dice ns, ambient-labeled for gateway
│   ├── harbor-pull-secret.yaml  InfisicalSecret → docker pull creds
│   ├── deployment.yaml          single-replica Go binary + embedded SPA
│   ├── service.yaml             ClusterIP :8080
│   ├── vpa.yaml
│   └── kustomization.yaml
└── overlays/
    └── kustomization.yaml       the deployed newTag
```

## Image flow

1. tor-dice-docker GitHub release published → docker-release.yaml promotes `:dev` to `:semver` tags
2. Its `deploy` job (release-action's deploy step, homelab#2320) commits the new `newTag` here: straight to main, or as a `deploy/tor-dice-<ver>` PR while tor-dice-docker's `AUTO_DEPLOY` is `false`. It edits only `newTag` lines, never downgrades, and sends a major to a PR.
3. Argo syncs the new tag, rolls the pod

## Routing

This repo does NOT define the HTTPRoute — `gateway-services` (the umbrella chart in the homelab repo) owns all gateway routing entries. The dice-roller's `dice.sirddail.net` host is registered there separately.

## Image source

[harbor.sirddail.net/ai/tor-dice](https://harbor.sirddail.net/harbor/projects/ai/repositories/tor-dice) — pushed by [tor-dice-docker](https://github.com/dvystrcil/tor-dice-docker)'s build workflows.

## License

[MIT](LICENSE). Same dual-license terms as the sibling tor-dice-docker.
