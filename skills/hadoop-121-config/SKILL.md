---

name: hadoop-121-config
description: Use this skill when configuring Apache Hadoop 1.2.1 files for a two-VM instructional cluster. This skill prepares Hadoop configuration files only. It must not start Hadoop daemons, stop Hadoop daemons, or format the NameNode unless the user explicitly requests that after cloning and verification are complete.
---

# Hadoop 1.2.1 Two-VM Configuration Skill

This skill is for configuring **Apache Hadoop 1.2.1** in a two-VM instructional cluster.

The expected workflow is:

1. After installing Hadoop on the first VM, automatically configure Hadoop files using this skill.
2. Manually clone the first VM to create the second VM.
3. Manually update /etc/hosts files on both VMs to list the private IP addresses.
4. Verify hostname resolution and passwordless SSH.
5. Manually format and start Hadoop after cloning is complete.

This skill must only prepare Hadoop configuration files. It must not start the Hadoop cluster automatically.

---

## Strict boundaries

Do not run the following commands unless the user explicitly asks after confirming that cloning, hostnames, networking, and SSH are complete:

```bash
hadoop namenode -format
start-dfs.sh
start-mapred.sh
stop-dfs.sh
stop-mapred.sh
start-all.sh
stop-all.sh
```

Do not assume the cluster is ready to start while only the first VM exists.

Do not format the NameNode during pre-clone configuration.

Do not start HDFS or MapReduce daemons during pre-clone configuration.

---

## Hadoop version policy

This skill is only for **Hadoop 1.2.1**.

Use Hadoop 1.x properties.

Correct Hadoop 1.2.1 properties for this course setup include:

```text
hadoop.tmp.dir
fs.default.name
dfs.replication
mapred.job.tracker
```

Avoid Hadoop 2.x and Hadoop 3.x YARN properties.

Do not use:

```text
fs.defaultFS
yarn-site.xml
yarn.resourcemanager.address
mapreduce.framework.name
mapreduce.jobtracker.address
dfs.namenode.name.dir
dfs.datanode.data.dir
dfs.name.dir
dfs.data.dir
```

When explaining configuration choices, remind the user that Hadoop 1.2.1 uses the older MapReduce v1 architecture, not YARN.

---

## Hadoop directory policy

Use `HADOOP_PREFIX`, not `HADOOP_HOME`.

The Hadoop installation directory for this course setup is:

```bash
export HADOOP_PREFIX=/usr/local/hadoop-1.2.1
```

The Hadoop configuration directory is:

```text
/usr/local/hadoop-1.2.1/conf
```

or equivalently:

```text
$HADOOP_PREFIX/conf
```

When referring to the Hadoop installation directory, use `HADOOP_PREFIX`.

Do not use `HADOOP_HOME` in generated instructions unless explaining that this course setup uses `HADOOP_PREFIX` instead.

---

## Expected cluster role assignment

The cluster has two VMs.

The first VM has a dual role:

```text
groupN-1 = NameNode + JobTracker + DataNode + TaskTracker
groupN-2 = DataNode + TaskTracker
```

The first VM should run both master services and worker services.

The second VM should run worker services.

The cluster does not require a SecondaryNameNode.

Therefore:

```text
masters file = blank
slaves file  = both hosts
```

Example:

```text
group1-1 = NameNode + JobTracker + DataNode + TaskTracker
group1-2 = DataNode + TaskTracker
```

---

## Hostname inference policy

Before editing Hadoop configuration files, detect the current VM hostname.

Use:

```bash
hostname
```

The current VM is expected to be the first VM and should follow this pattern:

```text
groupN-1
```

Examples:

```text
group1-1
group2-1
group10-1
```

When the current hostname matches the `groupN-1` pattern:

1. Treat the current VM as the first VM.
2. Use the current hostname as the NameNode host.
3. Use the current hostname as the JobTracker host.
4. Infer the second VM hostname by replacing the final `-1` with `-2`.

Example:

```text
Detected current hostname: group1-1
Inferred second VM hostname: group1-2
```

Then configure Hadoop using:

```text
NameNode host: group1-1
JobTracker host: group1-1
Worker hosts: group1-1, group1-2
```

If the hostname does not match the `groupN-1` pattern, do not guess the second hostname. Warn the user and ask them to fix the hostname or explicitly provide both hostnames.

Examples of hostnames that should trigger a warning:

```text
ubuntu
localhost
hadoop-vm
master
group1
group1-2
```

