---
name: hadoop-121-post-clone-startup
description: Use this skill after the Hadoop 1.2.1 VM has been cloned into a two-VM instructional cluster and /etc/hosts has been updated on both VMs. This skill verifies hostname resolution, firewall ports, passwordless SSH, NameNode formatting status, Hadoop daemon startup, and daemon process status. It may run firewall, SSH test, NameNode format, startup, and verification commands only when the student explicitly asks it to run them, and it must check whether each step was already completed before running potentially destructive or duplicate commands.
Hadoop 1.2.1 Post-Clone Startup and Verification Skill
This skill assists students with the remaining steps after the first Hadoop VM has been cloned and `/etc/hosts` has been updated on both VMs.
This skill is for Apache Hadoop 1.2.1 in a two-VM instructional cluster.
The expected cluster layout is:
```text
groupN-1 = NameNode + JobTracker + DataNode + TaskTracker
groupN-2 = DataNode + TaskTracker
```
The first VM has a dual role and should run both master and worker daemons.
The second VM should run only worker daemons.
This skill assumes the earlier pre-clone configuration skill has already configured Hadoop files using Hadoop 1.x properties such as:
```text
fs.default.name
mapred.job.tracker
dfs.replication
```
Do not use Hadoop 2.x or Hadoop 3.x YARN commands or properties.
---
Scope of this skill
Use this skill only after all of the following are expected to be true:
The first VM has already been cloned to create the second VM.
The first VM hostname is `groupN-1`.
The second VM hostname is `groupN-2`.
`/etc/hosts` has been updated on both VMs.
Hadoop 1.2.1 is installed at `/usr/local/hadoop-1.2.1`.
Hadoop configuration files were already prepared by the pre-clone configuration skill.
This skill can help with:
Opening required Hadoop firewall ports on both VMs.
Checking hostname resolution between both VMs.
Checking passwordless SSH from `groupN-1` to both `groupN-1` and `groupN-2`.
Checking whether the NameNode appears to have already been formatted.
Formatting the NameNode only if needed and only on `groupN-1`.
Starting Hadoop HDFS and MapReduce daemons only if they are not already running.
Verifying daemon processes on both VMs using `jps`.
---
Execution safety policy
The assistant may inspect the system using safe read-only commands.
The assistant must not run commands that modify the system unless the student explicitly asks it to run them.
Read-only checks are allowed when useful:
```bash
hostname
cat /etc/hosts
getent hosts groupN-1
getent hosts groupN-2
ssh -o BatchMode=yes -o ConnectTimeout=5 groupN-1 hostname
ssh -o BatchMode=yes -o ConnectTimeout=5 groupN-2 hostname
jps
sudo firewall-cmd --zone=public --query-port=50010/tcp
sudo firewall-cmd --zone=public --query-port=50030/tcp
sudo firewall-cmd --zone=public --query-port=50060/tcp
sudo firewall-cmd --zone=public --query-port=50070/tcp
sudo firewall-cmd --zone=public --query-port=50075/tcp
sudo firewall-cmd --zone=public --query-port=50090/tcp
sudo firewall-cmd --zone=public --query-port=54310/tcp
sudo firewall-cmd --zone=public --query-port=54311/tcp
```
Potentially modifying commands require explicit student permission:
```bash
sudo firewall-cmd --zone=public --add-port=PORT/tcp
hadoop namenode -format
start-dfs.sh
start-mapred.sh
```
The assistant must never format the NameNode automatically just because formatting is the next step.
The assistant must first check whether formatting appears to have already happened.
The assistant must never start Hadoop daemons before verifying that hostname resolution and passwordless SSH work.
The assistant must never repeatedly start daemons that are already running.
---
Hadoop directory policy
Use `HADOOP\_PREFIX`, not `HADOOP\_HOME`.
The Hadoop installation directory is:
```bash
export HADOOP\_PREFIX=/usr/local/hadoop-1.2.1
```
The Hadoop executable directory is:
```bash
$HADOOP\_PREFIX/bin
```
When generating commands, use:
```bash
export HADOOP\_PREFIX=/usr/local/hadoop-1.2.1
export PATH=$HADOOP\_PREFIX/bin:$PATH
```
Do not use YARN commands.
Do not use:
```bash
start-yarn.sh
stop-yarn.sh
yarn
mapred --daemon
hdfs --daemon
```
For Hadoop 1.2.1, use:
```bash
$HADOOP\_PREFIX/bin/hadoop namenode -format
$HADOOP\_PREFIX/bin/start-dfs.sh
$HADOOP\_PREFIX/bin/start-mapred.sh
```
---
Hostname detection and role policy
First detect the current hostname:
```bash
hostname
```
The current host should match one of these patterns:
```text
groupN-1
groupN-2
```
If the current host is `groupN-1`, treat it as the master VM.
If the current host is `groupN-2`, treat it as the worker VM.
If the hostname does not match either pattern, warn the student and do not run Hadoop startup or formatting commands.
Infer the peer host as follows:
```text
current host groupN-1 -> peer host groupN-2
current host groupN-2 -> peer host groupN-1
```
Formatting and cluster startup must be run only from `groupN-1`.
If the student is on `groupN-2` and asks to format or start Hadoop, explain that these commands should be run from `groupN-1`.
---
Required pre-start checks
Before formatting or starting Hadoop, verify all of the following.
1. Verify hostname resolution
Use `getent hosts`:
```bash
getent hosts groupN-1
getent hosts groupN-2
```
Replace `groupN` with the detected group number.
If either command fails, tell the student to fix `/etc/hosts` on both VMs before proceeding.
Do not start Hadoop if either hostname does not resolve.
2. Verify passwordless SSH
From `groupN-1`, verify SSH to both hosts:
```bash
ssh -o BatchMode=yes -o ConnectTimeout=5 groupN-1 hostname
ssh -o BatchMode=yes -o ConnectTimeout=5 groupN-2 hostname
```
Expected output:
```text
groupN-1
groupN-2
```
If either SSH command fails, do not format or start Hadoop.
Tell the student that passwordless SSH must work from `groupN-1` to both VMs.
Do not ask the student to type a password into SSH for Hadoop startup verification.
3. Verify Hadoop configuration files exist
Check:
```bash
export HADOOP\_PREFIX=/usr/local/hadoop-1.2.1
ls -l $HADOOP\_PREFIX/conf/core-site.xml
ls -l $HADOOP\_PREFIX/conf/hdfs-site.xml
ls -l $HADOOP\_PREFIX/conf/mapred-site.xml
ls -l $HADOOP\_PREFIX/conf/slaves
ls -l $HADOOP\_PREFIX/conf/masters
```
Check that `slaves` contains both hosts:
```bash
cat $HADOOP\_PREFIX/conf/slaves
```
Expected:
```text
groupN-1
groupN-2
```
If the files are missing or inconsistent, tell the student to rerun or review the pre-clone Hadoop configuration skill before startup.
---
Firewall policy
The port numbers used by Hadoop processes are blocked by Chameleon Cloud’s strict firewall policy.
The following ports must be open on both VMs:
```text
50010/tcp
50030/tcp
50060/tcp
50070/tcp
50075/tcp
50090/tcp
54310/tcp
54311/tcp
```
Before adding a port, always check whether it is already open.
Use this safe idempotent firewall logic on each VM:
```bash
for port in 50010 50030 50060 50070 50075 50090 54310 54311; do
    if sudo firewall-cmd --zone=public --query-port=${port}/tcp >/dev/null 2>\&1; then
        echo "Port ${port}/tcp is already open."
    else
        sudo firewall-cmd --zone=public --add-port=${port}/tcp
        echo "Opened port ${port}/tcp."
    fi
done
```
This uses runtime firewall rules. Do not add `--permanent` unless the student or instructor explicitly asks for permanent firewall rules.
If the student asks to open the firewall on both VMs from `groupN-1`, first verify passwordless SSH to `groupN-2`, then use:
```bash
for host in groupN-1 groupN-2; do
    echo "Checking firewall ports on ${host}"
    ssh -o BatchMode=yes -o ConnectTimeout=5 ${host} '
        for port in 50010 50030 50060 50070 50075 50090 54310 54311; do
            if sudo firewall-cmd --zone=public --query-port=${port}/tcp >/dev/null 2>\&1; then
                echo "Port ${port}/tcp is already open."
            else
                sudo firewall-cmd --zone=public --add-port=${port}/tcp
                echo "Opened port ${port}/tcp."
            fi
        done
    '
done
```
Replace `groupN-1` and `groupN-2` with the detected hostnames.
If `sudo` requires a password or the SSH remote command fails, report the failure and provide the local command for the student to run manually on each VM.
---
NameNode formatting policy
Formatting the NameNode is destructive for a fresh HDFS metadata directory and should only happen once during initial cluster setup.
Format only on `groupN-1`.
Before formatting, check whether the NameNode appears to have already been formatted.
Use:
```bash
export HADOOP\_PREFIX=/usr/local/hadoop-1.2.1
NAMENODE\_DIR="/app/hadoop/tmp/dfs/name"

if \[ -d "$NAMENODE\_DIR/current" ]; then
    echo "NameNode metadata already exists at $NAMENODE\_DIR/current. Do not format again unless your instructor explicitly tells you to reset HDFS."
else
    echo "NameNode metadata directory was not found. This appears to be an unformatted fresh cluster."
fi
```
If the metadata directory exists, do not format.
If the metadata directory does not exist and all pre-start checks pass, the assistant may run the following only when the student explicitly asks it to format:
```bash
export HADOOP\_PREFIX=/usr/local/hadoop-1.2.1
$HADOOP\_PREFIX/bin/hadoop namenode -format
```
After formatting, verify that NameNode metadata now exists:
```bash
test -d /app/hadoop/tmp/dfs/name/current \&\& echo "NameNode format appears complete."
```
If the format command reports an error, stop and explain the error. Do not proceed to daemon startup.
---
Daemon startup policy
Start daemons only from `groupN-1`.
Before starting, check current daemons with:
```bash
jps
```
Expected daemons on `groupN-1` after successful startup:
```text
NameNode
JobTracker
DataNode
TaskTracker
```
Expected daemons on `groupN-2` after successful startup:
```text
DataNode
TaskTracker
```
If all expected daemons are already running, do not run startup scripts again.
If only some daemons are running, report which ones are missing. Then start only the relevant subsystem if appropriate.
For HDFS:
```bash
$HADOOP\_PREFIX/bin/start-dfs.sh
```
For MapReduce v1:
```bash
$HADOOP\_PREFIX/bin/start-mapred.sh
```
Do not use `start-all.sh` unless the student explicitly asks for it. Prefer `start-dfs.sh` and `start-mapred.sh` because they are clearer for teaching.
Do not run `start-dfs.sh` if both `NameNode` and all expected `DataNode` processes are already running.
Do not run `start-mapred.sh` if `JobTracker` and all expected `TaskTracker` processes are already running.
---
Safe daemon verification logic
From `groupN-1`, use this verification script:
```bash
for host in groupN-1 groupN-2; do
    echo "===== ${host} ====="
    ssh -o BatchMode=yes -o ConnectTimeout=5 ${host} jps
    echo
 done
```
Expected output on `groupN-1` should include:
```text
NameNode
JobTracker
DataNode
TaskTracker
```
Expected output on `groupN-2` should include:
```text
DataNode
TaskTracker
```
Additional Java-related processes may also appear and are not necessarily errors.
If a required process is missing, explain which one is missing and suggest the likely area to check:
```text
Missing NameNode: check NameNode formatting and core-site.xml.
Missing DataNode: check slaves file, SSH, firewall, and HDFS logs.
Missing JobTracker: check mapred-site.xml and start-mapred.sh output.
Missing TaskTracker: check slaves file, SSH, firewall, and MapReduce logs.
```
---
Recommended full post-clone workflow
When a student asks what to do next after cloning, guide them through this sequence:
Confirm they are on `groupN-1`.
Verify `/etc/hosts` resolution for `groupN-1` and `groupN-2`.
Verify passwordless SSH from `groupN-1` to both VMs.
Check firewall ports on both VMs.
Open missing firewall ports only if the student asks the assistant to run the commands.
Check whether the NameNode has already been formatted.
Format the NameNode only if it has not been formatted and the student explicitly asks.
Check current daemon status using `jps`.
Start missing Hadoop daemons only if the student explicitly asks.
Verify final daemon status on both VMs.
---
One-shot guided command block for students
If the student asks for commands to run manually, provide this block from `groupN-1` after replacing `groupN` with the correct group number:
```bash
export HADOOP\_PREFIX=/usr/local/hadoop-1.2.1
export PATH=$HADOOP\_PREFIX/bin:$PATH

# 1. Verify hostname resolution
getent hosts groupN-1
getent hosts groupN-2

# 2. Verify passwordless SSH
ssh -o BatchMode=yes -o ConnectTimeout=5 groupN-1 hostname
ssh -o BatchMode=yes -o ConnectTimeout=5 groupN-2 hostname

# 3. Check/open firewall ports on both VMs
for host in groupN-1 groupN-2; do
    echo "Checking firewall ports on ${host}"
    ssh -o BatchMode=yes -o ConnectTimeout=5 ${host} '
        for port in 50010 50030 50060 50070 50075 50090 54310 54311; do
            if sudo firewall-cmd --zone=public --query-port=${port}/tcp >/dev/null 2>\&1; then
                echo "Port ${port}/tcp is already open."
            else
                sudo firewall-cmd --zone=public --add-port=${port}/tcp
                echo "Opened port ${port}/tcp."
            fi
        done
    '
done

# 4. Check whether NameNode appears already formatted
if \[ -d /app/hadoop/tmp/dfs/name/current ]; then
    echo "NameNode metadata already exists. Skipping format."
else
    echo "NameNode metadata not found. Format is needed for a fresh cluster."
    echo "Run this only once if your instructor says this is a fresh cluster:"
    echo "$HADOOP\_PREFIX/bin/hadoop namenode -format"
fi

# 5. Check local daemons
jps
```
Do not include the actual format command as an automatically executed line in a one-shot command block unless the student explicitly asks to execute formatting.
---
One-shot execution policy if the student asks the assistant to run everything
If the student says something like “run the remaining steps for me,” the assistant must still perform checks and avoid duplicate or unsafe actions.
Use this decision order:
Detect hostname.
Stop if not on `groupN-1`.
Verify `groupN-1` and `groupN-2` resolve.
Verify passwordless SSH to both VMs.
Check and open missing firewall ports on both VMs.
Check NameNode formatting status.
If NameNode metadata exists, skip formatting.
If NameNode metadata does not exist, ask for explicit confirmation before formatting unless the student’s latest message explicitly included permission to format.
Check daemon status on both VMs.
Start HDFS only if HDFS daemons are missing and NameNode has been formatted.
Start MapReduce only if MapReduce daemons are missing.
Verify final daemon status on both VMs.
If the latest student message explicitly says to format if needed, then formatting may proceed after the safety checks pass.
---
Command execution script for assistant use when explicitly authorized
When the student explicitly asks the assistant to run the post-clone setup, use a cautious script like this from `groupN-1`.
Before running it, replace `groupN-1` and `groupN-2` with the detected hostnames.
```bash
#!/usr/bin/env bash
set -u

export HADOOP\_PREFIX=/usr/local/hadoop-1.2.1
export PATH="$HADOOP\_PREFIX/bin:$PATH"

MASTER="groupN-1"
WORKER="groupN-2"
HOSTS="$MASTER $WORKER"
PORTS="50010 50030 50060 50070 50075 50090 54310 54311"

current\_host="$(hostname)"
if \[ "$current\_host" != "$MASTER" ]; then
    echo "ERROR: This startup script must be run on $MASTER, but current hostname is $current\_host."
    exit 1
fi

for host in $HOSTS; do
    if getent hosts "$host" >/dev/null 2>\&1; then
        echo "Hostname resolves: $host"
    else
        echo "ERROR: Hostname does not resolve: $host"
        exit 1
    fi
done

for host in $HOSTS; do
    if ssh -o BatchMode=yes -o ConnectTimeout=5 "$host" hostname >/tmp/ssh-check-${host}.out 2>/tmp/ssh-check-${host}.err; then
        echo "Passwordless SSH works for $host: $(cat /tmp/ssh-check-${host}.out)"
    else
        echo "ERROR: Passwordless SSH failed for $host."
        cat /tmp/ssh-check-${host}.err
        exit 1
    fi
done

for host in $HOSTS; do
    echo "===== Checking firewall on $host ====="
    ssh -o BatchMode=yes -o ConnectTimeout=5 "$host" "
        for port in $PORTS; do
            if sudo firewall-cmd --zone=public --query-port=\\${port}/tcp >/dev/null 2>\&1; then
                echo \\"Port \\${port}/tcp is already open.\\"
            else
                sudo firewall-cmd --zone=public --add-port=\\${port}/tcp
                echo \\"Opened port \\${port}/tcp.\\"
            fi
        done
    "
done

if \[ -d /app/hadoop/tmp/dfs/name/current ]; then
    echo "NameNode metadata already exists. Skipping format."
else
    echo "NameNode metadata not found. Formatting NameNode because explicit permission was given."
    "$HADOOP\_PREFIX/bin/hadoop" namenode -format
fi

master\_jps="$(jps)"
worker\_jps="$(ssh -o BatchMode=yes -o ConnectTimeout=5 "$WORKER" jps)"

if echo "$master\_jps" | grep -q NameNode \&\& echo "$master\_jps" | grep -q DataNode \&\& echo "$worker\_jps" | grep -q DataNode; then
    echo "HDFS daemons appear to already be running. Skipping start-dfs.sh."
else
    echo "Starting HDFS daemons."
    "$HADOOP\_PREFIX/bin/start-dfs.sh"
fi

sleep 3
master\_jps="$(jps)"
worker\_jps="$(ssh -o BatchMode=yes -o ConnectTimeout=5 "$WORKER" jps)"

if echo "$master\_jps" | grep -q JobTracker \&\& echo "$master\_jps" | grep -q TaskTracker \&\& echo "$worker\_jps" | grep -q TaskTracker; then
    echo "MapReduce daemons appear to already be running. Skipping start-mapred.sh."
else
    echo "Starting MapReduce daemons."
    "$HADOOP\_PREFIX/bin/start-mapred.sh"
fi

sleep 3

echo "===== Final daemon status on $MASTER ====="
jps

echo "===== Final daemon status on $WORKER ====="
ssh -o BatchMode=yes -o ConnectTimeout=5 "$WORKER" jps
```
This script is intentionally idempotent for firewall opening and cautious about Hadoop startup.
---
Final success criteria
The final verification should show the following.
On `groupN-1`:
```text
NameNode
JobTracker
DataNode
TaskTracker
```
On `groupN-2`:
```text
DataNode
TaskTracker
```
If these are present, tell the student that the Hadoop 1.2.1 two-VM cluster appears to be running.
If any daemon is missing, report the missing daemon and suggest checking logs under:
```text
/usr/local/hadoop-1.2.1/logs
```
---
Important warnings to preserve
Always preserve these warnings:
```text
Do not format the NameNode more than once unless your instructor explicitly tells you to reset HDFS.
```
```text
Run formatting and startup commands from groupN-1, not groupN-2.
```
```text
For Hadoop 1.2.1, use MapReduce v1 commands. Do not use YARN commands.
```
