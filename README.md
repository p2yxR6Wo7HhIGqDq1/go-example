# go-example

Step 1 is a simple GitHub Fork; there's not a lot to say about that except that you need to do it, because you want control of the repository that you're deploying from.

Step 2 is just cloning the repository to your local system so that you can edit and push changes to it.

Step 3 creates a fly.toml file to go into the repository.
A note about fly.toml in repositories: Usually, when we ship examples, we avoid putting the fly.toml file in the repository by including fly.toml in the .gitignore file. And users should be creating their own fly.toml with the fly launch command. When using GitHub Actions though, you want your fly.toml in the repository so that the action can use it in the deployment process.
API Tokens

Step 4 is about getting an API token. You can generate a deploy token to use to authorize a specific application. That's what flyctl tokens create deploy -x 9999999h gives you. For a more powerful token that can manage multiple applications, run flyctl auth token.

Steps 5 and 6 make your new token available to GitHub Actions that run against your repository. You'll add the token as a secret in the repository's settings. Under the Settings tab, go to Secrets and variables and select Actions. Click on the green "New repository secret" button, enter the name as FLY_API_TOKEN, and copy the token as the secret.

If you'd prefer an environment secret instead, make sure you list the environment you selected in your deploy step. For example:

deploy:
    name: Deploy app
    runs-on: ubuntu-latest
    environment: production

Building the workflow and Deployment

Step 7 is the heart of the process, where we put in place a workflow. Now, GitHub has a UI which allows you to select and edit workflows, but you can also modify them as part of the repository. So we create .github/workflows/fly.yml - you'll likely want to mkdir -p .github/workflows to quickly create the directories - and load up the file with a GitHub Action recipe. We'll go through it line by line now:

name: Fly Deploy

This sets the displayed name for the action.

on:
  push:
    branches:
      - master

When should this action be run. There's lots of options but in this case, in response to any push to the repository's master branch. If your repository uses a default branch other than master, such as main, then you should change that here.

jobs:
  deploy:
      name: Deploy app
      runs-on: ubuntu-latest

So an action is made up of named jobs, in this case one to deploy the application. The jobs run on a virtual machine. Here we are giving the "deploy" job the name "Deploy app" and telling GitHub Actions to run it on a virtual machine with the latest version of Ubuntu on it. The next part is to set up the steps needed to complete this job.

      steps:
        - uses: actions/checkout@v3

The first step is one of the built in Actions steps. The step uses the checkout@v3 action which checks out the repository into a directory on the virtual machine. We are now ready to deploy.

        - uses: superfly/flyctl-actions/setup-flyctl@master
        - run: flyctl deploy --remote-only

This step uses the superfly/flyctl-actions action. This is a GitHub action created by Fly.io which wraps around the flyctl command. The wrapper is invoked with the deploy argument which will take over the process of building and moving the application to the Fly.io infrastructure. It uses the settings from the fly.toml file to guide it and uses the FLY_API_TOKEN to authorize its access to the Fly.io GraphQL API.

          env:
            FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}

Here is where we pull the API token from GitHub's secrets engine and put it into the environmental variables passed to the action.

Step 8 pushes your two new files to the repository: fly.toml, the Fly Launch configuration file, and fly.yml, the GitHub action file. The push triggers your first automatic deploy. The GitHub action now triggers a redeploy each time you push changes to your repo.