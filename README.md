# RHEL

Ansible playbooks for Red Hat Enterprise Linux (RHEL) day-2 operations, subscription management, patching, and demo environment setup. Designed to run from Ansible Automation Platform (AAP) Controller.

---

## Playbooks

### Patching & Package Management

| Playbook | Description |
|---|---|
| `yumupdate.yml` | Apply all available yum/dnf updates |
| `yuminstall.yml` | Install a specific package |
| `yumremove.yml` | Remove a package |
| `dnfinstall.yml` | Install a package using dnf (RHEL 8+) |
| `RollbackYum.yml` | Roll back the last yum transaction |

### Subscription & Registration

| Playbook | Description |
|---|---|
| `subregister.yml` | Register a host with Red Hat Subscription Manager |
| `subregistersat.yml` | Register a host with a Red Hat Satellite server |
| `subunregister.yml` | Unregister a host from RHSM |
| `setRHEL7repos.yml` | Enable required RHEL 7 repositories |
| `setsatRHEL7repos.yml` | Enable RHEL 7 repositories via Satellite |

### Red Hat Insights

| Playbook | Description |
|---|---|
| `reginsights.yml` | Register a host with Red Hat Insights |
| `unreginsights.yml` | Unregister a host from Insights |
| `insightrole.yml` | Configure Insights using a role |
| `rollbackupdateinsights.yml` | Roll back an Insights-triggered update |

### Services & httpd

| Playbook | Description |
|---|---|
| `starthttpd.yml` | Start the Apache httpd service |
| `stophttpd.yml` | Stop the Apache httpd service |

### Users & SSH

| Playbook | Description |
|---|---|
| `createuser.yml` | Create a local user account |
| `authkeys.yml` | Deploy authorized_keys for SSH access |

### Inventory & Reporting

| Playbook | Description |
|---|---|
| `get_os_version.yml` | Return OS version as a registered fact |
| `checkhostsup.yml` | Ping inventory to determine reachable hosts |
| `CreateHTMLinventory.yml` | Generate an HTML inventory report |

### File Management

| Playbook | Description |
|---|---|
| `checkfilediffapply.yml` | Check a file diff and apply if changed |
| `checkfilediffnoapply.yml` | Check a file diff without applying |

### Demo Environment

| Playbook | Description |
|---|---|
| `setupmachine.yml` | Baseline configure a demo RHEL machine |
| `readydemomachine.yml` | Prepare a demo machine (packages, services) |
| `cleanupdemomachines.yml` | Reset demo machines to initial state |

### Notifications

| Playbook | Description |
|---|---|
| `slackafailedtaskalert.yml` | Send a Slack alert when a task fails |

---

## Collections Required

```yaml
# collections/requirements.yml
collections:
  - name: community.general
  - name: ansible.posix
  - name: redhat.insights
```

---

## Variable Convention

Variables are passed via AAP Controller surveys or extra vars. Common variables:

```yaml
package_name: ''        # package to install/remove
rhsm_username: ''       # passed from RHSM credential
rhsm_password: ''       # passed from RHSM credential
satellite_url: ''       # Satellite server URL
insights_account: ''    # Insights account number
```
