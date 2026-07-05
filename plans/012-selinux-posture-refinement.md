# Plan 012 (post-v3.0): SELinux posture refinement
- **Priority**: P3 | **Effort**: M | **Risk**: MED | **Category**: security | **Status**: TODO
Verdict from the 2026-07 review: v3 SELinux is fundamentally right (enforcing by default, distro policy RPMs baked into Leap images, per-pool escape hatch, loud fail when the rke2 module is missing - proven in CI). Three bounded refinements:
1. Document every allow rule in both .te files with its originating issue/incident (rules accreted incident-driven; future maintainers cannot tell which are still needed).
2. Unify templates/kube-hetzner-selinux.te (MicroOS path) and templates/k8s-custom-policies.te (Leap path) into one source with distro conditionals - the split already drifted once (kernel_t tcp rule, #2203).
3. Add an AVC-clean assertion to the live gate/CI: post-apply `ausearch -m avc -ts boot` must show no denials on the happy path for every preset - converts "open enough?" from opinion to evidence, and would flag future over- or under-permissioning.
Optional investigation: whether `container_t unreserved_port_t:tcp_socket { name_bind ... }` (the broadest rule) can be narrowed without breaking NodePort/hostNetwork workloads - evidence from (3) first.
