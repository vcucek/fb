# fb (File Browser)
### Minimalistic file manager/browser written in Bash.

- vim keybindings
- easy to use bookmarks 
- file selection gets persisted between fb processes
- cd on exit by adding alias to .bashrc `alias ff='fb; cd $(fb -d)'`
- execute command using marked files/folders as arguments
- does not fill up or clear terminal window space
- confirmation dialogs before executing copy/move/delete operations
- for other features please look at keybindings listed below

By default the program uses xdg-open for opening files, vim for editing files and st (simple terminal) for opening new terminals.
This project has been inspired by other suckless programs, their philosophy and has been made purely for fun.
Contibutions, additional testing or bug reporting is greatly apprechiated.

```
Movements:
 Keys: hjkl or up, down, left, right arrows 
 C^u: Move up 10 files
 C^d: Move down 10 files
 L,M,H : Move low, middle, high
 f[a-z]: Go to file name starting with
 .     : Repeat last "go to" action

Actions:
 q     : Exit script, mode, or dialog
 o     : Edit file using default editor or vi
 O     : Edit file using default editor or vi in a new terminal window
 t     : open new terminal window
 a     : Show hidden files/folders
 e     : Execute command using current file/folder or marked files
 /     : Filter
 ?     : Find
 :     : Run shell command

Bookmarks:
 b  : open/close bookmarks
 B  : bookmark current folder

Select:
 v  : mark current file
 V  : mark all files in current folder
 C  : copy marked files to clipboard as command arguments
 c  : copy marked files to clipboard (work in progress)
 y  : add marked files to selection
 yy : add current file to selection
 x  : clear selection and marked files

Edit:
 n  : create file or folder (to create folder end it with "/")
 r  : rename current file or folder
 p  : copy selection to current directory
 m  : move selection to current directory
 d  : delete marked files or folders

View:
 J: Increase height
 K: Decrease height
 F: Switch to full screen mode
 ```
