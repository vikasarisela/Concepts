Now your Docker command is correct. The new error is a Docker permission issue.

ERROR: permission denied while trying to connect to the docker API
at unix:///var/run/docker.sock

This means the Jenkins user (ec2-user in your log) does not have permission to access Docker.

1. Check Docker socket permissions

On the Jenkins/EC2 server:

ls -l /var/run/docker.sock

You will probably see something like:

srw-rw---- 1 root docker ... /var/run/docker.sock

This means only root and members of the docker group can use Docker.

2. Add ec2-user to Docker group
sudo usermod -aG docker ec2-user

Then restart/reload the session.

For example:

sudo systemctl restart docker

Then log out and log back in to refresh the group membership.

Check:

groups ec2-user

You should see:

ec2-user docker

Then test:

docker ps

If docker ps works without sudo, Jenkins should be able to run the build.

Important Jenkins point

Your log shows:

/home/ec2-user/.docker/config.json

So Jenkins is running the Docker command as ec2-user.

Therefore, adding the same user to the Docker group is important.

Your build command itself is now correct:

docker build -t 193849563622.dkr.ecr.us-east-1.amazonaws.com/roboshop/catalogue:1.0.0 .

So the flow is now:

ECR login       ✅
Docker command  ✅
Docker access   ❌ ← current problem

Once Docker permissions are fixed, the build should proceed to the next stage.


xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
The ECR login is successful. The failure is only because the user running Jenkins cannot access Docker.

Your error:

permission denied while trying to connect to the docker API
unix:///var/run/docker.sock
Fix it on the EC2/Jenkins server

Run:

sudo usermod -aG docker ec2-user

Then verify:

groups ec2-user

You should see docker in the output.

Important: restart Jenkins

Because Jenkins is already running, the new group membership may not be picked up by the Jenkins process.

Run:

sudo systemctl restart jenkins

Then test as ec2-user:

sudo -u ec2-user docker ps

If that works, your Jenkins build should be able to run:

docker build -t 193849563622.dkr.ecr.us-east-1.amazonaws.com/roboshop/catalogue:1.0.0 .
If it still fails

Check:

ls -l /var/run/docker.sock

Typically you want something like:

srw-rw---- 1 root docker ... /var/run/docker.sock

Also check the Jenkins process:

ps -ef | grep jenkins

Don't change your docker build command now — it is correct. The current problem is Docker socket permission, not the ECR command or image tag.