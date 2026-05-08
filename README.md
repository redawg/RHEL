# RHEL

Ansible playbooks for Red Hat Enterprise Linux day-2 operations — subscription management, patching, user provisioning, Insights, and demo environment setup. Designed to run from Ansible Automation Platform (AAP) Controller with variables supplied via surveys or credential types.

---

## Playbooks

### Subscription Management

#### `subregister.yml`
Registers a RHEL host with Red Hat Subscription Manager (RHSM) using username/password and attaches a specific pool. Sets system purpose (role, usage, SLA).

**Variables:**
```yaml
username: ''        # RHSM username
password: ''        # RHSM password
pool_id: ''         # Subscription pool ID
systemrole: "Red Hat Enterprise Server"
usage: "Dev/Test"
sla: "Self-Support"
```

---

#### `subregistersat.yml`
Registers a RHEL host to a **Red Hat Satellite** server using an organization ID and activation key.

**Variables:**
```yaml
server_hostname: ''   # Satellite FQDN
org: ''               # Organization ID
activationkey: ''     # Activation key name
pool_id: ''
systemrole: "Red Hat Enterprise Server"
usage: "Dev/Test"
sla: "Self-Support"
```

---

#### `subunregister.yml`
Unregisters a host from RHSM (`state: absent`). Use before decommissioning or rebuilding.

No variables required.

---

### Repository Management

#### `setRHEL7repos.yml`
Disables all repositories and enables the standard RHEL 7 repositories:
- `rhel-7-server-rpms`
- `rhel-7-server-extras-rpms`

Only runs when `ansible_distribution == "RedHat"` and major version is `7`.

---

#### `setsatRHEL7repos.yml`
Disables all repositories and enables the Satellite 6.7 repository set for RHEL 7:
- `rhel-7-server-rpms`
- `rhel-7-server-satellite-6.7-rpms`
- `rhel-server-rhscl-7-rpms`
- `rhel-7-server-satellite-maintenance-6-rpms`
- `rhel-7-server-ansible-2.9-rpms`

Runs `yum clean metadata` after enabling. Only runs on RHEL 7.

---

### Patching & Package Management

#### `yumupdate.yml`
Updates all installed packages to the latest available version.

```yaml
# No variables required
- yum:
    name: '*'
    state: latest
```

---

#### `yuminstall.yml`
Installs a single package by name using `yum`. Sends a Slack notification on success if `slack_token` is defined.

**Variables:**
```yaml
package: ''          # Name of the package to install
slack_token: ''      # Optional: Slack bot token for install notification
```

---

#### `dnfinstall.yml`
Installs a package using `dnf` (RHEL 8+). Runs serially (`serial: 1`) across hosts.

**Variables:**
```yaml
package: ''          # Name of the package to install
```

---

#### `yumremove.yml`
Removes a package from all target hosts using `yum`.

**Variables:**
```yaml
package: ''          # Name of the package to remove
```

---

#### `RollbackYum.yml`
Rolls back the last yum transaction using `yum history rollback last-1`. Useful for undoing a bad update. Prints the rollback output.

No variables required.

---

#### `rollbackupdateinsights.yml`
Rolls back the last yum transaction **and** runs `insights-client` to sync the new state with Red Hat Insights.

No variables required.

---

### Red Hat Insights

#### `reginsights.yml`
Registers a host with Red Hat Insights by running `insights-client --register`.

No variables required.

---

#### `unreginsights.yml`
Unregisters a host from Red Hat Insights by running `insights-client --unregister`.

No variables required.

---

#### `insightrole.yml`
Applies the `RedHatInsights.insights-client` Ansible role. Runs only on RedHat family OS, version 6 or higher.

Requires the `RedHatInsights.insights-client` role to be installed.

---

### Users & SSH

#### `createuser.yml`
Creates an Ansible service account on target hosts with optional sudo access.

**Variables:**
```yaml
svcansible_username: ''       # Service account name
svcansible_home: ''           # Home directory base path
svcansible_public_key: ''     # SSH public key string
svcansible_sudoer: 'false'    # Set to 'true' to add NOPASSWD sudo rule
```

---

#### `authkeys.yml`
Deploys an SSH public key from a local file (`/home/sshkey/redpub`) to the `authorized_keys` of a specified user.

**Variables:**
```yaml
key_user: ''     # The user account to add the key to
```

---

### Services

