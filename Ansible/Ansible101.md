#### 1.Ansible介绍和安装

>通用自动化工具 用于配置管理 和 工作流程自动化
>
>使用SSH + Python
>
>配置采用yml文件
>
>自动化
>
> - 部署应用
> - 管理配置
> - 持续交付
> - 云服务管理
> - 网络设备管理

>Ansible 项目
>
>* Ansible Galaxy
>* Ansible Container
>* Ansible Tower

>推荐使用对应instance type上的包管理工具安装
>
>如果安装后没有config file 需要自己手动去repo中下载然后放在此目录
>config file = /etc/ansible/ansible.cfg
>
>使用ansible --version 检查当前配置信息

#### 2. 配置和主机清单

> ![491F2116-5B7A-4527-9DDF-235911A13DDB](/Users/qspeng/Documents/ansible-config.png)
>
> 主机清单 -- 待操作的instance list
>
> 可使用Jinja2/yaml形式定义
>
>
>
> 可分组 可动态
>
> 默认组
>
> ​	-all - 所有的主机
>
> ​	-ungrouped - 没有组的主机
>
> 变量合并 优先级由低到高
>
>  - all ->parent group -> child group -> host
>
> 分为管理主机<运行ansible的主机>和节点主机<ssh连接上操作的主机>
>



###3. 开始使用

Ansible模块：

Ansible - Usage: ansible <host-pattern> [options]

​	Eg: ansible localhost -m ping

​		ansible localhost -m command -a ifconfig

Ansible-console - 交互式运行：**Usage: ansible-console <host-pattern> [options]**

Ansible-doc - 查看文档：**Usage: ansible-doc <host-pattern> [options]**

​	Eg: ansible-doc -l 

Ansible-galaxy - 下载第三方扩展模块 类似于pip

Ansible-playbook - 复杂任务编排/复用

Ansible-pull - pull模式 在配置的机器上从git repo里面pull

Ansible-vault - 敏感信息 需要加解密运行



#### 核心概念：

##### inventory - 主机清单

---

默认文件路径 - /etc/ansible/hosts

以组作为区分

可以使用python code生成动态主机清单 - ansible all -i inventory.py -m ping



##### patterns - 匹配主机

---

all * - 所有

: - |

\\!xx - !xx

& - &



##### Ad-hoc

---

执行 临时的不需要保存的简单命令 复杂 命令使用playbook

web - group

ansible web -a "ifconfig eth0" - 默认使用**command**模块

ansible web -m **shell** -a "ifconfig eth0 | grep addr" - shell模块

ansible web -m **copy** -a "src=/etc/hosts dest=/tmp/hosts"

ansible web -m **apt** -a "name=foo update_cache=yes"

ansible web -m **user** -a "name=user comment='I am user ' uid=1040 group=admin"

ansible web -m **git** -a "repo=https://github.com/Icinga/icinga2.git dest=/tmp/myapp version=HEAD"

ansible web -m **service** -a "name=httpd state=started"

ansible web **-B 3600** -a "/usr/bin/long_running_operation --do-stuff" - 后台执行 3600s

ansible web -m **async_status** -a "jid=123456789" - 检查后台任务状态

ansible test -m **cron** -a "name='check dirs' minute='0' hour='5,2' job='ls -alh > /dev/null'" - 定时任务

ansible all -m setup - 获取系统信息



##### 常用模块

---

> 1.ping - 检查指定主机是否 能联通
>
> ```ansible -i hosts -u vagrant  vagrant  -m ping```
>
> 2.raw - 执行原始的命令 - 不需要远程node上安装python 也支持windows 
>
> 3.yum - readhat/cent os
>
> 4.apt - ubuntu
>
> 5.pip - python
>
> 6.synchronize - 使用rsync同步文件 
>
> 7.template  - 基于模板方式生成一个文件复制到远程主机 - 会替换其中的变量
>
> 8.copy - 复制操作
>
> 9.user/group
>
> useradd userdel usermod
>
> groupadd groupdel groupmod
>
> 10.service - 服务管理
>
> 11.get_url - 类似于wget 下载文件
>
> 12.fetch - 拉取文件
>
> 13.file - 文件操作
>
> 14.unarchive - 解压文件
>
> 15.command 和 shell - shell可以有特殊字符 command不支持



