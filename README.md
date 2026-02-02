# CI/CD Usage Documentation

## Overview

The CI/CD workflows streamline the deployment of AI Hub applications across different environments. This automation facilitates the migration of applications from a source environment to a target environment using GitHub Actions.

```
┌─────────────────────┐                      ┌─────────────────────┐
│   SOURCE (Dev)      │                      │   TARGET (Prod)     │
│                     │                      │                     │
│  • Compile solution │    Fetch & Commit    │                     │
│  • Download app     │  ─────────────────>  │  • Upload solution  │
│                     │                      │  • Publish app      │
│                     │     Migrate          │  • Create deployment│
│                     │  ─────────────────>  │                     │
└─────────────────────┘                      └─────────────────────┘
```

---

## Prerequisites

### 1. Repository Secrets Setup

Go to your repository → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

#### Required Secrets

| Secret Name         | Description                                                            |
| ------------------- | ---------------------------------------------------------------------- |
| `SOURCE_HOST_URL` | Source (Dev) Instabase host URL (e.g.,`https://dev.instabase.com`)   |
| `SOURCE_TOKEN`    | Source (Dev) API token                                                 |
| `TARGET_HOST_URL` | Target (Prod) Instabase host URL (e.g.,`https://prod.instabase.com`) |
| `TARGET_TOKEN`    | Target (Prod) API token                                                |

#### mTLS Certificates (Required if mTLS is enabled)

If your Instabase environments require mutual TLS authentication, add these secrets:

| Secret Name                  | Description                                |
| ---------------------------- | ------------------------------------------ |
| `MTLS_SOURCE_CERT_CONTENT` | Source (Dev) certificate PEM file content  |
| `MTLS_SOURCE_KEY_CONTENT`  | Source (Dev) private key PEM file content  |
| `MTLS_TARGET_CERT_CONTENT` | Target (Prod) certificate PEM file content |
| `MTLS_TARGET_KEY_CONTENT`  | Target (Prod) private key PEM file content |

> **How to add certificate content:** Open your `.pem` file in a text editor, copy the entire content (including `-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----` lines), and paste it as the secret value.

#### Proxy Configuration (Optional)

| Secret Name        | Description                       |
| ------------------ | --------------------------------- |
| `PROXY_HOST`     | Proxy server hostname             |
| `PROXY_PORT`     | Proxy server port                 |
| `PROXY_USER`     | Proxy username (if auth required) |
| `PROXY_PASSWORD` | Proxy password (if auth required) |

---

## Configuration File Setup

### 1. Create a Feature Branch

Create a feature branch with a name starting as `feature/*` (required).

```bash
git checkout -b feature/my-app-migration
```

### 2. Add Configuration File

Add a `config.json` file in your feature branch with the following structure:

```json
{
  "source": {
    "is_advanced": true,
    "project_id": "<Build Project ID>",
    "flow_path": "<Advanced App Flow Path>",
    "sb_name": "<Solution Builder Name>",
    "flow_name": "<Solution Build Flow Name>",
    "org": "<Source Organization>",
    "workspace": "<Source Workspace>",
    "app_id": "<App ID>",
    "deployment_id": "<Deployment ID>",
    "dependencies": ["<dependency1>==<version>", "..."],
    "app_details": {
      "name": "<App Name>",
      "version": "<Version>",
      "description": "<Description>",
      "release_notes": "<Release Notes for this app version>"
    }
  },
  "target": {
    "org": "<Target Organization>",
    "workspace": "<Target Workspace>",
    "project_id": "<Target Project ID>"
  },
  "settings": {
    "rebuild": true
  },
  "regression": {},
  "release_notes": "<Release Notes>"
}
```

### Configuration Parameters

#### App Types

| App Type                     | Configuration                                                                  |
| ---------------------------- | ------------------------------------------------------------------------------ |
| **Build App**          | Set `is_advanced` to `false` and specify `project_id`                    |
| **Advanced App**       | Set `is_advanced` to `true` and specify `flow_path`                      |
| **Solution Build App** | Set `is_advanced` to `true` and specify both `sb_name` and `flow_name` |

#### Required Fields

| Field             | Description       | Required For               |
| ----------------- | ----------------- | -------------------------- |
| `org`           | Organization name | All apps (source & target) |
| `workspace`     | Workspace name    | All apps (source & target) |
| `app_id`        | Application ID    | Migrating existing app     |
| `deployment_id` | Deployment ID     | Migrating deployment       |

#### Optional Fields

