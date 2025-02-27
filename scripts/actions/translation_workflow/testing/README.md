# Manumatic testing of translation pipeline

## Background

There previously wasn't a way to run through the code components of the translation pipeline w/o pushing the code into the mainline branch, and running the workflows on github. We do have unit tests, but the coverage isn't complete, and may not cover interactions we care about -- connecting to the database, interacting with smartling api. 

The hope with this script is that it is a decent first attempt at being able to proactively test the pipeline end-to-end. You can run the script, and for some small controlable input, it will execute the primary steps of the translation process in less than a minute and give you general feedback. This is not necesarilly the best way to test, nor is it the end goal of what can be done, but is hopefully a decent iteration for the moment.

# Prerequisites

Most of the manual work involved is getting everything you need to be able to run the script. You will need to fill out most of the empty values for variables in the script, as well as make edits to a file.

## Fill in script variables

You'll need to fill out values for variables in `script.sh`.

1. Provide a github token. A GitHub token can be generated through the GitHub UI. 

  ```sh
  export GITHUB_TOKEN='' # token with no permissions
  ```

2. Provide translation variables. Values for the translation variables can be found through the vendor UI.

  ```sh
  export TRANSLATION_VENDOR_PROJECT='' # use machine translation project
  export TRANSLATION_VENDOR_USER='' # MT user
  export TRANSLATION_VENDOR_SECRET='' # MT user secret
  ```


3. Finally, the DB_CONNECTION_INFO secret needs to be populated in `connection_info.json`. The recommendation for this is to spin up a local postgres instance and connect to that, so that you dont need to test against a resource in the cloud. See the next section for setting up a local postgres instance on Docker. 

## How to spin up a local postgres instance on Docker

1. Run the following command to start a postgres container, with credentials  `user = root` and `password = root`:
    ```bash
    docker run -d --env POSTGRES_USER=root --env POSTGRES_PASSWORD=root --env POSTGRES_DB=translations --name postgres -p 5432:5432 postgres
    ```

2. Update the contents of `connection_info.json`:

    ```JSON
    {
        "username": "root",
        "password": "root",
        "host": "localhost",
        "database": "translations"
    }
    ```
3. Create the database by running `creation_and_cleanup.sql` on your postgres instance. See the next section to do this in pgadmin.

## How to start a pgadmin container to interact with your postgres container (optional)

This step is not a prerequisite for running the script, but may be useful for creating your database (which is required) and debugging. 

1. Run the following to start a pgadmin container:
    ```bash
    docker run -d --env PGADMIN_DEFAULT_EMAIL=username@username.com --env PGADMIN_DEFAULT_PASSWORD=password --name pgadmin -p 8080:80 -p 8081:443 dpage/pgadmin4
    ```
2. To connect to your postgres container, click on `Create A Server`. On the `Connection` tab, enter the following details: 
    ```
    Hostname/address: localhost
    Port: 5432
    Maintenance database: postgres
    Username: root
    Password: root
    ```
    You may also need to go into the `General` and give a name to the server. You can call it `translations` (but it can be called anything).


3. To create the database, right-click on your new `translations` database. then click on `Query Tool`. Enter and run the contents of `creation_and_cleanup.sql`.

## Use node 16
The script requires node.js version 16. The following commands use `nvm` to install and use node 16.
  ```bash
  nvm install 16
  nvm use 16
  ```

## Make a change to translate
The last thing you will need to do is make a unique change to the file `src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx`. The reason being that the PR URL the script uses is for a PR which changes only that file. In order to get the vendor to recognize content for translation, we have to submit unique content everytime. The simplest thing to do here is just delete the existing body of the document, and replace it with something like "bird" or a short phrase, etc.


# Run the script

Run script from docs-website (repo root)

Once all the setup is complete, assuming there are no issues that crop up, you should be able to run the script like so:
```sh
./scripts/actions/translation_workflow/testing/script.sh
```

