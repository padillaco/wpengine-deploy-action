# Deploy to WP Engine (GitHub Action)

This repository contains a GitHub action to setup code deployment from a GitHub repository to a WP Engine environment. The Shell script that runs the actual deployment can be found in [this repo](https://github.com/padillaco/action-deploy-to-remote-repository).

## Implementation

### 1. Copy the Example Workflow File

Copy the [.github](/example/.github/) from the example directory to the root of the repository. This folder contains the [wpengine-deploy.yml](/example/.github/workflows/wpengine-deploy.yml) workflow file where actions related to the WP Engine deployment can be added/removed.

### 2. Actions

The workflow file includes 3 actions:

1. Setup Node: Downloads and caches Node.js and adds it to the PATH
2. Install JS Dependencies and Build Theme Assets
3. Deploy to WP Engine

_Note: Actions 1 and 2 are optional, and can be removed if the WordPress theme does not require assets to be built before a deployment._

These actions are triggered when pushing to a specific branch or when a pull request is closed and merged into a branch. Feel free to adjust the branch names for the **on push** or **on pull request** trigger located at the top of the workflow file.

### 3. Variables and Configuration

1. The workflow file template contains the following placeholders that need to be replaced with their actual value:
    - GitHub Repository Branches:
        - [development-branch]
        - [staging-branch]
        - [production-branch]
    - WP Engine Environments:
        - [development-environment]
        - [staging-environment]
        - [production-environment]
    - WordPress Theme Folder
        - [theme-folder]

2. If deployments shouldn't occur for the development, staging, or production environment, you can remove references to those environments and related branches from the workflow file.

3. Configuring the **Deploy to WP Engine** Action
    - Set the path to the code that will be deployed using the `base_directory` variable. The default value is `.`, or the root of the repository.
    - Set the path to the destination directory on the WP Engine environment where the code will be deployed to using the `destination_directory` variable. The default value is `.`, or the WordPress install root of the WP Engine environment.
    - If needed, modify the list of excluded files that should be deployed using the `exclude_list` variable. The default value is: `.git, .github, .gitmodules, node_modules, .ddev`
    - You can delete the related variables for each environment that does not support deployments.

### 4. Generate an SSH Key Pair

WP Engine requires that an SSH public key be added to environment to enable GitPush. See [WP Engine: Git Version Control System](https://wpengine.com/support/git/) for more details.

1. Open the terminal and run:
    ```bash
    ssh-keygen -t ed25519 -C "wpengine-deploy" -f ~/.ssh/wpengine-deploy
    ```
    _You can replace `~/.ssh/wpengine-deploy` with any temporary file name and path since the key pair will be deleted after use._

2. Copy the private key by running:
    ```bash
    cat ~/.ssh/wpengine-deploy | pbcopy
    ```

    Go to the **Settings → Secrets and variables → Actions** page of the repository and add a new repository secret named `WPENGINE_SSH_PRIVATE_KEY` then enter the contents of the private key as the value.

3. Copy the public key by running:
    ```bash
    cat ~/.ssh/wpengine-deploy.pub | pbcopy
    ```

    From the WP Engine dashboard, go to the GitPush section of the related environment to add the public key. Enter `[repository-slug]-github-repo` as the name of the key (be sure to replace [repository-slug]), then enter the contents of the public key.

4. Remove the public and private key:
    ```bash
    rm ~/.ssh/wpengine-deploy ~/.ssh/wpengine-deploy.pub
    ```

---
### Each push and merged pull request into the development, staging, or production branch should now trigger a deployment.