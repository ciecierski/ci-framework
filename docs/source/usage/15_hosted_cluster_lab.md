# Management OCP + Hosted Control Plane (lab)

Constrained-lab path for a management OpenShift cluster and a KubeVirt
hosted control plane, without OSP 17.1 VMs. Requires nested virtualization
(CPU passthrough) on the hypervisor.

## Prerequisites

- Host roughly in the 128 GiB / 24 CPU class (or larger)
- Pull secret / CI token / OpenShift version patches as for a normal
  `create-infra` / `deploy-ocp` run
- `~/.ssh/id_cifw{.pub,}` on the host that runs `deploy-hosted-cluster.yml`
  (or override `cifmw_hosted_cluster_ssh_*`)

## Deploy

```bash
# 1) Infra (networks, tiny controller-0; OCP disks prepared, not started)
ansible-playbook create-infra.yml \
  -e @scenarios/reproducers/networking-definition.yml \
  -e @scenarios/reproducers/va-hci.yml \
  -e @scenarios/reproducers/va-hci-mgmt-ocp-only.yml \
  -e @<secrets> --flush-cache

# 2) Management OCP
ansible-playbook deploy-ocp.yml \
  -e @scenarios/reproducers/networking-definition.yml \
  -e @scenarios/reproducers/va-hci.yml \
  -e @scenarios/reproducers/va-hci-mgmt-ocp-only.yml \
  -e @<secrets> --flush-cache

# 3) Hosted control plane (verify management_interface + LVMS disks first)
MGMT_KC=$(echo ~/src/github.com/openshift-metal3/dev-scripts/ocp/*/auth/kubeconfig)
ansible-playbook deploy-hosted-cluster.yml \
  -e cifmw_openshift_kubeconfig="$MGMT_KC" \
  -e @scenarios/reproducers/va-hci-hosted-cluster-lab.yml \
  --flush-cache
```

## Cleanup HCP only

```bash
ansible-playbook cleanup-hosted-cluster.yml \
  -e cifmw_openshift_kubeconfig="$MGMT_KC" \
  -e @scenarios/reproducers/va-hci-hosted-cluster-lab.yml \
  --flush-cache
```

Always pass the **management** kubeconfig and `--flush-cache` after a
successful HCP deploy (retarget caches the guest kubeconfig).

## Related files

- `scenarios/reproducers/va-hci-mgmt-ocp-only.yml`
- `scenarios/reproducers/mgmt-ocp-only-adoption/hci.yml`
- `scenarios/reproducers/va-hci-hosted-cluster-lab.yml`
- `deploy-hosted-cluster.yml` / `cleanup-hosted-cluster.yml`
- `roles/hosted_cluster/README.md`
