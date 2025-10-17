# Deployment

We recommend when deploying Chat LangChain, you use Vercel for the frontend, GCP Cloud Run for the backend API, and GitHub action for the recurring ingestion tasks. This setup provides a simple and effective way to deploy and manage your application.

## Prerequisites

First, fork [chat-langchain](https://github.com/langchain-ai/chat-langchain) to your GitHub account.

## Weaviate (Vector Store)

We'll use Weaviate for our vector store. You can sign up for an account [here](https://console.weaviate.cloud/).

After creating an account click "Create Cluster". Follow the steps to create a new cluster. Once finished wait for the cluster to create, this may take a few minutes.

Once your cluster has been created you should see a few sections on the page. The first is the cluster URL. Save this as your `WEAVIATE_URL` environment variable.

Next, click "API Keys" and save the API key in the environment variable `WEAVIATE_API_KEY`.

The final Weaviate environment variable is "WEAVIATE_INDEX_NAME". This is the name of the index you want to use. You can name it whatever you want, but for this example, we'll use "langchain".

After this your vector store will be setup. We can now move onto the record manager.

## Supabase (Record Manager)

Visit Supabase to create an account [here](https://supabase.com/dashboard).

Once you've created an account, click "New project" on the dashboard page.
Follow the steps, saving the database password after creating it, we'll need this later.

Once your project is setup (this also takes a few minutes), navigate to the "Settings" tab, then select "Database" under "Configuration".

Here, you should see a "Connection string" section. Copy this string, and insert your database password you saved earlier. This is your `RECORD_MANAGER_DB_URL` environment variable.

That's all you need to do for the record manager. The LangChain RecordManager API will handle creating tables for you.

## Vercel (Frontend)

Create a Vercel account for hosting [here](https://vercel.com/signup).

Once you've created your Vercel account, navigate to [your dashboard](https://vercel.com/) and click the button "Add New..." in the top right.
This will open a dropdown. From there select "Project".

On the next screen, search for "chat-langchain" (if you did not modify the repo name when forking). Once shown, click "Import".

Finally, click "Deploy" and your frontend will be deployed!

## GitHub Action (Recurring Ingestion)

Now, in order for your vector store to be updated with new data, you'll need to setup a recurring ingestion task (this will also populate the vector store for the first time).

Go to your forked repository, and navigate to the "Settings" tab.

Select "Environments" from the left-hand menu, and click "New environment". Enter the name "Indexing" and click "Configure environment".

When configuring, click "Add secret" and add the following secrets:

```
OPENAI_API_KEY=
RECORD_MANAGER_DB_URL=
WEAVIATE_API_KEY=
WEAVIATE_INDEX_NAME=langchain
WEAVIATE_URL=
```

These should be the same secrets as were added to Vercel.

Next, navigate to the "Actions" tab and confirm you understand your workflows, and enable them.

Then, click on the "Update index" workflow, and click "Enable workflow". Finally, click on the "Run workflow" dropdown and click "Run workflow".

Once this has finished you can visit your production URL from Vercel, and start using the app!

## Backend API via Cloud Run

First, build the frontend:

```shell
cd frontend
yarn
yarn build
```

Then, to deploy to Google Cloud Run use the following command:

First create a `.env.gcp.yaml` file with the contents from [`.env.gcp.yaml.example`](.env.gcp.yaml.example) and fill in the values. Then run:

```shell
gcloud run deploy chat-langchain --source . --port 8000 --env-vars-file .env.gcp.yaml --allow-unauthenticated --region us-central1 --min-instances 1
```

Finally, go back to Vercel and add an environment variable `NEXT_PUBLIC_API_BASE_URL` to match your Cloud Run URL.