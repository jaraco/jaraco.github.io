Yesterday, MongoDB announced their new backend as a service, MongoDB Stitch. Elliot Horowitz ran through the service with a blog accepting comments.

Today I tried it out.

My goal was to create the "Hello, World" of stitch, to create an HTTP service that accepts a JSON payload and saves that payload to the database. It turned out to be not too difficult.

Here's how you can do the same:

1. You'll need an Atlas cluster. If you don't already have a suitable one, head over to https://cloud.mongodb.com and create your free tier cluster.

2. Still in MongoDB Cloud, create the Stitch App and link it to your cluster. Follow the getting-started wizard to enable authentication and create a db/collection in the cluster (this example uses hellodb/posts). Ignore the rest of the wizard.

3. Next, create a service. Select the HTTP hook. Even though it only mentions sending HTTP requests, it's also used for incoming HTTP. Give the service the name 'intake'.

   ![HTTP hook button](create-http.png "Create HTTP Service")

   Consider reviewing the [docs on the HTTP Service](https://docs.mongodb.com/stitch/services/http/).

4. In the new intake service, navigate to the Incoming Webhooks tab and create a New Webhook.

5. Change the Request Validation to use Require Secret as Query Param. This secret key authentication is there to prevent unauthorized systems from posting to the hook. Leave the secret key set to the default SECRET (though for a production app, definitely set this to a unique value).

6. Next set up two stages in the pipeline, and set the Output of the pipeline to "Single Document".

7. In the first stage of the pipeline, use the built-in service with the following literal action:

   ```
   {
     "items": [
       "%%vars.payload.body"
     ]
   }
   ```

   This action will cause the pipeline stage to emit exactly one item, which is the payload body of
   ``%%vars``.

   Additionally, bind data to ``%%vars`` and use the following to pass the ``%%args`` to ``%%vars``:

   ```
   {
     "payload": "%%args"
   }
   ```

   Save with Done.

8. In the second and final stage of the pipeline, select "mongodb-atlas" as the service and "insert" as the action. To the insert action, specify the database and collection to use (matching the DB and collection used in step 2).

   ```
   {
     "database": "hellodb",
     "collection": "posts"
   }
   ```

   Save with Done and save the Webhook.

9. Finally, test the service using cUrl. Copy the webhook URL from the Stitch Admin Console service and replace it for $WEBHOOK_URL in this command:

   ```
   curl '$WEBHOOK_URL?secret=SECRET' -X POST -d '{"hello": "world"}' -H "Content-Type: application/json"
   ```

   If all goes well, the command should return without error and the document `{"hello": "world"}` will appear in the database.


It worked for me and I'm excited to see the service in action.

Some aspects were slightly unintuitive, such as how to map args to a payload and a payload to the pipeline output, but with a bit of help from MongoDB engineers, it's now running, and they're clearly looking to smooth some of these rough edges during the beta period.

I'm looking forward to the improvements and to expanding my usage of this powerful paradigm.
