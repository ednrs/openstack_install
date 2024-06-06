# Deploy and use stackable operators
## Install stackable
Link to [stackable documentation](https://docs.stackable.tech/home/stable/getting-started.html)

```
curl -L -o stackablectl https://github.com/stackabletech/stackable-cockpit/releases/download/stackablectl-24.3.2/stackablectl-x86_64-unknown-linux-gnu
chmod +x stackablectl
mv stackablectl /usr/local/bin/
```
## Install a demo
Check available demos with the command `stackablectl demo list` and install ones you prefer.
```
stackablectl demo install nifi-kafka-druid-earthquake-data
stackablectl demo install nifi-kafka-druid-water-level-data
```
Check installed operators and stackelets.
```
stackablectl operator installed
stackablectl stacklet list
```
