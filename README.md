# CI/CD Usage Documentation

## Overview
The CI/CD workflows streamline the deployment of AI Hub applications across different environments. This automation facilitates the migration of applications from a source environment to a target environment using GitHub Actions.

---

##Configuration File Setup
1. Create a feature branch with a name starting as `feature/*` (required).
2. Add a `config.json` file in your feature branch with the following structure:

```json
{
  "source": {
    "is_advanced": true/false, 
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
    "rebuild": true/false
  },
  "regression": {},
  "release_notes": "<Release Notes>"
}
```

#### Configuration Parameters:

**App Types:**
- **Build App**: Set `is_advanced` to `false` and specify `project_id`.
- **Advanced App**: Set `is_advanced` to `true` and specify `flow_path`.
- **Solution Build App**: Set `is_advanced` to `true` and specify both `sb_name` and `flow_name`.

**Mandatory for All Apps:**
- `org` and `workspace`: Required for both source and target environments.
- `app_id`: Needed when migrating an app.
- `deployment_id`: Needed when migrating a deployment.

**Optional Settings:**
- `dependencies`: List of dependencies for Advanced or Solution Build apps.
- `rebuild`: Set to `true` to recreate the Build App project in the target environment.

**For creating new app or add a Version in Source:**
- Leave `app_id` blank.
- Provide `app_details` including the app's name, version, description, and release notes.

**Release Notes:**
- Document the changes made to the app (this is required).

---

## Instructions to Trigger the Workflows

### 1. Ensure Workflow Permissions
Make sure you have workflow permissions set in the repository’s settings.

### 2. Update the Configuration File
Modify the `config.json` file with your project details.

### 3. Commit and Push Changes
1. Commit the updated `config.json` file to your feature branch.
2. Push it to the remote repository.

### 4. Trigger CI/CD Workflow
- The CI/CD workflow will automatically fetch the project details from the source environment and commit the project data to the feature branch.
- Continue working on your project, and when you want to fetch the latest project data, update the config file (e.g., updating release notes for app changes).
- Commit again to trigger the CI/CD workflow, ensuring that the latest project details are fetched into the feature branch.
- Follow Git best practices such as pulling before committing, resolving merge conflicts, etc.

### 5. Migrate to Target Environment
1. Once your app is ready for migration, ensure all changes are committed and pushed to your `feature/*` branch.
2. The **Migrate to Target** workflow will be triggered automatically after the **Fetch Latest Code** workflow successfully completes on your `feature/*` branch—**merging into `main` is not required**.
3. The migration workflow will:
    - Validate your `config.json` and necessary fields.
    - Install dependencies and migration toolkit.
    - Create or update the project and app in the target environment, with rollback logic for failures.
    - Create the deployment if specified.
    - Create or update a Git tag for the app version.
    - Roll back or delete builds if app creation fails.
4. Monitor workflow progress and results in the PR checks or the GitHub Actions tab for your branch.

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
