---
layout: post
title: "Installing Composer with Ansible"
---

I recently had to install Composer on multiple machines, Ansible was the right
tool for the job.
Ansible was the right tool for the job. Ansible is a well-known provisioning
tool where all configuration is written in simple Yaml files, and the target
machines don't need any client installed.

In this post I will walk through the Ansible _role_ I created to install
Composer.

Here's the structure of the _role_.
Without surprise, I called this _role_ `composer`.

```text
roles/
└── composer
    └── tasks
        └── main.yml
```

As you can see, this _role_ is extremely simple.
It contains a single _task list_ in `./roles/composer/tasks/main.yml`:

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

The first _task_ downloads the _Phar_ version of Composer.
If Composer is already installed, Ansible won't re-download it.

Notice that version of Composer is hardcoded, this is not a big concern because
in the second _task_ we update Composer anyway.

Finally, the last _task_ adds the Composer's _global bin directory_ into the
PATH.
This is important because it lets you run binaries installed through the
`composer global require` command.

Before using this _role_ as-is, make sure it matches your environment:

1. Check if you really need to execute tasks with elevated privileges (`become: yes`).
2. Also check if the Composer's _global bin directory_ is correct. You can use
   this command to retrieve this value:
   `composer global config bin-dir --absolute`.
