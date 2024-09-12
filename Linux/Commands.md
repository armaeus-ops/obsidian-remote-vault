### 4. **File Manipulation**

   - **`cp source destination`**  
     Copies files or directories.

   - **`mv source destination`**  
     Moves or renames files or directories.

   - **`rm filename`**  
     Deletes a file.

   - **`rm -r directory_name`**  
     Deletes a directory and its contents recursively.

### 5. **Viewing File Contents**
   - **`cat filename`**  
     Displays the contents of a file.

   - **`less filename`**  
     Views the contents of a file one page at a time (useful for larger files).

   - **`head filename`**  
     Shows the first 10 lines of a file.

   - **`tail filename`**  
     Shows the last 10 lines of a file.

   - **`tail -f filename`**  
     Continuously monitors a file for new content (useful for log files).

### 6. **Search for Files**
   - **`find /path -name "filename"`**  
     Searches for files or directories starting from the specified path.

   - **`locate filename`**  
     Quickly finds the location of a file by searching a prebuilt database (faster than `find` but might not be up-to-date).

### 7. **Search Within Files**
   - **`grep "pattern" filename`**  
     Searches for a specific pattern or text within a file.

   - **`grep -r "pattern" /path`**  
     Recursively searches for a pattern in all files under the specified directory.

### 8. **Permissions Management**
   - **`chmod permissions filename`**  
     Changes the permissions of a file or directory.

   - **`chown user:group filename`**  
     Changes the owner and group of a file or directory.

### 9. **File Compression and Archiving**
   - **`tar -cvf archive.tar directory`**  
     Creates an archive file from a directory.

   - **`tar -xvf archive.tar`**  
     Extracts files from an archive.

   - **`gzip filename`**  
     Compresses a file using gzip.

   - **`gunzip filename.gz`**  
     Decompresses a gzip file.

### 10. **Disk Usage**
   - **`df -h`**  
     Shows the disk space usage of mounted filesystems in a human-readable format.

   - **`du -h`**  
     Shows the disk space usage of files and directories in a human-readable format.

### 11. **Process Management**
   - **`ps aux`**  
     Displays a list of all running processes.

   - **`top`**  
     Shows an interactive view of running processes with resource usage.

   - **`kill PID`**  
     Terminates a process by its Process ID (PID).

### 12. **Networking**
   - **`ping hostname`**  
     Checks the network connectivity to a host.

   - **`ifconfig`**  
     Displays or configures network interfaces (use `ip a` in newer distributions).

   - **`ssh user@hostname`**  
     Connects to a remote machine via SSH.

### 13. **Get System Information**
   - **`uname -a`**  
     Displays information about the system, kernel, and architecture.

   - **`uptime`**  
     Shows how long the system has been running, along with load averages.

   - **`whoami`**  
     Displays the current user.

### 14. **Exit Terminal**
   - **`exit`**  
     Closes the terminal or ends the session.


## 15. Keyboardlayout
`loadkeys <xx>`  - keyboard layout einmal ändern (de für Deutsch)
`sudo localectl set-keymap <xx>` - keyboardlayout permanent ändern