##### Playbook

---

> 由一系列的play组成
>
> 每个play是是对定义好的一组主机进行task操作  而task是利用ansible的module执行的
>
> playbook就是对play进行编排 
>
> 组成部分：
>
> Target section: 要执行playbook的远程主机组
>
> Variable section: 运行playbook需要使用的变量
>
> Task section: 将要在远程主机上执行的人物列表
>
> Handler section: 定义task执行完成后需要调用的任务 -- 即是回调



运行playbook前的一些检查：

```shell
# 语法检查
ansible-playbook -i hosts playbook-1.yml --syntax-check
# 主机名检查
ansible -i hosts vagrant --list-hosts
# 列出要执行的主机
ansible-playbook -i hosts playbook-1.yml --list-hosts
# 列出要执行的任务
ansible-playbook -i hosts playbook-1.yml --list-tasks

```

使用include来包含文件

```yaml
tasks:
  - include: foo.yml

#foo.yml
--- 
- name: test foo
  command: echo foo
```

可以动态包含

>```yaml
>#循环引用三次
>- include: foo.yml param={{item}}
>  with_items:
>  - 1
>  - 2
>  - 3
>#使用动态变量
>- include: "{{inventory_hostname}}.yml"
>#动态包含因为是动态的 - 也即是运行时确定的 所以不支持notify触发里面包含的task
>#以及--list-tasks不会显示 以及使用--start-at-task指定动态包含里面的人物
>#为解决上述限制 引入了静态包含
>- include: foo.yml
>  static: <yes|no|true|false>
>#符合条件的会自动被视为静态包含 - 没有循环 没有变量引用等等 也即是默认就是静态包含
>```

包含变量

>动态加载定义在其他文件中的变量 可以是json或者yaml
>
>```yaml
>- include_vars: myvars.yml
>- include_vars: "{{ item }}"
>  with_first_found:
>   - "{{ ansible_distribution }}.yml"
>   - "{{ ansible_os_family }}.yml"
>   - "default.yml"
>```

Role

>说白了Role一种组织文件的结构 遵循这种结构允许ansible自动去加载
>
>vars_files tasks and handlers.
>
>按Role分组的内容允许与其他用户共享.
>
>这里的分组 指的是 一个Role其实是同一类型的任务的分组 每个role下面就是上面所述的文件组织

>Role结构示例：
>
>```yaml
>site.yml
>webservers.yml
>fooservers.yml
>roles/
>   common/
>     tasks/
>     handlers/
>     files/
>     templates/
>     vars/
>     defaults/
>     meta/ - 可用于定义一些role的依赖
>   webservers/
>     tasks/
>     defaults/
>     meta/
>```
>
>上面的webservers和common就是role
>
>里面的目录名称是约定俗成的
>
>ansible要求**每个目录**必须包含一个main.yml 用于组织其目录里面相关的内容(tasks handlers defaults vars files templates meta)
>
>还可以指定目录用于包含自定义的module或者plugin
>
>Role的使用：
>
>```yaml
>---
>- hosts: webservers
>  roles:
>     - common
>     - webservers
>```
>
>Role的执行顺序
>
>1.pre_tasks
>
>2.上述tasks触发的handlers
>
>3.roles里面列出的role 有依赖会先执行依赖的role
>
>4.上述tasks触发的handlers
>
>5.post_taks
>
>6.上述tasks触发的handlers

> role和include都是用来拆分 组织playbook的方式
>
> 拆分很好理解 role就像是一个module

