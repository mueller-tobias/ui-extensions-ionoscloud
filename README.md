# ui-plugin-examples
Extensions for the Rancher Dashboard for Ionoscloud.

This repository contains the following extensions in the `pkg` folder.

|Name|Description|Rancher Version|
|----|-----------|---------------|
|ionoscloud|Adds Machine Config and Cloud Credeantials for the ionoscloud rancher driver|v2.7.0|

To use these example in Rancher Manager, add this repository as a Helm Repository - to do this:

- Go to the local cluster, to 'Apps' and 'Repositories'
- Click 'Create' and enter a name, e.g. 'example-extensions'
- Choose Target 'Git repository containing Helm chart or cluster template definitions'
- Enter `https://github.com/ionos-cloud/ui-extensions-ionoscloud` for the 'Git Repo URL'
- Enter `gh-pages` for the Git Branch
- Click `Create`

The extensions should then appear on the 'Extenssions' page in Rancher Manager.

## Building and running locally

You can build and run the extensions locally, to do so:

- Run `yarn install`
- Set the API environment variable to point to a Rancher backend
- Run Rancher in development mode with `yarn dev`
- Open a web browser to `https://127.0.0.1:8005`

Once you login, you should see Rancher load with the extensions automatically loaded. You can edit the code for the extensions
and then should hot-reload within the browser.

### Bugs & Issues
Please submit bugs and issues to [ionos-cloud/ui-extensions-ionoscloud](https://github.com/ionos-cloud/ui-extensions-ionoscloud/issues).

Or just [click here](https://github.com/ionos-cloud/ui-extensions-ionoscloud/issues/new) to create a new issue.

