## Based kairen/kubeadm-ansible

## __Generate New Keys__
~~~
ssh-keygen -t rsa -N "" -f keys/provision
~~~


## __Create Nodes__
~~~
make create

or

make recreate 

or just

vagrant up

~~~

## __Deploy Kubernetes Cluster__
~~~
cd ansible

ansible-playbook create_cluster.yml

~~~

## __Connect to cluster__
~~~
vagrant ssh k8s1

kubectl --kubeconfig=/etc/kubernetes/admin.conf get nodes

or

vagrant scp /etc/kubernetes/admin.conf .

export KUBECONFIG=~/admin.conf

kubectl get node

~~~


