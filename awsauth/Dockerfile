FROM lachlanevenson/k8s-helm:v3.3.1

ARG AWS_IAM_AUTHENTICATOR_URL=https://amazon-eks.s3-us-west-2.amazonaws.com/1.11.5/2018-12-06/bin/linux/amd64/aws-iam-authenticator
ADD ${AWS_IAM_AUTHENTICATOR_URL} /usr/local/bin/aws-iam-authenticator

RUN apk add --update ca-certificates gettext
RUN chmod +x /usr/local/bin/aws-iam-authenticator