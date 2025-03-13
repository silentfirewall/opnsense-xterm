### **Rapid Plugin Development (RPD) Document: Terminal Plugin for OPNsense Firewall**

---

#### **1. Purpose**  
Develop a terminal plugin for OPNsense to enable secure, browser-based access to SSH, serial consoles, and direct shell sessions within the firewall's web interface.

---

#### **2. Features**  
1. **Terminal Emulator**  
   - Support for SSH, serial (RS232), and direct shell access.  
   - Tabbed interface for managing multiple sessions.  
   - Copy/paste functionality and terminal theming.  

2. **SSH Integration**  
   - Connect to remote servers via SSH (password/key-based auth).  
   - Save/load connection profiles (host, port, credentials).  

3. **Serial Console**  
   - Detect local serial devices (e.g., `/dev/cuau0` on FreeBSD).  
   - Configure baud rate, parity, and flow control.  

4. **Direct Shell Access**  
   - Execute OPNsense CLI commands (e.g., `pluginctl`, `configctl`).  
   - Restrict access based on user roles (e.g., "root" vs "admin").  

5. **Security**  
   - Role-based access control (RBAC) via OPNsense permissions.  
   - Session encryption and inactivity timeouts.  
   - Input validation/sanitization to prevent command injection.  

6. **Audit Logging**  
   - Log terminal activity (commands, connections) to `/var/log/terminal_plugin.log`.  
   - Integrate with OPNsense’s Syslog/Reporting system.  

---

#### **3. Technical Requirements**  
- **Platform**: OPNsense 23.7+ (FreeBSD 13.2+).  
- **Languages**: PHP (MVC), JavaScript (xterm.js), Shell.  
- **Dependencies**:  
  - PHP: `ssh2`, `posix`, `pcntl` extensions.  
  - JavaScript: `xterm.js`, `xterm-addon-fit`, `xterm-addon-web-links`.  
  - FreeBSD: `comms/putty` (for serial support).  

---

#### **4. Development Steps**  
1. **Environment Setup**  
   - Install OPNsense dev tools (`os-devel`).  
   - Clone OPNsense plugin template:  
     ```bash  
     git clone https://github.com/opnsense/plugin-template.git os-terminal  
     ```

2. **MVC Structure**  
   - **Model**:  
     - `TerminalConnection` (handles SSH/serial configs).  
     - `ShellCommand` (sanitizes/executes CLI commands).  
   - **View**:  
     - `terminal.volt` (HTML/CSS for xterm.js integration).  
   - **Controller**:  
     - `TerminalController` (manages sessions and permissions).  

3. **SSH Client Implementation**  
   - Use PHP’s `ssh2_connect()` for SSH sessions.  
   - Example code:  
     ```php  
     $connection = ssh2_connect('host', 22);  
     ssh2_auth_pubkey_file($connection, 'user', '/path/to/pub', '/path/to/priv');  
     ```

4. **Serial Console**  
   - Use FreeBSD’s `cu` command for serial access:  
     ```php  
     shell_exec("cu -l /dev/cuau0 -s 9600");  
     ```

5. **UI Integration**  
   - Embed xterm.js in OPNsense’s UI:  
     ```javascript  
     const term = new Terminal();  
     term.open(document.getElementById('terminal-container'));  
     ```

6. **Security Implementation**  
   - Validate user roles via OPNsense’s `AuthGroup` class.  
   - Sanitize inputs using `escapeshellarg()`.  

---

#### **5. Testing Plan**  
1. **Unit Tests**  
   - Validate SSH/serial connection handling.  
   - Test shell command sanitization.  

2. **Integration Tests**  
   - Verify UI responsiveness with 10+ concurrent sessions.  
   - Test RBAC policies (e.g., deny "user" role root access).  

3. **Security Audit**  
   - Penetration testing for command injection vulnerabilities.  
   - Validate session encryption (TLS 1.3+).  

---

#### **6. Deployment**  
1. **Packaging**  
   - Build plugin using OPNsense’s `pkg-create` tool.  
2. **Documentation**  
   - Write a user guide for connection setup and troubleshooting.  
3. **Community Submission**  
   - Submit to OPNsense’s GitHub repo under `plugins/terminal`.  

---

#### **7. Risks & Mitigation**  
- **Risk**: SSH library incompatibility.  
  **Mitigation**: Test with multiple PHP versions.  
- **Risk**: Privilege escalation via shell access.  
  **Mitigation**: Restrict commands via `sudoers` policies.  

---

#### **8. Timeline**  
- **Phase 1**: Core functionality (4 weeks).  
- **Phase 2**: UI polish (2 weeks).  
- **Phase 3**: Testing/deployment (2 weeks).  

---

**Approvals**:  
- Lead Developer: [Signature]  
- Security Team: [Signature]  
- OPNsense Maintainers: [Signature]  

--- 

This RPD ensures alignment with OPNsense’s architecture and security standards while delivering a user-friendly terminal experience.
