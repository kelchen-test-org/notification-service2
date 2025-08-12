# Konflux Notification Service

The Notification Service is a controller that sends push pipelineruns results to 
[AWS SNS service](https://aws.amazon.com/sns/).
It watches for `push pipelineruns`, extracts the results from pipelineruns that ended successfully 
and sends them to a topic defined in `AWS SNS`.

Secrets and environment variables are needed to configure the `AWS SNS`.

## AWS credentials

`AWS Access key id` and `AWS secret access key` are needed to sign requests to AWS.
The keys can be provided as a secret (prefered option) or as environment variables.

### AWS credentials as a secret

The preferred way to supply the credentials is to create a secret containing the content 
of a credentials file.

The credentails file format is:
```
[default]
aws_access_key_id=<AWS_ACCESS_KEY_ID>
aws_secret_access_key=<AWS_SECRET_ACCESS_KEY>
```
reference: [AWS sdk go v2](https://aws.github.io/aws-sdk-go-v2/docs/configuring-sdk/#creating-the-credentials-file)

We will create a secret to be used by the controller:  
key name should be `credentials`.  
Key value should be the content of the credentials file encoded to base 64.

For example, if our credentials file content encoded to base 64 is: `dGVzdA==`, 
the secret will be:
```
kind: Secret
apiVersion: v1
metadata:
  name: aws-sns-secret
  namespace: notification-controller
data:
  credentials: dGVzdA==
type: Opaque
```
To use these supplied credentials we will mount the secret to the pod's `/.aws` directory so that
eventually we will have a file `/.aws/credentails` which will contain the value of the secret.

#### Deployment example

Create a secret containing the AWS credentials:
```
kind: Secret
apiVersion: v1
metadata:
  name: aws-sns-secret
  namespace: notification-controller
data:
  credentials: < Base 64 encoded credentials file >
type: Opaque
```

Create a deployment with the secret mounted:
``` 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: notification-controller
  name: notification-controller
  namespace: notification-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      run: notification-controller
  template:
    metadata:
      labels:
        run: notification-controller
    spec:
      volumes:
      - name: vol-secret
        secret:
          secretName: aws-sns-secret    
      serviceAccountName: notification-controller-sa
      containers:
      - name: notification-controller
        image: < Link to image >
        env:
        - name: NOTIFICATION_TOPIC_ARN
          value: < Topic ARN >
        - name: NOTIFICATION_REGION
          value: < Region >
        volumeMounts:
        - name: vol-secret
          mountPath: /.aws    
```

### AWS credentilas as Environment Variables

Another way to supply the credentials is Environment Variables.
| Name | Description |
| -- | -- |
| AWS_ACCESS_KEY_ID | AWS Key ID
| AWS_SECRET_ACCESS_KEY | AWS secret key

#### Deployment example

``` 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: notification-controller
  name: notification-controller
  namespace: notification-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      run: notification-controller
  template:
    metadata:
      labels:
        run: notification-controller
    spec:
      serviceAccountName: notification-controller-sa
      containers:
      - name: notification-controller
        image: < Link to image >
        env:
        - name: AWS_ACCESS_KEY_ID
          value: < AWS Access Key ID >
        - name: AWS_SECRET_ACCESS_KEY
          value: < AWS Secret Access Key >
        - name: NOTIFICATION_TOPIC_ARN
          value: < Topic ARN >
        - name: NOTIFICATION_REGION
          value: < Region >
```

## Define Topic and Region

These environment variables will be used to define the `SNS topic`

<!-- Dummy comment to trigger MintMaker bot activity -->