This will execute the script, and only one other step remains. Halfway through the script, after the file has been uploaded for translation, you will need to authorize the job with the vendor. To do that, navigate to the job in the vendor UI, and click `authorize` once you do that, the script should continue and run to completion.

A complete execution will produce an output similar to:
```sh
\$ node scripts/actions/add-files-to-translation-queue.js https://api.github.com/repos/newrelic/docs-website/pulls/3271/files
Done in 4.52s.
Successful response
Records to be sent: {"ja-JP":[{"id":3,"slug":"src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx","status":"PENDING","locale":"ja-JP","date_created":"2021-07-29T15:10:14.534Z","date_modified":"2021-07-29T15:10:14.534Z"}]}
Successful response
Successful response
[*] Successfully uploaded src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx.
[*] Successfully uploaded src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx context.
Successful response
Successful response
AWAITING_AUTHORIZATION
Job -- f5lauqavyoq9 -- is still translating. Current wait: 0
Successful response
AWAITING_AUTHORIZATION
Job -- f5lauqavyoq9 -- is still translating. Current wait: 5000
Successful response
AWAITING_AUTHORIZATION
Job -- f5lauqavyoq9 -- is still translating. Current wait: 10000
Successful response
AWAITING_AUTHORIZATION
Job -- f5lauqavyoq9 -- is still translating. Current wait: 15000
Successful response
IN_PROGRESS
Job -- f5lauqavyoq9 -- is still translating. Current wait: 20000
Successful response
IN_PROGRESS
Job -- f5lauqavyoq9 -- is still translating. Current wait: 25000
Successful response
COMPLETED
Translation Complete
Successful response
[*]Getting status for batch: glzideuwre8p
Successful response
Successful response
[*]Jobs completed: [{"id":3,"job_uid":"f5lauqavyoq9","batch_uid":"glzideuwre8p","status":"COMPLETED","locale":"ja-JP","date_created":"2021-07-29T15:10:22.543Z","date_modified":"2021-07-29T15:11:22.979Z"}][*]1 batches ready to be deserialized
[*]batchUids: glzideuwre8p
::set-output name=batchesToDeserialize::1
[*] Deserializing /accounts/accounts/account-maintenance/change-passwords-user-preferences
[*]Content deserialized
::set-output name=batchUids::,
::set-output name=completedBatches::[object Object]
::set-output name=deserializedFileUris::src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx
Successful response
Successful response
[*] Successfully deleted https://docs.newrelic.com/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences context.
[*] Successfully deleted src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx context.
[*] Successfully deleted https://docs.newrelic.com/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences context.
[*] Successfully deleted src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx context.
[*] Successfully deleted src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx context.
[!] Unable to delete src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx context.
[!] Unable to delete https://docs.newrelic.com/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences context.
[!] Unable to delete https://docs.newrelic.com/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences context.
[!] Unable to delete src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx context.
[*] Successfully deleted src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx context.
[!] Unable to delete https://docs.newrelic.com/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences context.
[*] Successfully deleted src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx context.
[*] Successfully deleted https://docs.newrelic.com/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences context.
[*] Successfully deleted src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx context.
[*] Successfully deleted https://docs.newrelic.com/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences context.
[!] Unable to delete src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx context.
[!] Unable to delete src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx context.
[!] Unable to delete src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx context.
[!] Unable to delete https://docs.newrelic.com/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences context.
[!] Unable to delete src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx context.
[!] Unable to delete all contexts
Deleting jobs: [{"id":3,"job_uid":"f5lauqavyoq9","batch_uid":"glzideuwre8p","status":"COMPLETED","locale":"ja-JP","date_created":"2021-07-29T15:10:22.543Z","date_modified":"2021-07-29T15:11:22.979Z"}]
Deleting translations: [{"id":3,"slug":"src/content/docs/accounts/accounts/account-maintenance/change-passwords-user-preferences.mdx","status":"COMPLETED","locale":"ja-JP","date_created":"2021-07-29T15:10:14.534Z","date_modified":"2021-07-29T15:11:23.643Z"}]
Deleted translations & jobs
```