| Field            | Description                                                 |
| ---------------- | ----------------------------------------------------------- |
| `dependencies` | List of dependencies for Advanced or Solution Build apps    |
| `rebuild`      | Set to `true` to recreate the Build App project in target |
| `app_details`  | Required when creating a new app (leave `app_id` blank)   |

---

## Workflow Execution

### Workflow 1: Fetch Latest Code

**Trigger:** Push to `feature/*` branch

**What it does:**

1. Checks out the code
2. Sets up mTLS certificates from secrets
3. Installs the CI/CD toolkit
4. Validates `config.json`
5. Fetches latest app data from source environment
6. Commits the fetched data to the feature branch

### Workflow 2: Migrate to Target

**Trigger:** Automatically runs after "Fetch Latest Code" completes successfully

**What it does:**

1. Checks out the code from the feature branch
2. Sets up mTLS certificates from secrets
3. Installs the CI/CD toolkit
4. Validates `config.json` and `app_id`
5. Creates or uploads the project to target environment
6. Publishes the app
7. Creates deployment (if `deployment_id` is specified)
8. Creates a Git tag for the release version
9. Rolls back on failure

---

## Step-by-Step Instructions

### Step 1: Ensure Workflow Permissions

1. Go to repository **Settings** → **Actions** → **General**
2. Under "Workflow permissions", select **Read and write permissions**
3. Check **Allow GitHub Actions to create and approve pull requests**

### Step 2: Configure Secrets

Add all required secrets as described in the [Prerequisites](#prerequisites) section.

### Step 3: Create Feature Branch and Config

```bash
# Create feature branch
git checkout -b feature/my-migration

# Create config.json with your settings
# (see Configuration File Setup section)

# Commit and push
git add config.json
git commit -m "Add migration config"
git push origin feature/my-migration
```

### Step 4: Monitor Workflow Execution

1. Go to **Actions** tab in your repository
2. Watch the "Fetch Latest Code" workflow run
3. Once complete, "Migrate to Target" will automatically trigger
4. Monitor both workflows for success/failure

### Step 5: Iterate Development

- Continue working on your project in the source environment
- When ready to sync, update `release_notes` in `config.json`
- Commit and push to trigger the fetch workflow again
- Follow Git best practices (pull before commit, resolve conflicts)

### Step 6: Verify Migration

1. Check the target environment for the deployed app
2. Verify the Git tag was created with the app version
3. Test the app in the target environment

---

## Sample Configuration File

```json
{
    "source": {
        "is_advanced": true,
        "project_id": "a56eb7e0-657d-497d-8c32-0c41d6314554",
        "flow_path": "SolEng/CI-CD/fs/Instabase Drive/AdvancedApp.ibflow",
        "sb_name": "DL_App",
        "flow_name": "DL_Flow",
        "org": "SolEng",
        "workspace": "CI-CD",
        "app_id": "a56eb7e0-657d-497d-8c32-0c41d6314554",
        "deployment_id": "0194a7fa-7474-7a0b-9b88-78f82c0684c6",
        "dependencies": ["model_DL_ca58a1844ac048a297157de827858c6b==0.0.8"],
        "app_details": {
          "name": "DL Test",
          "version": "0.0.2",
          "description": "description",
          "release_notes": "release notes"
        }
    },
    "target": {
        "org": "SolEng",
        "workspace": "CI-CD",
        "project_id": "a56eb7e0-657d-497d-8c32-0c41d6314554"
    },
    "settings": {
        "rebuild": false
    },
    "regression": {},
    "release_notes": "Initial release v0.0.1 - changed some prompts"
}
```

---

## Troubleshooting

### Common Issues

| Issue                                       | Cause                        | Solution                                                                |
| ------------------------------------------- | ---------------------------- | ----------------------------------------------------------------------- |
| Workflow fails at "Setup mTLS certificates" | Missing certificate secrets  | Add `MTLS_*_CONTENT` secrets                                          |
| `config.json file not found`              | File not in repository root  | Ensure `config.json` is in the root directory                         |
| `app_id is missing or null`               | Missing app_id for migration | Add `app_id` to source config, or provide `app_details` for new app |
| API authentication failed                   | Invalid or expired token     | Regenerate API token and update secret                                  |
| Certificate error                           | Invalid PEM content          | Ensure full PEM content is copied including headers                     |

### Checking Logs

1. Go to **Actions** tab
2. Click on the failed workflow run
3. Expand the failed step to see detailed logs
4. Look for error messages in the output

---

## Security Notes

- All secrets are encrypted and only exposed to workflows at runtime
- Certificate files are written to `$RUNNER_TEMP` (cleaned up after job)
- Certificates are set with `chmod 600` (owner read/write only)
- Secrets are never logged in workflow output

---