>```yaml
>---
>#2.4 以后 可以再task里面直接引用role
>- hosts: webservers
>  tasks:
>  - debug:
>      msg: "before we run our role"
>  - import_role:
>      name: example
>  - include_role:
>      name: example
>  - debug:
>      msg: "after we ran our role"
>```
>
>```yaml
>---
>#动态include
>- hosts: webservers
>  tasks:
>  - include_role:
>      name: some_role
>    when: "ansible_facts['os_family'] == 'RedHat'"
>```
>
>```yaml
>---
># 支持关键字覆盖role里面的属性
>- hosts: webservers
>  roles:
>    - common
>    - role: foo_app_instance
>      vars:
>         dir: '/opt/a'
>         app_port: 5000
>    - role: foo_app_instance
>      vars:
>         dir: '/opt/b'
>         app_port: 5001
>```
>
>默认一个role只执行一次
>
>allow_duplicates: true - 可以执行多次
>
>meta/main.yml - 用于定义role依赖
>
>```yaml
>---
>dependencies:
>  - role: common
>    vars:
>      some_parameter: 3
>  - role: apache
>    vars:
>      apache_port: 80
>  - role: postgres
>    vars:
>      dbname: blarg
>      other_parameter: 12
>```

变量

>字母 数字 下划线组成
>
>且字母开头
>
>不与python属性和方法名冲突

>来源：
>
>命令行传递 - extra vars
>
>```shell
>ansible-playbook release.yml -e "user=starbuck"
>```
>
>inventroy中定义 - inventory vars
>
>```
>host3 http_port=80 # 定义主机变量
>[webservers:vars] # 定义组的变量
>ntp_server=ntp.example.com
>```
>
>playbook中定义(包括角色和文件中的) - play vars
>
>```yaml
>- hosts: webservers
>   vars:
>     http_port: 80
>   include_vars: myvars.yml
>
>- hosts: webservers
>  vars_files:
>    - /vars/external_vars.yml		
>```
>
>role中有默认变量 - 存放于defaults/main.yml中
>
>通过交互定义变量
>
>```yaml
>---
>- hosts: server
>  vars_prompt:
>    - name: web
>      prompt: 'Please input the web server:'
>      private: no
>```
>
>定义role的变量
>
>```yaml
>roles:
>   - { role: app_user, name: Ian    }
>```
>
>注册变量
>
>```yaml
>---
>- hosts: all 
>  tasks:
>  - shell: uptime
>    register: result #注册此项任务的结果为变量result
>  - name: show uptime
>    debug: var=result
>```
>
>变量优先级
>
>• role defaults
> • inventory vars
> • inventory group_vars
> • inventory host_vars
> • playbook group_vars
> • playbook host_vars
> • host facts
> • play vars
> • play vars_prompt
> • play vars_files
> • registered vars
> • set_facts
> • role and include vars
> • block vars (only for tasks in block)
> • task vars (only for the task)
> • extra vars (always win precedence)
>
>如果多个组具有相同的变量，则最后一个加载获胜。
>
>变量范围
>
>全局 - config  环境变量 命令行设置
>
>play - task
>
>host

Facts

>Facts 是用来采集目标系统信息的，具体是用setup模块来采集得
>
>```ansible localhost -m setup```
>
>```ansible localhost -m setup -a 'filter=ansible_*_mb'```
>
>facts在host上可使用文件 redis memcached等作为缓存
>
>关闭facts:
>
>```yaml
>- hosts: whatever
>  gather_facts: no
>```

Jinjia2

>ansible中的template默认jinjia2
>
>可以用于过滤 获取 操作一些值 定义变量等等

条件判断和循环

