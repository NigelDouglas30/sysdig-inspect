# Sysdig Inspect Lab

## Part 1 - Install Sysdig with Basic Filters

We can compare Sysdig Inspect and Tetragon via the below Lab environment: <br/>
```https://isovalent.com/labs/tetragon-getting-started```
<br/><br/>
You can automatically install Sysdig Inspect via the below installer:
```
curl -s https://s3.amazonaws.com/download.draios.com/stable/install-sysdig | sudo bash
```
You can run Sysdig with no filters via the below command: <br/>
Similar to running Wireshark without filters, its impossible to read.

```
sysdig
```

Run a capture for ```5 Seconds``` via the below ```timeout``` commands:
```
timeout 5 sysdig -w nigel-capture.scap
```

You can read the content of the ```nigel-capture.scap``` file with the below command:
```
sysdig -r nigel-capture.scap
```

```epoll_pwait``` - event type is generated when a program waits for an I/O event on an epoll file descriptor
```https://linux.die.net/man/2/epoll_pwait```
```
sysdig -r nigel-capture.scap evt.type=epoll_pwait
```

```kube-apiserver``` process validates and configures data for the api objects which include pods, services, and replicationcontrollers in Kubernetes
```
sysdig -r nigel-capture.scap evt.type=epoll_pwait and proc.name=kube-apiserver
```

## Part 2 - Monitoring a Microservice Architecture

Introduce the Storefront web application to Kubernetes:
```
kubectl apply -f https://installer.calicocloud.io/storefront-demo.yaml
```
Check the IP addresses assigned to our workloads:
```
kubectl get pod -n storefront -o wide
```
Run a capture for ```5 Seconds``` to capture the 
```
timeout 5 sysdig -w storefront-capture.scap
```

Better understand what processes are running on the system with either ```ps aux``` or ```top``` commands:
<br/> 

```
ps aux
```

Read the content of the ```storefront-capture.scap``` file with the below command:
```
sysdig -r storefront-capture.scap proc.name=sandbox-agent or proc.name=peira
```

<img width="953" alt="Screenshot 2024-09-03 at 16 07 37" src="https://github.com/user-attachments/assets/55ab4efe-e8cb-4167-96e6-e79bd4e486ac">

## Part 3 - Introduce a Rogue/Malicious workload

```
kubectl apply -f https://installer.calicocloud.io/rogue-demo.yaml -n storefront
```

Capture all the bad traffic:
```
timeout 5 sysdig -w malicious-traffic.scap
```

Can we see all the nmap traffic from that newly-created pod:
```
sysdig -r malicious-traffic.scap proc.name=nmap
```

Use  ```-S``` or  ```--summary``` to print the event summary (i.e. the list of the top events) when the capture ends.

```
sysdig -r malicious-traffic.scap proc.name=nmap --summary
```

<img width="953" alt="Screenshot 2024-09-04 at 14 40 33" src="https://github.com/user-attachments/assets/b6e6e05e-f158-4667-b2a9-1bf82748a87c">



This could still be useless, as its missing some context. Mayve you would like to see the ```markdown format``` otuput:
```
sysdig --list-markdown
```

```
sysdig --list
```

<img width="953" alt="Screenshot 2024-09-04 at 14 10 59" src="https://github.com/user-attachments/assets/1f6b3b27-9dd4-40a8-b25f-f16ade9765a7">



On the flip side, I never really learned how to read packets in ```ASCII format```:
```
sysdig -r storefront-capture.scap proc.name=sandbox-agent or proc.name=nmap --print-hex-ascii
```

Delete the rogue workload when no longer needed:
```
kubectl delete -f https://installer.calicocloud.io/rogue-demo.yaml -n storefront
```

## Part 4 - Getting the help you need

```
sysdig --help
```

```
sysdig --log-level=warning
```

``` https://man7.org/linux/man-pages/man8/sysdig.8.html ```

## Part 5 - Opening and Deleting Files

```
echo "helloworld" > helloworld.txt
```

```
cat helloworld.txt
```

Instead, we need a background process/program that will do this throughout the length of the capture:
```
wget https://raw.githubusercontent.com/nigel-falco/sysdig-inspect/main/file_watcher.sh
```

```
chmod +x file_watcher.sh
```

```
./file_watcher.sh &
```

Print all the open system calls invoked by cat
```
sysdig proc.name=cat and evt.type=open
```

Compare and contrast the info collect from the Modern eBPF probe:
```
sysdig proc.name=cat --modern-bpf
```

<img width="953" alt="Screenshot 2024-09-04 at 14 37 45" src="https://github.com/user-attachments/assets/b74868c5-666f-4cb8-b379-57792ae65dee">


Print the name of the files opened by cat
```
sysdig -p"%evt.arg.name" proc.name=cat and evt.type=open
```
