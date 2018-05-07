# Fonos

Open source multi-room speaker system for Raspberry Pis


## Summary

Mraa mraa mraa long live cyber squirrel.


## Deploying it

- clone the repo
- copy `hosts.sample` to `hosts`
- customize `hosts`
  - set proper hostnames
  - fill in the soundcloud and spotify variables (with tokens provided [here](https://www.mopidy.com/authenticate/))
- deploy to target:
  ```bash
  ansible-playbook playbook.yml -i hosts
  ```


## Principles

- systemd user units
- user configuration files (in ~/.config)
- try to be independent from system config files as much as possible