---

## Required student warning

Before applying configuration changes, display a warning like this:

```text
Please verify that this VM hostname is correct and follows the required groupN-1 pattern.

This skill will configure the current VM as the NameNode and JobTracker.
It will also configure both groupN-1 and groupN-2 as workers.

After cloning, the second VM must be renamed to groupN-2.
Both hostnames must resolve correctly through /etc/hosts or DNS before Hadoop daemons are started.

This skill will only configure Hadoop files. It will not format the NameNode or start Hadoop daemons.
```

---

## Files to configure

Configure the following Hadoop 1.2.1 files:

```text
/usr/local/hadoop-1.2.1/conf/core-site.xml
/usr/local/hadoop-1.2.1/conf/hdfs-site.xml
/usr/local/hadoop-1.2.1/conf/mapred-site.xml
/usr/local/hadoop-1.2.1/conf/masters
/usr/local/hadoop-1.2.1/conf/slaves
/usr/local/hadoop-1.2.1/conf/hadoop-env.sh
```

Do not create or configure:

```text
yarn-site.xml
```

---

## Backup policy

Before editing files, back up existing files.

Use `HADOOP_PREFIX`:

```bash
export HADOOP_PREFIX=/usr/local/hadoop-1.2.1
```

Suggested backup commands:

```bash
cp $HADOOP_PREFIX/conf/core-site.xml $HADOOP_PREFIX/conf/core-site.xml.bak
cp $HADOOP_PREFIX/conf/hdfs-site.xml $HADOOP_PREFIX/conf/hdfs-site.xml.bak
cp $HADOOP_PREFIX/conf/mapred-site.xml $HADOOP_PREFIX/conf/mapred-site.xml.bak
cp $HADOOP_PREFIX/conf/masters $HADOOP_PREFIX/conf/masters.bak
cp $HADOOP_PREFIX/conf/slaves $HADOOP_PREFIX/conf/slaves.bak
cp $HADOOP_PREFIX/conf/hadoop-env.sh $HADOOP_PREFIX/conf/hadoop-env.sh.bak
```

Only back up files that exist.

---

## core-site.xml policy

Configure `core-site.xml` using the current VM hostname as the NameNode host.

Use:

```text
/app/hadoop/tmp
```

as the Hadoop temporary directory.

Use port:

```text
54310
```

for `fs.default.name`.

For example, if the current hostname is:

```text
group1-1
```

then `/usr/local/hadoop-1.2.1/conf/core-site.xml` should contain:

```xml
<configuration>
<property>
  <name>hadoop.tmp.dir</name>
  <value>/app/hadoop/tmp</value>
  <description>A base for other temporary directories.</description>
</property>

<property>
  <name>fs.default.name</name>
  <value>hdfs://group1-1:54310</value>
  <description>The name of the default file system. Provide the hostname or ip address of
  your master node. The port number must be 54310 or 8020. </description>
</property>
</configuration>
```

Use the detected `groupN-1` hostname instead of hard-coding `group1-1`.

Do not use `fs.defaultFS` for Hadoop 1.2.1.

---

## hdfs-site.xml policy

Configure `hdfs-site.xml` with only the `dfs.replication` property.

Use replication factor:

```text
2
```

because both VMs are configured as workers.

The file `/usr/local/hadoop-1.2.1/conf/hdfs-site.xml` should contain:

```xml
<configuration>
<property>
  <name>dfs.replication</name>
  <value>2</value>
  <description>Default block replication.</description>
</property>
</configuration>
```

Do not include:

```text
dfs.name.dir
dfs.data.dir
```

Do not use Hadoop 2.x properties such as:

```text
dfs.namenode.name.dir
dfs.datanode.data.dir
```

---

## mapred-site.xml policy

Configure `mapred-site.xml` using the current VM hostname as the JobTracker host.

Use port:

```text
54311
```

for `mapred.job.tracker`.

For example, if the current hostname is:

```text
group1-1
```

then `/usr/local/hadoop-1.2.1/conf/mapred-site.xml` should contain:

```xml
<configuration>
<property>
  <name>mapred.job.tracker</name>
  <value>group1-1:54311</value>
  <description>The host and port that the MapReduce job tracker runs
  at. Provide the hostname or ip address of your master node. The port number must be 54311 or 8021.
  </description>
</property>
</configuration>
```

Use the detected `groupN-1` hostname instead of hard-coding `group1-1`.

