# ansible-haproxy-nginx-postgresql-elastic
follow installation by using next playbooks

    bootstrap-mgmt.sh - Install/Config Ansible & Deploy Code Snippets
             examples - Code Snippets
          Vagrantfile - Defines Vagrant Environment

rsync back from vagrant virtual box

vagrant ssh-config    - check actual IP address and private_key
scp -rP 2222 -i /vagrant/.vagrant/machines/mgmt/virtualbox/private_key vagrant@127.0.0.1:~/* ~/tmp

Recap of ansible steps to run this cool stuff

                                      vagrant up - setup all environment
                                vagrant ssh mgmt - connects to management node
                            ansible --list-hosts - display all known hosts
                            ansible web1 -m ping - check web1 node activation
                                ssh-keyscan web1 - scan web1 node
    ssh-keyscan lb web1 web2 >> .ssh/known_hosts - add node to known_hosts
                       ssh-keygen -t rsa -b 2048 - generate private ssh key
    ansible-playbook 0.ssh.addkey.yml --ask-pass - ask for pass one time, share/push public key to nodes
                  ansible all -m ping --ask-pass - check connection to nodes
                     ansible-playbook 1-site.yml - install nginx, deploy test html file
                  curl -I http://localhost:8080/ - check connection and run smoke tests for deployment
                 ansible-playbook e3-rolling.yml - 0-down-time deployment to cluster
    ansible web1 -m apt -a "name=postgresql state=installed" --become - installing postgres
    ansible web1 -m copy -a "src=/home/vagrant/files/db.conf dest=/etc/db.conf mode=644 owner=root group=root" --sudo
    ansible web1 -m service -a "name=nginx state=restarted"
    ansible all -m shell -a "uptime"
    ansible all -m shell -a "uname -a"
    ansible web -m shell -a "/sbin/reboot"
    ansible web1 -m setup | less
    ansible web1 -m setup -a "filter=ansible_distribution"
    ansible web1 -m setup -a "filter=ansible_distribution*"
    ansible web1 -m setup -a "filter=ansible_all_ipv4_addresses"
    sudo apt-get install apache2-utils - stress testing
    ab -n 10000 -c 25 http://localhost:8080/