>When - 后跟jinja2表达式
>
>```yaml
>tasks:
>  - name: "shut down Debian flavored systems"
>    command: /sbin/shutdown -t now
>    when: ansible_os_family == "Debian"
>```
>
>支持与或非以及过滤器过滤
>
>循环判断等等
>
>Failed_when -满足条件时 任务失败
>
>```yaml
>tasks:
>    - command: echo faild.
>      register: command_result
>      failed_when: "'faild' in command_result.stdout"
>   - debug: msg="echo test"
>```
>
>Changed_when - 当xx改变任务的状态为changed
>
>以上两个适用于当我们需要自己去确定任务是否失败或者改变的情况下使用 也即是默认的行为不适用
>
>When也可用于任务/Role之间的依赖设置
>
>```yaml
>tasks:
>  - shell: service nginx configtest
>    ignore_errors: True
>    register: result
>
>  - shell: service nginx reload
>    when: result|success
>
>  - local_action: mail subject='Nginx config error.'
>    when: result|failed
>```

分组

>在满足条件时执行多个任务 - 这多个任务就需要分组
>
>```yaml
>tasks:
>  - block:
>      - debug: msg='i execute normally'
>      - command: /bin/false
>      - debug: msg='i never execute, cause ERROR!'
>      when: ansible_distribution == 'CentOS'
>    rescue:
>      - debug: msg='I caught an error'
>      - command: /bin/false
>      - debug: msg='I also never execute :-('
>    always:
>      - debug: msg="this always executes"
>```
>
>rescue中可指定handler运行
>
>```yaml
>tasks:
>  - block:
>      - debug: msg='i execute normally'
>        notify: run me even after an error
>      - command: /bin/false
>    rescue:
>      - name: make sure all handlers run
> handlers:
>   - name: run me even after an error
>     debug: msg='this handler runs even on error'
>```

Debug

>在执行期间打印自定义信息
>
>```yaml
>- debug: msg="System {{ inventory_hostname }} has uuid {{ ansible_product_uuid }}"
>- debug: var=result verbosity=2
>
>```
>
>开启调试模式 会在任务失败时调用调试器进入调试器模式
>
>```yaml
>- hosts: test
>  strategy: debug
>  gather_facts: no
>  vars:
>    var1: value1
>  tasks:
>    - name: wrong variable
>      ping: data={{ wrong_var }}
>```
>
>调试器可使用命令
>
>> p - 显示此次失败的原因
>>
>>
>>p task - 显示此次任务的名称
>>
>>
>>p task.args - 显示模块的参数
>>
>>
>>p host - 显示执行此次任务的主机
>>
>>
>>p result - 显示此次任务的结果
>>
>>
>>p vars - 显示当前的变量
>>
>>
>>vars[key] = value - 更新vars中的值
>>
>>
>>task.args[key] = value - 更新模块的参数。
>>
>>
>>r - 再次执行此任务
>>
>>
>>c - 继续执行
>>
>>q - 退出debug模式

高级特性

>async - 异步操作
>
>poll - 轮询
>
>--check 检查模式，类比于dryrun
>
>ignore_errors: true - 在任务执行错误时 忽略并继续执行任务
>
>serial: n - 滚动执行，即以n个主机 n个主机开始按批次执行
>
>max_fail_percentage: 30 - 最大失败数目
>
>run_once - task run once
>
>any_errors_fatal: true - 任一错误时中断ansible
>
>environment:
>
>​	key: value
>
>设置环境变量
>
>prompts: 运行时 输入内容
>
>tags - 标记任务
>
>```shell
>ansible-playbook example.yml --list-tags
>ansible-playbook example.yml --tags packages
>ansible-playbook example.yml --skip-tags "notification"
>```
>
>默认情况下ansible运行就像指定了'`-tags all`'。运行playbook中的未标记任务 -tags untagged
>
>wait_for - 等待端口、文件可用 才执行任务
>
>wair_for: port=8000 delay=10

Lookup插件

>File - 获取文件内容
>
>```yaml
>---
>- hosts: all
>  vars:
>     contents: "{{ lookup('file', '/etc/foo.txt') }}"
>tasks:
>- debug: msg="the value of foo.txt is {{ contents }}"
>```
>
>还有其他功能包含 生成密码字符串 读取csv文件 ini文件
>
>dig命令



支持windows系统 - 利用PowerShell 管理机必须是Linux且必须预安装Python的Winrm模块

