# Privilege Escalation

- Privilege escalation allows a non-root user to perform tasks requiring superuser rights. 
- Instead of enabling direct root logins—which poses security risks—you can delegate specific commands to trusted users via `sudo`. 
- This approach enforces the principle of least privilege and keeps your system secure.

## SUDO
- In Linux, sudo (short for "superuser do") is a command that allows users to execute commands with elevated privileges, typically as the root user.

### How Does sudo Work?
1. Privileges:
- When a user runs a command with sudo, the system temporarily elevates their privileges, allowing them to execute commands that require administrative rights.
2. Password Prompt:
- By default, sudo requires the user's password (not the root password) to confirm their identity and authorize the command.
3. Time-limited Access:
- Once a user successfully enters their password, they won’t need to re-enter it for subsequent sudo commands within a default timeout period (typically 15 minutes).

### The `sudoers` File
- The `sudoers` file controls who can use sudo and what commands they are allowed to execute.
    - Location: `/etc/sudoers`
    - Editing Safely: Use `visudo` to edit the `sudoers` file to prevent syntax errors.
    ```bash
    sudo visudo
    ```
- Here’s a sample excerpt:
    ```bash
    # User privilege specification
    root    ALL=(ALL:ALL) ALL

    # Members of the admin group may gain root privileges
    %admin  ALL=(ALL)       ALL

    # Allow members of group sudo to execute any command
    %sudo   ALL=(ALL:ALL)   ALL

    # Allow mark to run any command
    mark    ALL=(ALL:ALL)   ALL

    # Allow sarah to reboot the system
    sarah   localhost=/usr/bin/shutdown -r now

    # See sudoers(5) for more information on "#include" directives:
    #includedir /etc/sudoers.d
    ```

| Field                |Description                                              | Example | 
|----------------------|---------------------------------------------------------|---------------|
| User or Group        | Username (e.g., `mark`) or group (`%sudo`)              | `%admin`|
| Host(s)              | Hosts where the rule applies (usually `ALL`)            | `localhost` |
| Run-As Specification | User and group for command execution (in `(` and `)`)   | (ALL:ALL) |
| Commands             | Allowed commands or `ALL` for full rights                 | `/usr/bin/shutdown -r now` |
| Comments             | Lines beginning with `#` are ignored	                     | `# User privilege specification` | 

## Best Practices for sudo Configuration
- Grant only the commands necessary for a task
- Use group-based rules to simplify management
- Avoid `NOPASSWD` unless automation requires it
- Keep custom rules in `/etc/sudoers.d/` for modularity
