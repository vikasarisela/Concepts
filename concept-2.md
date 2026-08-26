The error is very clear:

```text
bash: line 1: java: command not found
Exit code=127
```

Your **Jenkins agent/server does not have Java installed**, or Java is not in its `PATH`.

### On the Jenkins agent

SSH into the agent and check:

```bash
java -version
```

If you get `command not found`, install Java 21:

```bash
sudo dnf install java-21-openjdk -y
```

Then verify:

```bash
java -version
```

You should see Java 21.

Then make sure the Jenkins agent user can find Java:

```bash
sudo -u ec2-user java -version
```

If that works, go back to Jenkins and **relaunch/reconnect the agent**.

### What happened

Jenkins successfully did this:

```text
Copying latest remoting.jar
Verified agent jar
Starting agent process
```

But when it tried:

```text
java -jar remoting.jar
```

the agent couldn't find `java`.

So the problem is **on the Jenkins agent machine, not with `remoting.jar` or SSH**.
