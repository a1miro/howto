# Running Ansible example


```bash
source ~/venvs/flask/bin/activate && ansible-navigator run test_remote.yml -i inventory --execution-environment-image postgresql_ee --mode stdout --pull-policy missing --container-options='--user=0' -u root
```

```bash
ansible-playbook test_remote.yml -i inventory -u root
```

```bash
docker run -d -it \
  --name ansible-dev \
  -v $(pwd):/workspace \
  -w /workspace \
  ghcr.io/ansible-community/community-ee-base \
  tail -f /dev/null
```

```bash
docker exec ansible-dev ansible-playbook test_remote.yml -i inventory -u root
```

## References
[Ansible Community Documentatiion](https://docs.ansible.com/ansible/latest/getting_started_ee/run_execution_environment.html)