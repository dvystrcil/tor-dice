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
│   ├── image-updater-rbac.yaml  Role for IU controller to read pullsecret
│   └── kustomization.yaml
└── image-updater/
    └── tor-dice-image-updater.yaml  ImageUpdater CR watching Harbor
```

## Image flow

1. tor-dice-docker GitHub release published → docker-release.yaml promotes `:dev` to `:semver` tags
2. argocd-image-updater scans Harbor every ~3 min, finds new `0.x` tag
3. IU writes back to this repo's `base/kustomization.yaml` (`newTag: ...`)
4. Argo syncs the new tag, rolls the pod

## Routing

This repo does NOT define the HTTPRoute — `gateway-services` (the umbrella chart in the homelab repo) owns all gateway routing entries. The dice-roller's `dice.sirddail.net` host is registered there separately.

## Image source

[harbor.sirddail.net/ai/tor-dice](https://harbor.sirddail.net/harbor/projects/ai/repositories/tor-dice) — pushed by [tor-dice-docker](https://github.com/dvystrcil/tor-dice-docker)'s build workflows.

## License

[MIT](LICENSE). Same dual-license terms as the sibling tor-dice-docker.
