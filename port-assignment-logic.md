# Port Assignment Logic
- Editor Container ports are always in the 2000-2999 range, separated by 10s to allow 
  for future expansion. Port numbers are fixed, static, and never move. PBX has a static 
  routing table mapping prefixes to ports.
- Shell Container ports are in the 4000s, also separated by 10s.
- Dashboard Container ports are in the 6000s, also separated by 10s.
- Global services (Clacks, Caddy, Gitlab, Archive, Deep Freeze) use ports in the 19000s 
  to clearly distinguish them from containerized services.

# Determining Hostnames and Port Assignments
- For FastAPI service code all LAN hostnames and port assignments are imported from service-hosts.py, which is generated from the service-hosts.yaml file.
- For C# code, the same service-hosts.yaml file is used to generate a C# class with static properties for each hostname and port assignment.
- For Typescript code, the same service-hosts.yaml file is used to generate a TypeScript module with constants for each hostname and port assignment.
- For process-compose, the service-hosts.yaml file is used to generate the docker-compose.yml file with the appropriate port mappings for each container.
- For docker-compose, the service-hosts.yaml file is used to generate the docker-compose.yml file with the appropriate port mappings for each container.
- /etc/hosts for each container is generated from the service-hosts.yaml file to ensure that all containers can resolve the hostnames of other services correctly.

## Regenerating Hostname and Port Assignments
- Whenever a new service is added or an existing service's port assignment changes, the service-hosts.yaml file should be updated with the new hostname and port information.
- After updating the service-hosts.yaml file, the code generation scripts should be run to regenerate the service-hosts.py, C# class, TypeScript module, and docker-compose.yml files to reflect the new hostname and port assignments.
- Be sure to commit the updated service-hosts.yaml file and the regenerated files to version control to keep track of changes to hostnames and port assignments over time.
- CI/CD pipelines are configured to automatically run the code generation scripts whenever changes are detected in the service-hosts.yaml file.