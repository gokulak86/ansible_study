# ansible_study
Case studied in ansible


Sample output logs of execution,

 [root@localhost ansible_playbooks]#
 
    ansible-playbook test.yml --tags=task_role1
    
[WARNING]:  * Failed to parse /etc/ansible/hosts with yaml plugin: YAML inventory has invalid
structure, it should be a dictionary, got: <class 'ansible.parsing.yaml.objects.AnsibleUnicode'>
[WARNING]:  * Failed to parse /etc/ansible/hosts with ini plugin: /etc/ansible/hosts:44: Expected
key=value host variable assignment, got: slc15drk
[WARNING]: Unable to parse /etc/ansible/hosts as an inventory source
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit
localhost does not match 'all'

PLAY [Playbook1] *************************************************************************************

TASK [include_role : role1] **************************************************************************

TASK [role1 : Task1 inside role] *********************************************************************
ok: [localhost] => {
    "msg": "Inside task1 role1"
}

TASK [role1 : Role1 task2 accessong role var] ********************************************************
ok: [localhost] => {
    "msg": " Role var name is gokul "
}

PLAY [Plabook2] **************************************************************************************

TASK [command] ***************************************************************************************
changed: [localhost]

TASK [include_role : {{  role_name_var }}] ***********************************************************
skipping: [localhost] => (item=role2)    # Here it is interesting, it is due to tags: always inside include_role. Due to always include_role block is called. But none of the tasks inside included roles are executed since they dont have the passed tag --tags=task_role1


     - include_role:
          name: "{{  role_name_var }}"
          apply:
            tags: "{{  role_name_var }}"
        tags: always
        
Though it is executed No adverse effects. Since include_role is dynamic, tag: always is just applied to the include_role statement not to the tasks inside the included roles. That is the reason why we had to write the below code along with include_role, so the corresponding tags( role1 and role2 will be applied  to all tasks inside the respective roles)and executed when called with --tag=role1 or --tag=role2

          apply:
            tags: "{{  role_name_var }}
            

           

PLAY RECAP *******************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[root@localhost ansible_playbooks]# 
[root@localhost ansible_playbooks]# 
[root@localhost ansible_playbooks]# 
    
    ansible-playbook test.yml --tags=role1  # role1 will be executed.
    
[WARNING]:  * Failed to parse /etc/ansible/hosts with yaml plugin: YAML inventory has invalid
structure, it should be a dictionary, got: <class 'ansible.parsing.yaml.objects.AnsibleUnicode'>
[WARNING]:  * Failed to parse /etc/ansible/hosts with ini plugin: /etc/ansible/hosts:44: Expected
key=value host variable assignment, got: slc15drk
[WARNING]: Unable to parse /etc/ansible/hosts as an inventory source
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit
localhost does not match 'all'

PLAY [Playbook1] *************************************************************************************

PLAY [Plabook2] **************************************************************************************

TASK [command] ***************************************************************************************
changed: [localhost]

TASK [include_role : {{  role_name_var }}] ***********************************************************
skipping: [localhost] => (item=role2) 

TASK [role1 : Task1 inside role] *********************************************************************
ok: [localhost] => {
    "msg": "Inside task1 role1"
}

TASK [role1 : Role1 task2 accessong role var] ********************************************************
ok: [localhost] => {
    "msg": " Role var name is gokul "
}

PLAY RECAP *******************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[root@localhost ansible_playbooks]# 
    
    ansible-playbook test.yml --tags=role2 # role2 will be executed. But will be skipped since we have written a when clause in the include_role 
[WARNING]:  * Failed to parse /etc/ansible/hosts with yaml plugin: YAML inventory has invalid
structure, it should be a dictionary, got: <class 'ansible.parsing.yaml.objects.AnsibleUnicode'>
[WARNING]:  * Failed to parse /etc/ansible/hosts with ini plugin: /etc/ansible/hosts:44: Expected
key=value host variable assignment, got: slc15drk
[WARNING]: Unable to parse /etc/ansible/hosts as an inventory source
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit
localhost does not match 'all'

PLAY [Playbook1] *************************************************************************************

PLAY [Plabook2] **************************************************************************************

TASK [command] ***************************************************************************************
changed: [localhost]