Do not use YARN properties.

Do not use:

```text
mapreduce.framework.name
yarn.resourcemanager.address
```

---

## masters file policy

The `masters` file controls where Hadoop starts the SecondaryNameNode.

For this instructional cluster, the SecondaryNameNode is not needed.

Therefore, leave `/usr/local/hadoop-1.2.1/conf/masters` blank.

The file may exist, but it should contain no hostnames.

Do not put the master hostname in `masters` unless the user explicitly says they want to run a SecondaryNameNode.

---

## slaves file policy

The `slaves` file should include both VMs because both VMs should run DataNode and TaskTracker daemons.

If the detected hostname is:

```text
group1-1
```

then `/usr/local/hadoop-1.2.1/conf/slaves` should contain:

```text
group1-1
group1-2
```

General rule:

```text
groupN-1
groupN-2
```

The first VM has a dual role and should be included in `slaves`.

---

## hadoop-env.sh policy

Automatically configure `JAVA_HOME` in the Hadoop 1.2.1 environment file.

The file to edit is:

```text
/usr/local/hadoop-1.2.1/conf/hadoop-env.sh
```

Set the Java JDK directory to:

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

Before editing, back up the file if it exists:

```bash
cp /usr/local/hadoop-1.2.1/conf/hadoop-env.sh /usr/local/hadoop-1.2.1/conf/hadoop-env.sh.bak
```

When editing `hadoop-env.sh`:

1. If an existing active `export JAVA_HOME=...` line is present, replace it with:

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

2. If a commented line such as `# export JAVA_HOME=...` is present, add the required active `export JAVA_HOME=...` line below it.

3. If no `JAVA_HOME` line exists, append this line near the Java environment configuration section or at the end of the file:

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

4. Do not use a different Java path unless the user explicitly asks.

5. After editing, verify the setting using:

```bash
grep JAVA_HOME /usr/local/hadoop-1.2.1/conf/hadoop-env.sh
```

Expected active setting:

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

---

## /etc/hosts guidance

This skill may inspect `/etc/hosts`, but should be careful about editing it.

The final cluster requires both hostnames to resolve correctly on both VMs.

Example:

```text
192.168.56.101 group1-1
192.168.56.102 group1-2
```

However, before cloning, the second VM may not exist yet.

Therefore:

1. Do not assume the final IP address of the second VM.
2. Do not hard-code guessed IP addresses.
3. Provide a warning that `/etc/hosts` must be verified after cloning.
4. Only edit `/etc/hosts` if the user explicitly provides the correct IP addresses or asks for that change.

Prefer hostname-based Hadoop configuration over IP-based Hadoop configuration.

---

## Validation after file edits

After editing configuration files, validate only the configuration files.

Safe validation steps include:

```bash
export HADOOP_PREFIX=/usr/local/hadoop-1.2.1

cat $HADOOP_PREFIX/conf/core-site.xml
cat $HADOOP_PREFIX/conf/hdfs-site.xml
cat $HADOOP_PREFIX/conf/mapred-site.xml
cat $HADOOP_PREFIX/conf/masters
cat $HADOOP_PREFIX/conf/slaves
grep JAVA_HOME $HADOOP_PREFIX/conf/hadoop-env.sh
```

If available, validate XML syntax using:

```bash
xmllint --noout /usr/local/hadoop-1.2.1/conf/core-site.xml
xmllint --noout /usr/local/hadoop-1.2.1/conf/hdfs-site.xml
xmllint --noout /usr/local/hadoop-1.2.1/conf/mapred-site.xml
```

If `xmllint` is not installed, do not install packages unless the user asks. Instead, visually inspect the XML structure.

Do not start daemons as part of validation.

---

## Final message after configuration

After preparing the files, provide a final checklist to the user.

The checklist should include:

