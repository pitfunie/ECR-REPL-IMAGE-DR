# Delete images created in repositories
aws ecr batch-delete-image \
  --repository-name ecr-repository \
  --image-ids imageTag=webapp imageTag=webapp2 \
  --region us-east-1
aws ecr batch-delete-image \
  --repository-name ecr-repository \
  --image-ids imageTag=webapp imageTag=webapp2 \
  --region us-west-1


# Delete created repositories
aws ecr delete-repository --repository-name ecr-repository --region us-east-1
aws ecr delete-repository --repository-name ecr-repository --region us-west-1