TASK [include_role : {{  role_name_var }}] ***********************************************************
skipping: [localhost] => (item=role2) 

PLAY RECAP *******************************************************************************************
localhost                  : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

# Block tags
    ansible-playbook test.yml --tags=block2.
Here the entire block is executed. So all tasks inside the block will be executed irrespective of the task specific tags

PLAY [Playbook1] *************************************************************************************

PLAY [Plabook2] **************************************************************************************

TASK [command] ***************************************************************************************
changed: [localhost]

TASK [debug] *****************************************************************************************
ok: [localhost] => {
    "msg": "inside block2"
}

TASK [include_role : {{  role_name_var }}] ***********************************************************
skipping: [localhost] => (item=role2) 

TASK [role1 : Task1 inside role] *********************************************************************
ok: [localhost] => {
    "msg": "Inside task1 role1"
}

TASK [role1 : Role1 task2 accessong role var] ********************************************************
ok: [localhost] => {
    "msg": " Role var name is gokul "
}

PLAY RECAP *******************************************************************************************
localhost                  : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

# Note:

When I tried to execute by commenting out tags:always in the include_role task and uncommenting tags: '{{  role_name_var }}' like below. It says role_name_var is not defined!!  Though variable name is accessible inside 
the when command its not accessible in the tags

      - include_role:
          name: "{{  role_name_var }}"
          apply:
            tags: "{{  role_name_var }}"
        tags: '{{  role_name_var }}'
        #tags: always
        when:
         - 'role_name_var not in var2.stdout'
        
[root@localhost ansible_playbooks]# 

    ansible-playbook test.yml --tags=role1
[WARNING]:  * Failed to parse /etc/ansible/hosts with yaml plugin: YAML inventory has invalid
structure, it should be a dictionary, got: <class 'ansible.parsing.yaml.objects.AnsibleUnicode'>
[WARNING]:  * Failed to parse /etc/ansible/hosts with ini plugin: /etc/ansible/hosts:44: Expected
key=value host variable assignment, got: slc15drk
[WARNING]: Unable to parse /etc/ansible/hosts as an inventory source
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit
localhost does not match 'all'

PLAY [Playbook1] *************************************************************************************

PLAY [Plabook2] **************************************************************************************
       
        ERROR! 'role_name_var' is undefined
[root@localhost ansible_playbooks]# 
       
       

# Another interesting scenario, tag inside roles.

 I have added a tag r1t2 to the second task in role1, file below. Am calling with main test.yml with --tags=r1t2 , and if you check the O/P. That task tagged with r1t2 alone is executed. It gets executed only with, tag: always
 

      - include_role:
          name: "{{  role_name_var }}"
          apply:
            tags: "{{  role_name_var }}"
        tags: always
    

[root@localhost ansible_playbooks]# 
        
    cat roles/role1/tasks/main.yml 

    ---
    tasks file for role1

    - name: "Task1 inside role"
      debug:
       msg: "Inside task1 role1"

    - name: "Role1 task2 accessong role var"
      debug:
        msg: " Role var name is {{ name }} "
      tags: r1t2
  
  
[root@localhost ansible_playbooks]# 


[root@localhost ansible_playbooks]#
      
    ansible-playbook test.yml --tags=r1t2

[WARNING]:  * Failed to parse /etc/ansible/hosts with yaml plugin: YAML inventory has invalid
structure, it should be a dictionary, got: <class 'ansible.parsing.yaml.objects.AnsibleUnicode'>
[WARNING]:  * Failed to parse /etc/ansible/hosts with ini plugin: /etc/ansible/hosts:44: Expected
key=value host variable assignment, got: slc15drk
[WARNING]: Unable to parse /etc/ansible/hosts as an inventory source
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit
localhost does not match 'all'

PLAY [Playbook1] *************************************************************************************

PLAY [Plabook2] **************************************************************************************

TASK [command] ***************************************************************************************
changed: [localhost]

TASK [include_role : {{  role_name_var }}] ***********************************************************
skipping: [localhost] => (item=role2) 

TASK [role1 : Role1 task2 accessong role var] ********************************************************
ok: [localhost] => {
    "msg": " Role var name is gokul "
}

PLAY RECAP *******************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

