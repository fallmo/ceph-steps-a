# Ceph components
- mon (monitor)
- mgr (manager dashboard)
- osd (disks)
- mds (filesystem manager)
- rgw (rados object gateway)


# Install steps
1. Register
```
subscription-manager register
```
```
subscription-manager refresh
```
2. Attach appropriate pool id
```
subscription-manager list --available --matches 'Red Hat Ceph Storage'
```
```
subscription-manager attach --pool=POOL_ID
```
3. Enable the repos
```
subscription-manager repos --disable=*
subscription-manager repos --enable=rhel-9-for-x86_64-baseos-rpms
subscription-manager repos --enable=rhel-9-for-x86_64-appstream-rpms
subscription-manager repos --enable=rhceph-6-tools-for-rhel-9-x86_64-rpms
```
```
dnf update
```
4.  Install packages
```
dnf install cephadm-ansible
```
5. Copy ssh-keys and enable no login on all nodes
```
ansible <your_target_group> -m redhat_subscription -a "state=present username=<your_username> password=<your_password> pool=<your_pool_id>"
```
5. Run ansible preflight playbook
```
cd /usr/share/cephadm-ansible
```
create ./hosts (inventory file)
```ini
[nodes]
client01
client02
client03
```
```bash
ansible-playbook -i ./hosts ./cephadm-preflight.yml --extra-vars "ceph_origin=rhcs"
```
6. Copy root ssh public key to hosts, as well as ceph.pub and also ceph.conf file

6. Create Spec file (spec.yaml) and ./registry.json file
```yaml
service_type: host
addr: 10.21.0.10 
hostname: storage-1
location:
  root: default
  datacenter: dc_accel
  room: floor_2
  rack: rack_1
  row: "21"
labels:
  - _admin
---
service_type: host
addr: 10.21.0.11 
hostname: storage-2
location:
  root: default
  datacenter: dc_accel
  room: floor_2
  rack: rack_1
  row: "19"
---
service_type: host
addr: 10.21.0.12
hostname: storage-3
location:
  root: default
  datacenter: dc_accel
  room: floor_2
  rack: rack_1
  row: "17"
labels:
  - _admin
---
service_type: host
addr: 10.21.0.13
hostname: storage-4
location:
  root: default
  datacenter: dc_accel
  room: floor_2
  rack: rack_1
  row: "15"
---
service_type: host
addr: 10.21.0.14
hostname: storage-5
location:
  root: default
  datacenter: dc_accel
  room: floor_2
  rack: rack_1
  row: "13"
---
service_type: host
addr: 10.21.0.15
hostname: storage-6
location:
  root: default
  datacenter: dc_accel
  room: floor_2
  rack: rack_1
  row: "11"
---
service_type: mon
placement:
  hosts:
  - storage-1
  - storage-2
  - storage-3
---
service_type: mgr
placement:
  hosts:
  - storage-3
  - storage-4
---
service_type: mds
service_id: fsdaemon
service_name: mds.fsdaemon
placement:
  hosts:
  - storage-2
  - storage-3
---
service_type: rgw
service_id: objectgw
service_name: rgw.objectgw
placement:
  hosts:
  - storage-5
  - storage-6
spec:
  rgw_frontend_port: 8080
---
service_type: osd
service_id: default_drive_group
service_name: osd.all-available-devices
placement:
  host_pattern: "*"
data_devices:
  all: true
```
```json
{
 "url":"registry.redhat.io",
 "username":"myuser1",
 "password":"mypassword1"
}
```

7. Run deployment
```bash
cephadm bootstrap --mon-ip=10.21.0.10 \
--cluster-network 10.20.0.0/24 \
--allow-fqdn-hostname \
--registry-json ./registry.json \

--apply-spec=spec.yaml \
--initial-dashboard-password=redhat \
```


