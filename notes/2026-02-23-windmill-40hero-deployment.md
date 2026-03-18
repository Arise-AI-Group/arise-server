# Windmill 4.0 Hero Deployment - 2026-02-23

## What was deployed

Windmill workflow engine at https://wm.40hero.com — separate instance for 4.0 Hero.

**Services:** PostgreSQL 16, Windmill server, default worker, native worker (4 threads)
**Routing:** Traefik → wm-40h-server:8000
**DokPloy ID:** `rftcK1q7c4J8iB5vIJzon` (shared-services project)
**Compose:** `compose/windmill-40hero.yaml`
**DNS Record ID:** `89c5a5441b1f8ffc5732fb6ec8d94338`

## Decisions

- Cloned from Arise Windmill (`compose/windmill.yaml`) with renamed services (`wm-40h-*`) to avoid container name conflicts.
- Same resource allocation: 1 default worker + 1 native worker (4 threads).

## Post-deployment TODO

- [ ] Change default admin password (`admin@windmill.dev` / `changeme`)
- [ ] Set base URL in Instance Settings to `https://wm.40hero.com`
