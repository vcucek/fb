# fb (File Browser)
### Minimalistic file manager/browser written in Bash.

- VIM like keybindings (hjkl - move, o - create file/folder, i - open file, v - mark files, y - copy, p - paste, ...).
- Does not fill up or replace terminal window space (press 'F' to toggle fullscreen). The idea is that fb should supplement the workflow in terminal, without opening additional window or clearing existing terminal content.
- SSH to a remote machine and copy files. Switch between connected hosts with TAB key.
- Bookmarks stored in a text file with file paths and optional descriptions. They can also represent remote SSH locations in form (ssh://root@hostname).
- File selection (used for copying or moving) gets persisted between fb processes.
- Toggle between list view and tree view (use ~)
- Colorized output as in `ls` shell command.
- Quick filter while typing ( press / )
- Sort files and folders based on modification time, name and size.
- File content search with matched line displayed alongside filename.
- Switch current directory on exit by adding alias to .bashrc `alias ff='fb; cd $(fb -d)'`.
- Execute command using marked files/folders as arguments (for example `git -C {} status` prints statuses of marked git folders).
- Confirmation dialogs before executing copy/move/delete operations.
- Use + key to reveal current mode actions and keybindings.
- For other features please look at keybindings listed below.

This project has been inspired by suckless programs and their philosophy. It has been made for fun and to acquire some additional knowledge about shell and its ecosystem.
Contibutions, additional testing or bug reporting is greatly apprechiated.

#### Installation
1. Copy fb script to /usr/local/bin
2. export `EDITOR` and `TERMINAL` environment variables to override defaults (vi, st)
3. Add following alias to .zshrc or .bashrc to be able to cd to a navigated folder:
```
alias ff='fb; if [ $(fb -h) ]; then ssh -t -o "ControlPath ~/.cache/fb/remotes/$(fb -h)" ssh://$(fb -h) "cd $(fb -d);\$SHELL"; else cd $(fb -d); fi'

```

#### Requirements
- Linux OS
- Bash version greater then 4.0

#### List view example
```
[vito ~/suckless]%
[vito ~/suckless]% fb
---------------------------------------------------------------------------------------
/home/vito/suckless
---------------------------------------------------------------------------------------
drwxr-xr-x 3 vito vito 4.0K Jan 23 23:05 dmenu
drwxr-xr-x 4 vito vito 4.0K Feb 15 00:09 dwm
drwxr-xr-x 3 vito vito 4.0K Feb  8 21:16 dwmstatus
drwxr-xr-x 2 vito vito 4.0K Feb  6 01:48 scripts
drwxr-xr-x 3 vito vito 4.0K Jan 23 21:11 sent
drwxr-xr-x 3 vito vito 4.0K Jan 26 19:50 st
drwxr-xr-x 3 vito vito 4.0K Jan 24 00:59 surf
drwxr-xr-x 3 vito vito 4.0K Feb  1 21:08 wmname
-rwxr-xr-x 1 vito vito  212 Jan 23 22:58 calc.sh
-rw-r--r-- 1 vito vito    0 Jan 23 22:00 dwm_loop
---------------------------------------------------------------------------------------
Directory mode: 'q' to quit, '+' to see current mode actions
```

#### Tree view example
```
[vito /]%
[vito /]% fb
---------------------------------------------------------------------------------------
/
---------------------------------------------------------------------------------------
|bin
|boot
|   |Boot
|   |EFI
|   |en-us
|   |loader
|   |System Volume Information
|   |bootmgfw.efi
|   |bootmgr
|   |bootmgr.efi
|   |initramfs-linux-fallback.img
|   |initramfs-linux.img
|   |initramfs-linux-lts-fallback.img
|   |initramfs-linux-lts.img
|   |vmlinuz-linux
|   |vmlinuz-linux-lts
|dev
|   |block
|   |bus
|   |   |usb
---------------------------------------------------------------------------------------
Directory mode: 'q' to quit, '+' to see current mode actions
```

#### How to navigate
Use 'j','k' keys to navigate through files and folders or 'C^u','C^d' to move 10 files up or down. Press 'h' or 'Backspace' to move up the filesystem hierarchy. Press `l` or `Return` to enter a folder.
When using the tree view model (press '~' to toggle), use 'l','h' keys to expand or colapse a folder content and 'Return','Backspace' to change the current directory up or down the hierarchy.
Press 'gg' or 'G' to move the cursor to fist or last file in a directory or 'L','M','H' to navigate to top, middle or bottom of the view.
By pressing 'f', followed by a character, you can quickly jump to a first file occurance stating with that character. Press '.' to repeat this action.
When searching for a specific file in the current directory, press '/' to filter the output. For searching files recursively press '?'.  It is also possible to search files based on content by pressing 'g?'.

#### How to Copy, Move files and folders
Use 'v' key to mark desired files and folders or 'V' to mark everything. After that, press 'y' to add them to the selection list. The selection list is used when executing copy or move actions. When files are stored in the selection, the user can close the application and navigate to the destination folder by other means (for exmaple executing 'cd -' in a shell to jump to a previous folder) or use the fb folder navigation commands. When fb is located in the desired destination folder you can copy or move files, stored in the selection, by pressing 'p' or 'm' keys respectively.

#### How to use bookmarks
Bookmarks provide a quick way to navigate to frequently visited locations. Press 'B' to add the current folder to bookmarks. After that you can press 'b' key to view stored bookmarks and 'l' to enter a desired folder. Bookmarks can also include remote SSH folders. You can edit stored bookmarks by pressing 'i', which opens up the VI editor.

#### Remote SSH
Majority of functions supported when browsing local machine are also supported when fb is connected to a remote SSH host.
Remote fb connection can be initiated either by pressing '@' key or opening previously bookmarked remote location. Pressing 'TAB' key switches between local machine and connected remotes.
Fb can also be used to quickly ssh to a remote host folder and perform some shell command by pressing ":".

#### Default programs
By default the program uses xdg-open for opening files, vim for editing files and st (simple terminal) for opening new terminals.

#### Shortcuts

```
Usage:
 fb [-db] [folder]
 
Options:
 -d : print last folder
 -b : show bookmarks
 
Enable cd on exit by adding alias to ~/.bashrc file:
alias ff='fb; cd $(fb -d)'
 
View commands: 
 ~   : Switch between tree and list view
 @   : Connect to SSH host
 Tab : Switch between connected SSH hosts
 J,K : Increase/decrease drawing area
 F   : Use all avaiable drawing space
 
Movements:
 Keys     : hjkl or up, down, left, right arrows
 Return   : Open file using xdg-open or enter folder
 Backspace: Go back one folder
 C^u      : Move up 10 files
 C^d      : Move down 10 files
 L,M,H    : Move low, middle, high
 f[a-z]   : Go to file name starting with
 .        : Repeat last "go to" action
 
Actions:
 q     : Exit script, mode, or dialog
 I     : Edit file using default editor or vi
 i     : Edit file using default editor or vi in a new terminal window (st by default)
 t     : open new terminal window (st by default)
 a     : Show hidden files/folders
 e     : Execute command using current or marked files
         Occurances of {} in specified command will be replaced by marked
         files and executed sequentially
 /     : Filter display (works also in selection or bookmarks mode)
 ?     : Recursive search
 g?    : Recursive content search
 :     : Open new shell in curent directory and close fb
 +     : list current mode (directory,selection,bookmarks,search) keyboard commands
 
Bookmarks:
 b  : open/close bookmarks
 i  : Edit bookmarks (bookmarks mode)
 d  : remove file/folder from bookmarks (bookmarks mode)
 B  : Add current or marked folders and files to bookmarks
 
Select:
 v  : mark current file
 V  : mark all files in current folder
 gc : copy marked files to clipboard as command arguments
 y  : set marked files to selection
 Y  : append marked files to selection
 x  : clear selection and marked files
 
Selection:
 s  : show/hide current selection
 i  : Edit selection (selection mode)
 d  : remove file/folder from selection (selection mode)
 x  : clear selection and marked files
 
Edit:
 o  : create named file (add "/" sufix to create a folder)
 c  : rename current file or folder
 p  : copy selection to current directory
 P  : copy selection into folder under cursor
 m  : move selection to current directory
 M  : move selection into folder under cursor
 d  : delete marked files or folders
 
File sorting:
 wn: sort by name
 wm: sort by modified time
 ws: sort by size
 wr: Refresh view
 ```
