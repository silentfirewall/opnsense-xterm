# Terminal Plugin for OPNsense Firewall

---

## 1. Purpose
Develop a terminal plugin for OPNsense to enable secure, browser-based access to SSH, serial consoles, and direct shell sessions within the firewall's web interface, designed for scalability and targeting network administrators and advanced users.

---

## 2. Features
1. **Terminal Emulator**
   - Support for SSH, serial (RS232), and direct shell access with session persistence across reconnects.
   - Tabbed interface for managing multiple sessions.
   - Copy/paste functionality and terminal theming.

2. **SSH Integration**
   - Connect to remote servers via SSH (password/key-based auth, tunneling support).
   - Save/load connection profiles (host, port, credentials).

3. **Serial Console**
   - Detect local serial devices (e.g., `/dev/cuau0` on FreeBSD).
   - Configure baud rate, parity, and flow control.

4. **Direct Shell Access**
   - Execute OPNsense CLI commands (e.g., `pluginctl`, `configctl`).
   - Restrict access based on user roles (e.g., "root" vs "admin").

5. **Quick Commands**
   - Provide pre-defined, frequently used CLI commands (e.g., `ifconfig`, `pfctl`) via a dropdown or hotkeys.

6. **Security**
   - Role-based access control (RBAC) via OPNsense permissions.
   - Session encryption (AES-256), inactivity timeouts, and emergency session termination.
   - Input validation/sanitization to prevent command injection.

7. **Audit Logging**
   - Log terminal activity (commands, connections) to `/var/log/terminal_plugin.log`.
   - Integrate with OPNsense’s Syslog/Reporting system.

---

## 3. Technical Requirements
- **Platform**: OPNsense 23.7+ (FreeBSD 13.2+).
- **Languages**: PHP (MVC), JavaScript (xterm.js), Shell.
- **Dependencies**:
  - PHP: `ssh2` (>= 1.3), `posix`, `pcntl` extensions; fallback to `exec()` if `ssh2` unavailable.
  - JavaScript: `xterm.js` (>= 5.0), `xterm-addon-fit`, `xterm-addon-web-links`.
  - FreeBSD: `comms/putty` (for serial support).
- **Resource Limits**: Configurable max CPU/memory per session (e.g., 10% CPU, 512MB RAM).

---

## 4. Development Steps
1. **Environment Setup**
   - Install OPNsense dev tools (`os-devel`) and configure a local OPNsense VM for testing.
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
   - Use PHP’s `ssh2_connect()` with error handling:
     ```php
     $connection = ssh2_connect('host', 22);
     if (!$connection || !ssh2_auth_pubkey_file($connection, 'user', '/path/to/pub', '/path/to/priv')) {
         throw new Exception("SSH connection failed");
     }
     ```

4. **Serial Console**
   - Use `proc_open()` for secure serial access:
     ```php
     $descriptor = ['pipe', 'r', 'pipe', 'w', 'pipe', 'w'];
     $process = proc_open("cu -l /dev/cuau0 -s 9600", $descriptor, $pipes);
     ```

5. **UI Integration**
   - Embed xterm.js with accessibility:
     ```javascript
     const term = new Terminal({ screenReaderMode: true });
     term.open(document.getElementById('terminal-container'));
     term.element.setAttribute('aria-label', 'Terminal Emulator');
     ```

6. **Security Implementation**
   - Validate user roles via OPNsense’s `AuthGroup` class.
   - Sanitize inputs using `escapeshellarg()`.
   - Sandbox shell processes for added isolation.

---

## 5. Testing Plan
1. **Unit Tests**
   - Validate SSH/serial connection handling (e.g., malformed keys, missing devices).
   - Test shell command sanitization.

2. **Integration Tests**
   - Verify UI responsiveness with 50+ concurrent sessions.
   - Test RBAC policies (e.g., deny "user" role root access).

3. **Security Audit**
   - Penetration testing and fuzzing for command injection vulnerabilities.
   - Validate session encryption (TLS 1.3+, AES-256).

---

## 6. Deployment
1. **Packaging**
   - Build plugin using OPNsense’s `pkg-create` tool and test against stable/rolling releases.
2. **Documentation**
   - Write a user guide with setup, troubleshooting, and FAQ sections (e.g., "SSH key not working").
3. **Community Submission**
   - Submit to OPNsense’s GitHub repo under `plugins/terminal`.

---

## 7. Risks & Mitigation
- **Risk**: SSH library incompatibility.
  - **Mitigation**: Test with multiple PHP versions (e.g., 7.4, 8.1).
- **Risk**: Privilege escalation via shell access.
  - **Mitigation**: Restrict commands via `sudoers` and sandbox shell processes.
- **Risk**: Resource exhaustion from too many sessions.
  - **Mitigation**: Implement a configurable session limit (e.g., max 20 sessions per user).

---

## 8. Timeline
- **Phase 1: Core Functionality (4 weeks)**
  - Week 1: Environment setup and MVC structure (1 week).
  - Weeks 2-3: SSH/serial implementation (2 weeks).
  - Week 4: Basic UI integration (1 week).
- **Phase 2: UI Polish (2 weeks)**
  - Accessibility enhancements and theming (2 weeks).
- **Phase 3: Testing/Deployment (2 weeks)**
  - Week 1: Unit/integration tests (1 week).
  - Week 2: Security audit and deployment (1 week).

---

## Approvals
- **Lead Developer**: [Signature]
- **Security Team**: [Signature]
- **OPNsense Maintainers**: [Signature]

---

### Summary of Changes
- **Purpose**: Added scalability and audience focus.
- **Features**: Added session persistence, SSH tunneling, quick commands, and enhanced security specifics (AES-256, kill switch).
- **Technical Requirements**: Specified dependency versions, fallback options, and resource limits.
- **Development Steps**: Improved error handling (SSH), replaced `shell_exec()` with `proc_open()` (serial), added accessibility (UI).
- **Testing**: Expanded scope with edge cases, scalability tests (50+ sessions), and fuzzing.
- **Deployment**: Added compatibility testing and FAQ in documentation.
- **Risks**: Included resource exhaustion risk with mitigation.
- **Timeline**: Broke down phases into detailed tasks.

This revised RPD provides a more robust, secure, and user-friendly plugin while maintaining alignment with OPNsense’s standards.
