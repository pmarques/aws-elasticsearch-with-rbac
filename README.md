# AWS Managed Elasticsearch with Cognito RBAC

This is compact solution / template to show how you can configure AWS Elasticsearch with cognito and apply some restrictions.

1. Create a CloudFormation Stack with elastisearch.yaml
2. Create users in the AWS Cognito User Pool console and assign on of the groups
3. Access the kibana, you can find the URL in the Outputs of the CloudFormation Stack

## Tests

Using Kibana Dev Tools:

### Admin
Only users in the **Admin** group can create docs:

```
PUT twitter/_doc/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

### Developers

Users in other groups can only read docs in their projects:

* **ProjectAGroup** can readon from **-proj-a** cluster
* **ProjectBGroup** can readon from **-proj-b** cluster

```
GET twitter/_doc/1
```

## Findings

At the time, 6th of January 2020:

* AWS ES service only allows you to control access to ES indexes based on the URL path.

This means that ES HTTP Request with index on the query itself (HTTP data) can bypass the IAM Polices.
There is a way to configure ES only allow requests with explicit index in the URL Path, although this breaks Kibana and possible the apps.

* AWS ES Service doesn't support cross cluster search

* \_bulk ES REST API can be used as a back door for write requests

To block it we can add a Deny statement for the \_bulk requests.

Other APIs would need to be evaluated to check other possible issues