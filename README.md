# Prerequisites:
1. Make sure [Docker](https://docs.docker.com/get-docker/) is installed
2. Make sure [Minikube](https://minikube.sigs.k8s.io/docs/start/) is installed

# Installation of Kubevirt
## For minikube
```
minikube addons enable kubevirt
```
## For other clusters:
```
export VERSION=$(curl -s https://api.github.com/repos/kubevirt/kubevirt/releases | grep tag_name | grep -v -- '-rc' | sort -r | head -1 | awk -F': ' '{print $2}' | sed 's/,//' | xargs)

echo $VERSION

kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml
```
## Installation of Virtctl(For Windows)
1. Make sure [git](https://git-scm.com/download/win) is installed.

2. Download krew.exe from the [Releases page](https://github.com/kubernetes-sigs/krew/releases) to a directory.

3. Launch a command prompt with administrator privileges and navigate to that directory.

4. Run the following command to install krew
 `.\krew install krew`

5. Add the %USERPROFILE%\.krew\bin directory to your PATH environment variable

6. Launch a new command-line window.

7. Run `kubectl krew` to check the installation.

8. Run `kubectl krew install virt` to install virtctl( to be used as kubectl virt )

# Create a VM instance
1. Use any of the manifest file of linux, in this case, we are using cirros.yaml

2. Run `kubectl create -f cirros.yaml` to create vm

3. Run `kubectl get vms` to check status of vm

4. Start the vm by running the command
```
kubectl virt start cirros
```

5. Run `kubectl get vmi` to check status of vm instance

6. Once the phase status in above command changes to Running, run the following command to access vm
```
kubectl virt console cirros
```

7. Disconnect from vm console by typing ctrl+]

8. To shut down vm instance, run 
```
kubectl virt stop cirros
```

9. To delete vm, run 
```
kubectl delete vm cirros
``` 

# Creating a VM for Windows
1. Download [windows iso file](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2012-r2)

2. Install CDI by running the following commands
```
export VERSION=$(curl -s https://api.github.com/repos/kubevirt/containerized-data-importer/releases/latest | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/')


kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-operator.yaml


kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-cr.yaml
```

3. Upload the folder of the windows iso file using docker (Change the FILE_PATH)
```
docker run -it -d -p 8080:80 --name web -v {FILE_PATH}:/usr/local/apache2/htdocs/ httpd:2.4
```

4. localhost:8080 has the uploaded folder, copy the path of iso file to used in step 7

5. Run `kubectl create -f window-pvc.yaml` to create PVC

6. Make these necessary changes to windows.yaml
 - Change the URL to the path of iso file
 - Change localhost to  host.minikube.internal

7. Run `kubectl create -f window.yaml` to create VM
 - It will take some time for the windows file to be uploaded

8. Run `kubectl get vms` to check status of vm

9. Start the vm by running the command
```
kubectl virt start vm-windows-datavolume
```

10. Run `kubectl get vmi` to check status of vm instance

11. Once the phase status in above command changes to Running, run the following command to access vm
```
kubectl virt vnc vm-windows-datavolume -proxy-only
```

12. Above command will show the PORT_NUMBER where we can access windows

13. In any vnc viewer, enter url- 127.0.0.1::{PORT_NUMBER} to access windows.
