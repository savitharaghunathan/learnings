# Addressing Docker Permissions Issue in Ansible Automation
Date: Jan 16th, 2024

While automating infrastructure with Ansible and deploying a Quarkus app in dev mode using Docker, I faced a specific permission issue. The aim was to achieve a smooth automated deployment, but the initial script run encountered the following error:

```
"stderr": "permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get \"http://%2Fvar%2Frun%2Fdocker.sock/v1.24/version\": dial unix /var/run/docker.sock: connect: permission denied",
"stderr_lines": ["permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get \"http://%2Fvar%2Frun%2Fdocker.sock/v1.24/version\": dial unix /var/run/docker.sock: connect: permission denied"],

```

To troubleshoot the issue, my initial suspicion was that the User lacked membership in the Docker group. Consequently, I added the local user to the Docker group and ensured the user had the necessary permissions to execute Docker commands:

```
- name: Add user to the docker group
  become: true
  user:
    name: "{{ ansible_user }}"
    groups: docker
    append: yes
```

Despite this, the problem persisted. Further looking into the issue revealed that the group permissions did not take effect immediately. Changes in group membership only become effective after the user logs out and logs back in, and Ansible tasks are executed in separate shells.

A solution that proved effective was to leverage the Ansible directive `meta: reset_connection`. This ensured that the connection was reset, and group permissions were applied immediately:

```
- name: Add user to the docker group
  become: true
  user:
    name: "{{ ansible_user }}"
    groups: docker
    append: yes

- meta: reset_connection

- name: Display Docker version
  shell: docker version
  register: docker_version

- name: Display Docker version
  debug:
    var: docker_version.stdout_lines

```

By incorporating this approach, the Docker permissions issue was successfully addressed, providing a smoother automation process for deploying Quarkus apps with Ansible.

