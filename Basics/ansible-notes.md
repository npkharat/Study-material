# Ansible Interview Notes

## What is Ansible?

Ansible is an open-source IT automation tool used for:

* Configuration Management
* Application Deployment
* Task Automation
* Infrastructure Provisioning
* Orchestration

Ansible follows a **push-based model**, where the Control Node pushes configurations and commands to target machines.

---

## Why Did Ansible Come Into Existence?

Ansible was created to simplify infrastructure automation by:

* Eliminating the need for agents
* Using SSH for communication
* Providing simple YAML-based configuration
* Reducing operational complexity
* Improving readability and maintainability

Compared to older tools, Ansible offers faster setup and easier learning.

---

# Puppet vs Chef vs Ansible

| Feature          | Puppet           | Chef               | Ansible                        |
| ---------------- | ---------------- | ------------------ | ------------------------------ |
| Architecture     | Agent-Based      | Agent-Based        | Agentless                      |
| Language         | DSL              | Ruby               | YAML                           |
| Setup Complexity | High             | High               | Low                            |
| Learning Curve   | Moderate         | High               | Easy                           |
| Communication    | Agent            | Agent              | SSH/WinRM                      |
| Best For         | Large Enterprise | Complex Automation | Fast Infrastructure Automation |

### Interview Answer

Puppet, Chef, and Ansible are configuration management tools used in DevOps.

Puppet and Chef are agent-based solutions that require software installation on every managed node. They are powerful for large-scale enterprise environments but involve higher setup and maintenance complexity.

Ansible is agentless and uses SSH or WinRM for communication. It is easier to install, learn, and maintain because automation is written in simple YAML playbooks. Due to its simplicity and quick adoption, Ansible is widely used in modern DevOps environments.

---

# Ansible Architecture

Ansible Architecture consists of:

## 1. Control Node

The machine where Ansible is installed and automation tasks are executed.

## 2. Managed Nodes

Target servers managed by Ansible.

Communication happens through:

* SSH (Linux)
* WinRM (Windows)

No agent installation is required.

## 3. Inventory

Contains information about managed hosts and host groups.

## 4. Playbooks

YAML files containing automation tasks.

## 5. Modules

Reusable units of work used by Ansible.

Examples:

* apt
* yum
* copy
* file
* service
* user

### Architecture Flow

Control Node → Inventory → Playbook → Module → Managed Nodes

---

# Tasks Performed by Ansible

Ansible can automate:

### Configuration Management

* Install packages
* Configure services
* Manage users and groups

### Application Deployment

* Deploy applications
* Update applications
* Rollback deployments

### Server Provisioning

* Create servers
* Configure cloud resources

### Orchestration

* Coordinate workflows across multiple systems

### Cloud Automation

Provision resources in:

* AWS
* Azure
* Google Cloud

### System Administration

* File management
* Service management
* Permission management
* Scheduled tasks

---

# Ansible vs Shell vs Python

| Feature                   | Shell     | Python      | Ansible   |
| ------------------------- | --------- | ----------- | --------- |
| Complexity                | Low       | Medium/High | Low       |
| Scalability               | Limited   | Good        | Excellent |
| Infrastructure Automation | Limited   | Possible    | Native    |
| Readability               | Moderate  | Good        | Excellent |
| Multi-Server Management   | Difficult | Custom Code | Built-In  |

### Interview Answer

Shell scripting is suitable for simple operating system tasks.

Python is used for complex automation and custom logic.

Ansible is designed specifically for infrastructure automation and configuration management across multiple servers using simple YAML playbooks. It is easier to scale and maintain in enterprise environments.

---

# Prerequisites for Using Ansible

To use Ansible, you need:

### Control Node

Ansible installed machine.

### Managed Nodes

Target servers to manage.

### Network Connectivity

Control node must reach managed nodes.

### SSH Access

Required for Linux systems.

### WinRM

Required for Windows systems.

### Python

Python must be installed on Linux managed nodes.

### Inventory

Defines target hosts.

### Permissions

Appropriate sudo privileges are typically required.

---

# What is Passwordless Authentication?

Passwordless authentication allows the Ansible Control Node to connect to Managed Nodes using SSH key pairs instead of passwords.

### Components

* Private Key → Control Node
* Public Key → Managed Node

---

# Why Passwordless Authentication is Important?

Passwordless authentication enables:

* Fully automated playbook execution
* Faster server access
* Improved security
* Easier management of large environments

Without it, automation would require manual password entry.

---

# What are Ansible Ad-hoc Commands?

Ad-hoc commands are one-line Ansible commands used for quick tasks without creating playbooks.

### Common Uses

* Ping hosts
* Install packages
* Restart services
* Copy files
* Gather system information

### When to Use

* Quick tasks
* Troubleshooting
* Validation

For repeatable automation, Playbooks are preferred.

---

# What is YAML?

