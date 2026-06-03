<div align="center">

# 🚀 vxcloud Deploy — GitHub Action

**Deploy, provision, or install _anything_ on [vxcloud](https://vxcloud.io) — straight from a GitHub Actions workflow.**

It installs [`vxcli`](https://vxcloud.io/download/cli), authenticates with your API key, and runs the command you give it — the same control plane your terminal and the web app use.

[![Marketplace](https://img.shields.io/badge/Marketplace-vxcloud%20Deploy-2088FF?logo=githubactions&logoColor=white)](https://github.com/marketplace/actions/vxcloud-deploy)
[![Release](https://img.shields.io/github/v/release/prodxcloud/vxcloud-deploy-action?label=release&color=blue)](https://github.com/prodxcloud/vxcloud-deploy-action/releases)
[![Powered by vxcli](https://img.shields.io/badge/powered%20by-vxcli-00c8ff)](https://vxcloud.io/download/cli)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue)](./LICENSE)

</div>

---

## ⚡ Quick start

```yaml
- name: Deploy to vxcloud
  uses: prodxcloud/vxcloud-deploy-action@v1
  with:
    username: ${{ secrets.VXCLOUD_USERNAME }}
    api-key:  ${{ secrets.VXCLOUD_API_KEY }}
    command:  deploy fastapi --source-dir ./ --app-name api
              --host ${{ vars.HOST }} --ssh-user ubuntu --key-pair-name AWSPRODKEY1
              --entry app.app:app --requirements requirements.txt
              --enable-ssl --domain api.example.com --ssl-email ops@example.com
```

That's it — on every push, your app is built, shipped to your VM, and served over
HTTPS. `command` is just raw `vxcli` arguments, so **anything vxcli can do, this
action can do.**

## 🧰 What you can run

| Capability | Example `command:` |
|---|---|
| 🚢 **Deploy a stack** (18 stacks) | `deploy nextjs --source-dir ./ --app-name web --host <ip> --ssh-user ubuntu --key-pair-name KEY --enable-ssl --domain www.example.com --ssl-email ops@example.com` |
| 🐳 **Deploy a container** | `deploy container --image grafana/grafana:latest --name grafana --ports 3000:3000 --host <ip> --ssh-user ubuntu --key-pair-name KEY` |
| ☁️ **Provision a VM** (5 clouds) | `vm create --name api --cloud aws --region us-east-1 --instance-type t3.small --key-pair-name KEY` |
| 📦 **Install a service** | `install prometheus --host <ip> --ssh-user ubuntu --key-pair-name KEY` |
| 🔁 **Manage services** | `services restart api --host <ip> --ssh-user ubuntu --key-pair-name KEY` |
| 🗄️ **Managed database** | `deploy metaldb --resource-name app-db --host <ip> --ssh-user ubuntu --key-pair-name KEY` |
| ☸️ **Kubernetes** | `kubernetes create --name prod --cloud aws --region us-east-1 --node-count 3` |
| 🤖 **Run an AI agent** | `vxcomputer run --objective "Check disk usage and report" --channel cloudshell` |
| 🛒 **Marketplace** | `marketplace agents deploy prompt_agent --host <ip> --ssh-user ubuntu --key-pair-name KEY` |

…and everything else in `vxcli --help`. Run `command:` empty to just install +
authenticate, then call `vxcli` directly in later steps.

## 🔧 Inputs

| Input | Required | Default | Description |
|---|:--:|---|---|
| `username` | ✅ | — | The vxcloud username your API key belongs to |
| `api-key` | ✅ | — | Developer key (`xc_live_*` / `xc_dev_*`) — **store as a secret** |
| `command` | — | `''` | `vxcli` command **without** the leading `vxcli`. Empty = install + auth only |
| `version` | — | `latest` | vxcli version to install |
| `working-directory` | — | `.` | Directory to run the command in |

## 📤 Outputs

| Output | Description |
|---|---|
| `version` | The vxcli version that was installed |

## 📚 Examples

Ready-to-copy workflows in [`examples/`](./examples):

| File | Shows |
|---|---|
| [deploy-to-production.yml](./examples/deploy-to-production.yml) | Build, test, deploy a stack on push to `main` |
| [deploy-container-https.yml](./examples/deploy-container-https.yml) | Deploy a Docker container with Let's Encrypt HTTPS |
| [provision-vm-and-deploy.yml](./examples/provision-vm-and-deploy.yml) | Provision a VM **then** deploy to it (full IaC) |
| [install-services.yml](./examples/install-services.yml) | Install Prometheus + Grafana on a schedule |
| [ai-devops-task.yml](./examples/ai-devops-task.yml) | Run a governed AI agent / nightly ops sweep |

## 🧩 Usage patterns

**One-shot deploy** — pass a `command`:
```yaml
- uses: prodxcloud/vxcloud-deploy-action@v1
  with:
    username: ${{ secrets.VXCLOUD_USERNAME }}
    api-key:  ${{ secrets.VXCLOUD_API_KEY }}
    command:  deploy container --image myorg/api:latest --name api --host ${{ vars.HOST }} --ssh-user ubuntu --key-pair-name KEY
```

**Set up once, run many** — leave `command` empty (like `setup-node`):
```yaml
- uses: prodxcloud/vxcloud-deploy-action@v1
  with:
    username: ${{ secrets.VXCLOUD_USERNAME }}
    api-key:  ${{ secrets.VXCLOUD_API_KEY }}
- run: vxcli vm create --name api --cloud aws --region us-east-1 --instance-type t3.small --key-pair-name KEY
- run: vxcli deploy container --image myorg/api:latest --name api --host ${{ vars.HOST }} --ssh-user ubuntu --key-pair-name KEY
- run: vxcli services list --host ${{ vars.HOST }} --ssh-user ubuntu --key-pair-name KEY
```

**Prefer the SDK?** Skip the action and script it:
```yaml
- uses: actions/setup-python@v5
  with: { python-version: '3.12' }
- run: pip install vxsdk
- run: python deploy.py
  env:
    VXCLOUD_USERNAME: ${{ secrets.VXCLOUD_USERNAME }}
    VXCLOUD_API_KEY:  ${{ secrets.VXCLOUD_API_KEY }}
```

## 🔐 Secrets & variables

Add under **Settings → Secrets and variables → Actions**:

| Kind | Name | Value |
|---|---|---|
| Secret | `VXCLOUD_USERNAME` | your vxcloud username |
| Secret | `VXCLOUD_API_KEY` | a key from [app.vxcloud.io/developer/keys](https://app.vxcloud.io/developer/keys) |
| Variable | `HOST` | target VM IP / hostname (used by the examples) |

The API key is passed via an environment variable and never printed; GitHub also
masks secret values in logs.

## 📝 Notes

- **Runners:** Linux and macOS (the installer auto-detects OS/arch).
- **Quoting:** `command` is word-split, so most flag strings work as-is. For
  arguments containing spaces/quotes, use the *set-up-once* pattern and a plain
  `run: vxcli …` step for full shell control.
- **Auth scope:** the key authenticates to your tenant; deploys target the VMs and
  keys in your workspace Vault — nothing is stored in the runner.

---

<div align="center">

**[vxcloud.io](https://vxcloud.io)** · [CLI](https://vxcloud.io/download/cli) · [Docs](https://vxcloud.io/docs) · [SDK](https://github.com/prodxcloud/vxcloud) · [Node image](https://hub.docker.com/r/vxcloud/vxnode)

© PRODXCLOUD · Apache-2.0

</div>
