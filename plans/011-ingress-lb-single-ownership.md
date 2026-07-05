# Plan 011 (design): single ownership for the ingress load balancer lifecycle

## Status
- **Priority**: P2 | **Effort**: M (design) / L (implement) | **Risk**: HIGH | **Depends on**: none | **Category**: tech-debt (design)
- **Planned at**: commit 061d9af+, 2026-07-05 | **Status**: TODO

## Problem (live + CI evidence)
The ingress LB has two owners: terraform creates `hcloud_load_balancer.cluster` (+network attachment, +target) AND the hcloud CCM adopts it via the name annotation. On destroy this dual ownership cannot be sequenced correctly:
- Without graceful Service deletion: the LB orphans (original live-gate bug), blocking network deletion.
- With graceful deletion (current cleanup): CCM deletes the LB; terraform's `hcloud_load_balancer_network` detach then fails 422 "resource not found" (6/6 CI destroy failures at 061d9af when a settle wait made CCM deletion deterministic).
- Without a settle: benign already-detaching race (~50% of CI destroys), absorbed by the CI destroy retry.

## Design task
Choose and specify ONE owner:
(a) CCM-only: terraform stops creating the ingress LB; annotations let CCM create it; terraform learns the IP post-hoc (output via data source with retry/depends chain, or drop the IP output in favor of a documented lookup). Assess: outputs/back-compat, `combine_load_balancers_effective` interplay, kube.tf.example/docs, upgrade path for existing state (removed blocks + state rm guidance).
(b) Terraform-only: CCM adoption stays but Service deletion must NOT delete the LB — investigate hcloud CCM ownership semantics/labels for adopted-vs-created LBs and any keep-on-delete mechanism; if none exists upstream, this option requires an upstream contribution.
Deliverable: decision doc with upgrade-safe migration, then an implementation plan.
