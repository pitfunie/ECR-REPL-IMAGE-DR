# ECR-REPL-IMAGE-DR
#Replication to Another Region of an Amazon ECR Image Record

#replication Amazon Elastic Container Registry in multi-region. Stored container images must be available 
#in a secondary region before the application can be configured during the disaster.

# Create a Repository in the Registry
# Save your AWS account ID to an environment variable.
sudo yum install jq -y
export AWSACCOUNT=$(aws sts get-caller-identity | jq -r '.Account')


# Create a repository in registry private. Note: Every AWS Account already has one registry private.
aws ecr create-repository \
  --repository-name ecr-repository --region us-east-1


# Create an image and push to the created Repository
# Create a test page: index.html
cat > index.html << EOF
    <!doctype html>
    <html lang="en">
        <head>
            <meta charset="utf-8">
            <title>Docker Nginx</title>
        </head>
        <body>
            <h2>Hello from Nginx container</h2>
        </body>
    </html>
EOF


#Create the Dockerfile
cat > Dockerfile << EOF
FROM nginx:latest
COPY ./index.html /usr/share/nginx/html/index.html
EOF


# Execute the docker build in order to create the docker image with the repository name.
docker build -t $AWSACCOUNT.dkr.ecr.us-east-1.amazonaws.com/ecr-repository:webapp .


# To push the image to the repository using docker, you must authenticate to the repository first. To do this, use the get-login command and get the password to authenticate the docker with the docker login command.
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $AWSACCOUNT.dkr.ecr.us-east-1.amazonaws.com

# After authenticating, upload your new image using the docker push command.
docker push $AWSACCOUNT.dkr.ecr.us-east-1.amazonaws.com/ecr-repository:webapp


# List the images in the Images Registry.
aws ecr list-images --repository-name ecr-repository --region us-east-1
Set up replication for another region in the image registry


# Get the record id.
export REGISTRYID=$(aws ecr describe-registry | jq -r '.registryId')


# Create a json file with configuration parameters. Replace the registration id for your environment. Notice that the replication configuration will replicate to the California (us-west-1).
cat > replication.json << EOF
{ 
    "rules": [ 
        { 
            "destinations": [ 
                {
                    "region": "us-west-1", 
                    "registryId": "$REGISTRYID" 
                } 
            ] 
        } 
    ] 
} 
EOF



# Enable replication with the command below.
aws ecr put-replication-configuration \
  --replication-configuration file://replication.json \
  --region us-east-1

# Make an update to the image
# Update the container image by generating a new version with the following command.
docker build -t $AWSACCOUNT.dkr.ecr.us-east-1.amazonaws.com/ecr-repository:webapp2 .

# Push the new version. Notice that the push takes place to the region of origin, in this case N.Virginia (us-east-1).
docker push $AWSACCOUNT.dkr.ecr.us-east-1.amazonaws.com/ecr-repository:webapp2

# Validate that the image is in both regions
# List the image repositories from both regions. On the return of the command, note the parameter RepositoryURI.
aws ecr describe-repositories --region us-east-1
aws ecr describe-repositories --region us-west-1


# List images from both regions. Note that the images are the same size, the same manifest. However, the 
# push date in the repository is slightly different (ImagePushedAt parameter).

aws ecr describe-images --repository-name ecr-repository --region us-east-1
aws ecr describe-images --repository-name ecr-repository --region us-west-1

