# Usage Example

## Install Role

### From Galaxy

```bash
ansible-galaxy install ndkprd.fortiadc-glb-servers
```

### From Repository
```bash
cat <<EOF > requirements.yaml
- name: ndkprd.fortiadc-glb-data-centers
  scm: git
  src: https://github.com/ndkprd/ansible-role-fortiadc-glb-servers.git
  version: main
EOF
ansible-galaxy install -r requirements.yaml
```

## Run Playbook

```bash
ansible-playbook -i hosts fortiadc-glb-servers.yaml --skip-tags debug
```