#### `starthttpd.yml`
Starts the `httpd` service on all target hosts. Uses `state: started` so it is idempotent.

No variables required.

---

#### `stophttpd.yml`
Stops the `httpd` service on all target hosts. Used in demos to trigger a Dynatrace problem alert.

No variables required.

---

### Configuration Drift / File Management

#### `checkfilediffnoapply.yml`
Clones a source file repository, compares a file to its live version in `/etc/` using `check_mode: yes` and `diff: yes`, and generates an HTML diff report on a container host. **Does not apply changes.**

**Variables:**
```yaml
sourcefilerepo: ''     # Git repo URL containing source config files
file2comp: ''          # Filename to compare (e.g. resolv.conf)
home_dir: ''           # Directory on container host for report output
container_host: ''     # Hostname of the container/report host
```

---

#### `checkfilediffapply.yml`
Same as above but **applies the change** if the file differs. Does not generate an HTML report.

**Variables:**
```yaml
sourcefilerepo: ''     # Git repo URL containing source config files
file2comp: ''          # Filename to compare and apply
```

---

### Reporting

#### `CreateHTMLinventory.yml`
Generates an HTML OS inventory report using a Jinja2 template (`report-osversion.j2`) and saves it to a web-accessible directory on a report host. Sends a Slack notification with the report URL when complete.

**Variables:**
```yaml
www_dir: ''          # Target directory for the HTML report on the report host
slack_token: ''      # Slack bot token
```

---

### Connectivity

#### `checkhostsup.yml`
Tests SSH reachability across the inventory. Reachable hosts are dynamically added to the `running_hosts` group (used by downstream plays). Hosts that fail send a Slack alert with task failure details.

**Variables:**
```yaml
slack_token: ''      # Slack bot token
channel: ''          # Slack channel (e.g. #ansible-alerts)
slackuser: ''        # Display name for the Slack message
```

---

### Notifications

#### `slackafailedtaskalert.yml`
Sends a formatted Slack alert containing the hostname, failed task name, action, and full error JSON. Designed to be called from an `on_error` notification template or a rescue block.

**Variables:**
```yaml
slack_token: ''      # Slack bot token
message: ''          # Custom message text
channel: ''          # Slack channel
slackuser: ''        # Display name for the Slack message
```

---

### Demo Environment

#### `readydemomachine.yml`
Full baseline provisioning playbook for demo RHEL machines. Performs the complete setup sequence:

1. Waits for SSH connectivity and groups live hosts as `running_hosts`
2. Subscribes to RHSM (supports both RHEL-only and RHEL+Ansible pool combos)
3. Configures correct repositories for RHEL 7 or RHEL 8
4. Installs and starts Cockpit on RHEL 8 AWS instances (creates `cockpitadmin` user)
5. Deploys SSH banner, MOTD, and `/etc/issue` from local files
6. Installs baseline packages: `insights-client`, `net-tools`, `wget`, `mlocate`, `nc`, `telnet`
7. Updates all packages to latest
8. Registers with Red Hat Insights
9. Records timestamps at key stages via `set_stats` for workflow visibility

**Variables:**
```yaml
username: ''        # RHSM username
password: ''        # RHSM password
pool_id: ''         # Subscription pool ID
ansiblenode: ''     # 'true' to attach Ansible pool alongside RHEL pool
slack_token: ''     # Slack bot token (for SSH wait failure alerts)
channel: ''         # Slack channel
slackuser: ''       # Slack display name
systemrole: "Red Hat Enterprise Server"
usage: "Dev/Test"
sla: "Self-Support"
```

---

#### `cleanupdemomachines.yml`
Resets a demo machine by unregistering from Insights and removing the RHSM subscription. Run this before re-provisioning a demo host.

No variables required.

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

## AAP Controller Setup

All sensitive values (RHSM credentials, Slack tokens, Satellite activation keys) should be stored in AAP Controller as **custom credential types** or passed via **job template surveys** — never hardcoded in playbooks.

Example custom credential type for RHSM:

**Input Configuration:**
```yaml
fields:
  - id: username
    type: string
    label: RHSM Username
  - id: password
    type: string
    label: RHSM Password
    secret: true
  - id: pool_id
    type: string
    label: Pool ID
```

**Injector Configuration:**
```yaml
extra_vars:
  username: '{{ username }}'
  password: '{{ password }}'
  pool_id: '{{ pool_id }}'
```