YAML (YAML Ain't Markup Language) is a human-readable data serialization language used for configuration and automation.

### Features

* Simple syntax
* Uses indentation
* Key-value structure
* Easy to read and maintain

Ansible playbooks are written in YAML.

---

# What is a Playbook?

A Playbook is a YAML-based automation script that defines a sequence of tasks to configure and manage systems in a predictable and repeatable manner.

### Benefits

* Reusable
* Version controlled
* Easy to maintain
* Idempotent

---

# What is a Role?

An Ansible Role is a structured and reusable way to organize automation code.

Roles separate automation into standardized components such as:

* Tasks
* Variables
* Handlers
* Templates
* Files

This improves maintainability and reusability.

---

# Role Directory Structure

## tasks/

Contains the main automation logic.

### Entry File

main.yml

### Examples

* Install packages
* Configure services

---

## handlers/

Contains tasks triggered only when notified.

### Example

Restart NGINX after configuration changes.

---

## templates/

Stores dynamic Jinja2 template files.

### Example

nginx.conf.j2

Used when configuration files need variable substitution.

---

## files/

Contains static files.

### Example

* Scripts
* Certificates
* Static configuration files

Files are copied without modification.

---

## vars/

Contains role variables with higher priority.

Used when values should not be easily overridden.

---

## defaults/

Contains default variables.

### Characteristics

* Lowest precedence
* Can be overridden by inventory or playbooks

---

## meta/

Contains role metadata.

### Example

Role dependencies.

A webserver role may depend on a common role.

---

## README.md

Role documentation.

Includes:

* Purpose
* Variables
* Usage examples
* Dependencies

---

# What is Ansible Vault?

Ansible Vault is a security feature used to encrypt sensitive information stored in Ansible files.

### Common Use Cases

* Passwords
* API Keys
* Database Credentials
* Certificates
* Tokens

### Benefits

* Secure storage of secrets
* Integration with playbooks
* Version control friendly
* Prevents accidental exposure of credentials

### simple way to understand

Ansible Vault is used to encrypt sensitive data such as passwords, API keys, and credentials inside Ansible playbooks and variable files. It helps secure secrets while allowing them to be managed alongside infrastructure code.

---

# What is Idempotency in Ansible?

Idempotency is one of the core principles of Ansible.

It means that running the same playbook multiple times will always produce the same result without making unnecessary changes if the desired state is already achieved.

### Example

If a playbook installs NGINX:

* First run → NGINX gets installed
* Second run → No changes are made because NGINX is already installed

### Benefits

* Predictable automation
* Safe repeated executions
* Reduced configuration drift
* Easier troubleshooting

### Interview Answer

Idempotency means that running an Ansible playbook multiple times produces the same result after the desired state is reached. Ansible checks the current state before making changes, ensuring tasks are performed only when required.

---

# What are Ansible Modules?

Modules are reusable units of code that Ansible executes on managed nodes to perform specific tasks.

They are the building blocks of Ansible automation.

### Common Modules

#### Package Management

* apt
* yum
* dnf

#### File Management

* copy
* file
* template

#### Service Management

* service
* systemd

#### User Management

* user
* group

#### Cloud Management

* ec2_instance
* s3_bucket

### Interview Answer

Ansible Modules are reusable programs used to perform specific tasks on managed nodes such as installing packages, managing services, copying files, creating users, and provisioning cloud resources. Playbooks use modules to execute automation tasks.

---

# Difference Between Playbook and Role

| Playbook                                  | Role                                                       |
| ----------------------------------------- | ---------------------------------------------------------- |
| YAML file containing automation tasks     | Structured collection of reusable automation components    |
| Suitable for small automation tasks       | Suitable for large and reusable projects                   |
| Can become difficult to manage when large | Improves maintainability and scalability                   |
| Contains tasks directly                   | Organizes tasks, handlers, variables, templates, and files |
| Less reusable                             | Highly reusable                                            |

### Interview Answer

A Playbook is a YAML file that defines automation tasks, while a Role is a structured way to organize and reuse those tasks. Roles help break large playbooks into modular components, making automation easier to maintain and scale.

---

# What are Handlers?

Handlers are special tasks that execute only when notified by another task.

They are commonly used to restart services after configuration changes.

### Example Use Cases

* Restart NGINX
* Restart Apache
* Reload Systemd
* Restart Application Services

### Why Use Handlers?

Without handlers:

* Services may restart unnecessarily.

With handlers:

* Services restart only when a change occurs.

### Interview Answer

Handlers are special Ansible tasks that run only when triggered by a notification from another task. They are typically used for actions such as restarting or reloading services after configuration changes.

---

# Difference Between include_tasks and import_tasks

Both are used to split playbooks into smaller files, but they behave differently.

| include_tasks                  | import_tasks                                    |
| ------------------------------ | ----------------------------------------------- |
| Dynamic inclusion              | Static inclusion                                |
| Processed at runtime           | Processed before execution                      |
| Supports conditions and loops  | Conditions apply to imported tasks individually |
| More flexible                  | Faster and predictable                          |
| Suitable for dynamic workflows | Suitable for fixed workflows                    |

### Interview Answer

`include_tasks` dynamically loads tasks during playbook execution and is useful when task inclusion depends on conditions or variables. `import_tasks` statically loads tasks before execution begins, making it more predictable and suitable for fixed task structures.

### When to Use

#### Use include_tasks

* Conditional execution
* Loops
* Dynamic task loading

#### Use import_tasks

* Fixed playbook structure
* Better readability
* Predictable execution flow

---

