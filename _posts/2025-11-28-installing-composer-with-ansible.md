---
layout: post
title: "Install Composer with Ansible"
---

I recently had to install Composer on multiple machines. Ansible proved to be
the right tool for the job.

Ansible is a well-known provisioning tool where all configuration is written in
simple YAML files. The target machines require no client installation.

In this post, I will walk through the Ansible role I created to install
Composer.

Here is the structure of the role. I named this role `composer`:

```text
roles/
└── composer
    └── tasks
        └── main.yml
```

As you can see, this role is very simple and contains a single task file in
`./roles/composer/tasks/main.yml`:

```yaml
---

- name: Download Composer
  ansible.builtin.get_url:
    url: https://github.com/composer/composer/releases/download/2.9.2/composer.phar
    dest: /usr/local/bin/composer
    mode: '+x'
  become: yes

- name: Update Composer
  ansible.builtin.command: /usr/local/bin/composer self-update --no-interaction
  become: yes

- name: Add Composer to PATH
  ansible.builtin.lineinfile:
    path: '{{ ansible_env.HOME }}/.bashrc'
    regexp: '^export PATH=.*composer/vendor/bin'
    line: 'export PATH="{{ ansible_env.HOME }}/.config/composer/vendor/bin:$PATH"'
```

The first task downloads the Phar version of Composer. If Composer is already
installed, Ansible will not re-download it.

Note that the version of Composer is hardcoded. However, this is not a concern
because the second task updates Composer anyway.

Finally, the last task adds Composer's global `bin` directory to the PATH. This is
important because it allows you to run binaries installed through the
`composer global require` command.

Before using this role, make sure it matches your environment:

1. Check if you really need to execute tasks with elevated privileges
   (`become: yes`).
2. Verify that Composer's global `bin` directory is correct. You can use this
   command to retrieve this value:
   `composer global config bin-dir --absolute`.
