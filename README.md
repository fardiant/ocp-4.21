Step-by-Step Execution

dnf install ansible-core -y


echo "Starting deployment checks"
uptime
df -h /
echo "Task complete"

Run the playbook against your target inventory or localhost:

ansible-playbook -i localhost, -c local run_commands.yml
