Step-by-Step Execution


Plaintext
echo "Starting deployment checks"
uptime
df -h /
echo "Task complete"
Run the playbook against your target inventory or localhost:

Bash
ansible-playbook -i localhost, -c local run_commands.yml
