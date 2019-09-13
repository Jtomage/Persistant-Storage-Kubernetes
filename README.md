# Persistant Storage Kubernetes

One of the big challenges with kubernetes is to configure persistant storage for databases. It can be done using these 4 yaml files. 

Hope this helps out anyone else working with kubernetes and getting database containers up and running as this was challenging to figure out. I originally used helm charts version of postgreSQL but it would fail so I stripped everything that I thought was not need to troubleshoot and get it up and running. 

### Environment
- centoOS
- AWS
- NFS Server
  - make sure no_root_squash is set on the NFS server as postgreSQL uses chown to take control of the directory. If its unable to take ownership the pod will fail to deploy. This was the biggest headache to figure out.

### nfs-client-provisioner.yaml
This was generated with a shell script using helm charts. On our instance of kubernetes had problems with which level / directory it was on and had to be modified to accomidate. Please check that your helm template is pointing to the right directory

### PVC-NFS-creation.yaml
If you have an nfs-client-provisioner these do not need to manual generated. However, during the course of troubleshooting and getting persistant storage to work I created the Persistent Volume and Persistant Volume Claim manually to understand how kubernetes works. Also, it helped to have manually created version as nfs-client-provisioner may create new nfs paths if the name of the deployment or stateful set changes.

The layers for persistant storage works is storage class is the lowest layer, followed by Persistant Volume, then Persistant Volume Claim.

My current undestanding is that you can create multiple Persistant Volumes with different paths in case you want to use the same NFS storage.

### postgresqlStatefulSet
Here the deployment of the postgreSQL. A service is used to make sure an pod / application can connect to the database. 
A ConfigMap to contain the necessary information postgreSQL needs to run. This way a different config map can be used based on jursidiction aka Dev, Test, Prod, and etc. A future change would be use Secrets built into kubernetes instead of passwords for better security. Lastly, the statefulSet that deploys the postgreSQL pod. The main reason its a statefulset opposed to a deployment is when troubleshooting the pod name was consistant. This can be converted to a deployment.

### dummy-pod
Create a dummy pod using centos to test the persistants of postgreSQL. It uses centos which I would enter using kubectl exec -it dummy-pod -- /bin/sh
then install psql client and test the connection using the service. **[servicename].[namespace].svc.cluster.local**