```text
1. Clone this VM to create the second VM.
2. Rename the first VM as groupN-1 if it is not already named correctly.
3. Rename the second VM as groupN-2.
4. Verify /etc/hosts or DNS resolution on both VMs.
5. Verify passwordless SSH from groupN-1 to both groupN-1 and groupN-2.
6. The port numbers used by Hadoop processes are blocked by Chameleon Cloud’s strict firewall policy. You need to unblock them by running the following commands on both VMs:

   sudo firewall-cmd --zone=public --add-port=50010/tcp
   sudo firewall-cmd --zone=public --add-port=50030/tcp
   sudo firewall-cmd --zone=public --add-port=50060/tcp
   sudo firewall-cmd --zone=public --add-port=50070/tcp
   sudo firewall-cmd --zone=public --add-port=50075/tcp
   sudo firewall-cmd --zone=public --add-port=50090/tcp
   sudo firewall-cmd --zone=public --add-port=54310/tcp
   sudo firewall-cmd --zone=public --add-port=54311/tcp

7. Only after cloning, hostname verification, SSH verification, and firewall configuration, format the NameNode on groupN-1 if this is a fresh cluster.
8. Start HDFS and MapReduce manually.
9. Use jps to verify NameNode, JobTracker, DataNode, and TaskTracker processes.
```

Include the warning:

```text
Do not format the NameNode before cloning unless your instructor specifically tells you to do so.
```
---

## Manual post-clone commands

These commands are for user reference only. Do not run them automatically.

After cloning and verifying hostnames/networking/SSH, the user may run on the first VM:

```bash
export HADOOP_PREFIX=/usr/local/hadoop-1.2.1
$HADOOP_PREFIX/bin/hadoop namenode -format
$HADOOP_PREFIX/bin/start-dfs.sh
$HADOOP_PREFIX/bin/start-mapred.sh
```

Then verify with:

```bash
jps
```

Expected processes on `groupN-1`:

```text
NameNode
JobTracker
DataNode
TaskTracker
```

Expected processes on `groupN-2`:

```text
DataNode
TaskTracker
```

Depending on the environment, additional Java-related processes may also appear.

---

## Safety behavior

If the user asks to start Hadoop before cloning is complete, warn them that the cluster may fail because the second VM does not exist yet.

If the user asks to format the NameNode before cloning, warn them that this can create confusing or duplicated HDFS state after cloning.

If the user explicitly confirms that both VMs have been cloned, hostnames are correct, `/etc/hosts` or DNS works, and passwordless SSH works, then it is acceptable to provide manual startup commands.

Still do not run startup or formatting commands unless the user explicitly requests command execution.

---

## Summary of intended generated files

For a VM whose hostname is `group1-1`, the intended files are:

`/usr/local/hadoop-1.2.1/conf/core-site.xml`

```xml
<configuration>
<property>
  <name>hadoop.tmp.dir</name>
  <value>/app/hadoop/tmp</value>
  <description>A base for other temporary directories.</description>
</property>

<property>
  <name>fs.default.name</name>
  <value>hdfs://group1-1:54310</value>
  <description>The name of the default file system. Provide the hostname or ip address of
  your master node. The port number must be 54310 or 8020. </description>
</property>
</configuration>
```

`/usr/local/hadoop-1.2.1/conf/hdfs-site.xml`

```xml
<configuration>
<property>
  <name>dfs.replication</name>
  <value>2</value>
  <description>Default block replication.</description>
</property>
</configuration>
```

`/usr/local/hadoop-1.2.1/conf/mapred-site.xml`

```xml
<configuration>
<property>
  <name>mapred.job.tracker</name>
  <value>group1-1:54311</value>
  <description>The host and port that the MapReduce job tracker runs
  at. Provide the hostname or ip address of your master node. The port number must be 54311 or 8021.
  </description>
</property>
</configuration>
```

`/usr/local/hadoop-1.2.1/conf/masters`

```text
```

`/usr/local/hadoop-1.2.1/conf/slaves`

```text
group1-1
group1-2
```

`/usr/local/hadoop-1.2.1/conf/hadoop-env.sh`

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

Use the detected `groupN-1` hostname and inferred `groupN-2` hostname instead of hard-coding `group1-1` and `group1-2`.

## Required SSH Key Setup Before VM Cloning

After configuring the Hadoop files, the assistant must verify that the current user's SSH public key is present in `~/.ssh/authorized_keys`.

This step is required before the VM is cloned. The cloned VM must already contain the public key in `authorized_keys` so that passwordless SSH can work between the two VMs after `/etc/hosts` is updated.

The assistant must use the following safe shell logic:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh

if [ ! -f ~/.ssh/id_rsa.pub ]; then
    ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa
fi

touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

if grep -qxF "$(cat ~/.ssh/id_rsa.pub)" ~/.ssh/authorized_keys; then
    echo "SSH public key is already present in ~/.ssh/authorized_keys."
else
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    echo "SSH public key has been appended to ~/.ssh/authorized_keys."
fi
```

The assistant must not duplicate the key if it is already present.

