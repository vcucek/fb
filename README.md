# fb (File Browser)
### Minimalistic file manager/browser written in Bash.

- vim keybindings
- easy to use bookmarks 
- file selection gets persisted between fb processes
- list view and tree view (use ~ to toggle)
- colorized output as in `ls` shell command
- sort files and folders based on modification time, name and size
- file content search with matched line displayed alongside filename
- cd on exit by adding alias to .bashrc `alias ff='fb; cd $(fb -d)'`
- execute command using marked files/folders as arguments (for example `git -C {} status` prints statuses of marked git folders)
- does not fill up or clear terminal window space (press 'F' to toggle)
- confirmation dialogs before executing copy/move/delete operations
- for other features please look at keybindings listed below

#### Installation
1. Copy fb script to /usr/local/bin
2. export `EDITOR` and `TERMINAL` environment variables to override defaults (vi, st)

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



By default the program uses xdg-open for opening files, vim for editing files and st (simple terminal) for opening new terminals.
This project has been inspired by other suckless programs, their philosophy and has been made purely for fun.
Contibutions, additional testing or bug reporting is greatly apprechiated.

```
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
 O     : Edit file using default editor or vi
 o     : Edit file using default editor or vi in a new terminal window (st by default)
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
 c  : copy marked files to clipboard as command arguments
 y  : add marked files to selection
 Y : append marked files to selection
 x  : clear selection and marked files

Selection:
 s  : show/hide current selection
 i  : Edit selection (selection mode)
 d  : remove file/folder from selection (selection mode)
 x  : clear selection and marked files

Edit:
 n  : create named file (add "/" sufix to create a folder)
 r  : rename current file or folder
 p  : copy selection to current directory
 P  : copy selection into folder under cursor
 m  : move selection to current directory
 M  : move selection into folder under cursor
 d  : delete marked files or folders

View:
 J : Increase height
 K : Decrease height
 F : Switch to full screen mode
 ~ : Switch between tree and list view
 wr: Refresh view

File sorting:
 wn: sort by name
 wm: sort by modified time
 ws: sort by size
 ```
