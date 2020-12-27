```shell
git clone https://github.com/lingxiankong/ansible-kubernetes-on-openstack -b v1.20
cd ansible-kubernetes-on-openstack

# Prepare OpenStack credentials
sourcedemo
image=$(openstack image list --name ubuntu-bionic -c ID -f value); echo $image
auth_url=$(export | grep OS_AUTH_URL | awk -F '"' '{print $2}'); echo $auth_url
sourceadm
user_id=$(openstack user show demo -c id -f value)
tenant_id=$(openstack project show demo -c id -f value)
sourcedemo
password=password

# Install Kubernetes cluster
cat <<EOF > vars.json
{
  "reource_prefix": "lingxian-k8s",
  "rebuild": true,
  "master_node_number": 1,
  "worker_node_number": 1,
  "auth_url": "$auth_url",
  "user_id": "$user_id",
  "password": "$password",
  "tenant_id": "$tenant_id",
  "region": "RegionOne",
  "image": "$image",
  "flavor": "2u4g10g",
  "key_name": "lingxian_key",
  "private_key_file": "$HOME/.ssh/id_rsa",
  "external_network_name": "public",
  "boot_from_vol": "true",
  "bootstrap": true,
  "subnet_dns_nameservers": "8.8.8.8",
  "remove_worker_floatingip": false,
  "allow_pod_on_master": true,
  "k8s_version": "1.20.1",
  "enable_addons": false
}
EOF
ansible-playbook site.yaml -v --extra-vars "@vars.json"

# Destroy cluster
cat <<EOF > clean_vars.json
{
  "reource_prefix": "lingxian-k8s",
  "auth_url": "$auth_url",
  "user_id": "$user_id",
  "password": "$password",
  "tenant_id": "$tenant_id",
  "region": "RegionOne",
  "master_node_number": 1,
  "worker_node_number": 1
}
EOF
ansible-playbook clean.yaml -e "@clean_vars.json"
```