#!/bin/bash
# Author: Vito Cucek (vito.cucek@gmail.com)

# Print help
if [[ $1 = "--help" ]]; then
    echo 'File browser'
    echo 'Author: Vito Cucek'
    echo ' '
    echo 'Usage:'
    echo ' fb [-dbh] [folder]'
    echo ' '
    echo 'Options:'
    echo ' -h : print last ssh destination'
    echo ' -d : print last folder'
    echo ' -b : show bookmarks'
    echo ' '
    echo 'Enable cd on exit by adding alias to ~/.bashrc file:'
    echo 'alias ff='"'"'fb; cd $(fb -d)'"'"
    echo ' '
    echo 'View commands: '
    echo ' ~   : Switch between tree and list view'
    echo ' @   : Connect to SSH host'
    echo ' Tab : Switch between connected SSH hosts'
    echo ' J,K : Increase/decrease drawing area'
    echo ' F   : Use all avaiable drawing space'
    echo ' '
    echo 'Movements:'
    echo ' Keys     : 'hjkl' or up, down, left, right arrows'
    echo ' Return   : Open file using xdg-open or enter folder'
    echo ' Backspace: Go back one folder'
    echo ' C^u      : Move up 10 files'
    echo ' C^d      : Move down 10 files'
    echo ' L,M,H    : Move low, middle, high'
    echo ' f[a-z]   : Go to file name starting with'
    echo ' .        : Repeat last "go to" action'
    echo ' '
    echo 'Actions:'
    echo ' q     : Exit script, mode, or dialog'
    echo ' I     : Edit file using default editor or vi'
    echo ' i     : Edit file using default editor or vi in a new terminal window (st by default)'
    echo ' t     : open new terminal window (st by default)'
    echo ' a     : Show hidden files/folders'
    echo ' e     : Execute command using current or marked files'
    echo '         Occurances of {} in specified command will be replaced by marked'
    echo '         files and executed sequentially'
    echo ' /     : Filter display while typing (works also in selection or bookmarks mode)'
    echo ' ?     : Recursive search'
    echo ' g?    : Recursive content search'
    echo ' :     : Open new shell in curent directory and close fb'
    echo ' +     : list current mode (directory,selection,bookmarks,search) keyboard commands'
    echo ' '
    echo 'Bookmarks:'
    echo ' b  : open/close bookmarks'
    echo ' i  : Edit bookmarks (bookmarks mode)'
    echo ' d  : remove file/folder from bookmarks (bookmarks mode)'
    echo ' B  : Add current or marked folders and files to bookmarks'
    echo ' '
    echo 'Select:'
    echo ' v  : mark current file'
    echo ' V  : mark all files in current folder'
    echo ' gc : copy marked files to clipboard as command arguments'
    echo ' y  : set marked files to selection'
    echo ' Y  : append marked files to selection'
    echo ' x  : clear selection and marked files'
    echo ' '
    echo 'Selection:'
    echo ' s  : show/hide current selection'
    echo ' i  : Edit selection (selection mode)'
    echo ' d  : remove file/folder from selection (selection mode)'
    echo ' x  : clear selection and marked files'
    echo ' '
    echo 'Edit:'
    echo ' o  : create named file (add "/" sufix to create a folder)'
    echo ' c  : rename current file or folder'
    echo ' p  : copy selection to current directory'
    echo ' P  : copy selection into folder under cursor'
    echo ' m  : move selection to current directory'
    echo ' M  : move selection into folder under cursor'
    echo ' d  : delete marked files or folders'
    echo ' '
    echo 'File sorting:'
    echo ' wn: sort by name'
    echo ' wm: sort by modified time'
    echo ' ws: sort by size'
    echo ' wr: Refresh view'
    exit 0
fi

## Terminal settings ###########################################################

SAVEIFS=""
function terminal_setup {
    # set array separator
    SAVEIFS=$IFS
    FB_IFS=$(echo -en "\n\b")
    IFS=$FB_IFS
    # disable line wrapping
    printf '\e[?7l'
    # hide cursor
    printf '\e[?25l'

    # do not show user input
    stty -echo
}

function terminal_fetch {
    IFS=$' '
    read -r LINES COLUMNS < <(stty size)
    IFS=$FB_IFS
}

function terminal_restore {
    # restore line wrapping
    printf '\e[?7h'
    # show cursor
    printf '\e[?25h'
    # restore array separator
    IFS=$SAVEIFS
    # show user input
    stty echo
}

## Constants ###################################################################
APP_TERM=${TERMINAL:-st}
APP_EDITOR=${EDITOR:-vi}

FB_PATH="$0"
FB_PID="$$"

# bottom/top padding when scrolling
PADDING=4
# Variable representing visible file system area height
HEIGHT=20
# Start in full screen mode
FULLSCREEN=false

# cache files
config_dir=${HOME}/.config/fb
config_bookmarks=$config_dir/bookmarks

cache_dir=${HOME}/.cache/fb
cache_dir_remotes=$cache_dir/remotes
cache_selection=$cache_dir/selection
cache_cd=$cache_dir/last_dir
cache_host=$cache_dir/last_host

# colors
col_cursor=$'\e[1;31m'
col_select_bg=$'\e[1;103m'
col_select_fg=$'\e[1;30m'
col_nav=$'\e[1;32m'

col_link=$'\e[1;96m'
col_exec=$'\e[1;32m'
col_socket=$'\e[1;95m'
col_char=$'\e[1;93m'
col_folder=$'\e[1;94m'
col_none=$'\e[0m'

## Parameters ##################################################################

show_hidden=false

# Bottom bar text
status_message=""
notification_message=""

# dir,search,bookmarks,selection,marks,action
mode="dir"
# tree,flat
dir_mode="flat"

# copy,move,delete
action=""

# Draw context variables
fullscreen_mode=$FULLSCREEN

# Variable representing cursor file index
dc_cursor=0
# Variable representing top visible file index
dc_begin=0
# Variable representing number of visible files
dc_height=$HEIGHT
# lines to draw
dc_lines=()

# Current folder variables
files=()
current_file=
current_dir=
current_pwd=

# name, access, modified, size
sort_by="name"

# files attributes
declare -A files_expanded
declare -A files_marked

last_key=""

marks=()

current_host=0
hosts=("")
hosts_pwd=("$PWD")

## General Functions ###########################################################

function _read_fb {
    # show user input
    stty echo
    # show cursor
    printf '\e[?25h'
    read $1
    # move one line up
    printf "\e[1A"
    # hide cursor
    printf '\e[?25l'
    # do not show user input
    stty -echo
}

function _textinput {

    # show user input
    stty echo
    # show cursor
    printf '\e[?25h'

    key=
    text=

    # move one line down and clear
    printf "\033[1B"
    printf "\033[2K"
    # Draw
    printf "$1$text"

    while :
    do
        # read key
        k1=
        k2=
        k3=
        read -sn1 k1
        if [[ $k1 == $'\e' || $k1 == $'\x' ]]; then
            read -sn1 -t 0.0001 k2
            read -sn1 -t 0.0001 k3
        fi
        key=${k1}${k2}${k3}

        # Empty text and Return 
        if [[ $text == '' && $key = '' ]]; then
            key="consumed"
            # Call terminate action
            eval " $3"
            break
        # Return 
        elif [[ $key = '' ]]; then
            key="consumed"
            # Call confirm action
            eval " $2"
            break
        # Stop 
        elif [[ $key = 'q' || $key = $'\x1b' ]]; then
            key="consumed"
            # Call terminate action
            eval " $3"
            break
        # Backspace 
        elif [[ $text != '' && $key = $'\x7f' ]]; then
            text=${text:0:-1}
        elif [[ $k1 != $'\e' && $k1 != $'\x' ]]; then
            text=$text$key
        fi

        key="consumed"

        eval " $4"

        dc_cursor=0
        dc_lines=()

        # move one line up to beginning
        printf "\e[2000D"
        printf "\e[1A"

        render

        # move one line down, clear and print
        printf "\033[1B"
        printf "\033[2K"
        # Draw
        printf "$1$text"
    done

    # move one line up to beginning
    printf "\e[2000D"
    printf "\e[1A"

    # hide cursor
    printf '\e[?25l'
    # do not show user input
    stty -echo
}

function _cd {
    local host="${hosts[current_host]}"
    if [ "$host" = "" ]; then
        command cd "$@" || return 1
        hosts_pwd[current_host]=$PWD
        current_pwd="${hosts_pwd[current_host]}"
    else
        current_pwd="${hosts_pwd[current_host]}"
        hosts_pwd[current_host]=$(_execute_remote "cd \"$@\"; pwd")
        current_pwd="${hosts_pwd[current_host]}"
    fi
    [ ${current_pwd: -1} != '/' ] && current_pwd="$current_pwd/"
    return 0
}

function _copy {
    local src=$(echo "$1" | sed 's/^ssh/scp/')
    local trg=$(echo "$2" | sed 's/^ssh/scp/')
    src=$(printf "%q" "$src")
    trg=$(printf "%q" "$trg")
    #echo "Copy source: $src, to target: $trg"
    scp -r -3 -o "$ssh_master" -o "$ssh_path" -o "$ssh_persist" $src $trg
}

function _delete {
    if [[ "$1" =~ "^ssh:" ]]; then
        local host=$(echo "$1" | sed -n 's|ssh://\([^/]*\).*|\1|p')
        local path=$(echo "$1" | sed -n 's|ssh://[^/]*/\(.*\)|\1|p')
        #echo "Removing remote file: $path, Host: $host"
        _ssh_host $host printf "%s" "$path"
    else
        #echo "Removing local file: $1"
        rm -rf "$1"
    fi
}

function _move {
    _copy "$1" "$2" && _delete "$1"
}

function _execute {
    local host="${hosts[current_host]}"
    if [ "$host" = "" ]; then
        eval "$@"
    else
        _execute_remote "$@"
    fi
}

## SSH helper functions and parameters #########################################

#ssh_path="ControlPath $cache_dir_remotes/%r@%h:%p"
ssh_path="ControlPath $cache_dir_remotes/%r@%h"
ssh_master="ControlMaster=auto"
ssh_persist="ControlPersist 1h"

function _ssh_host {
    local host="${hosts[current_host]}"
    ssh -o "$ssh_master" -o "$ssh_path" -o "$ssh_persist" "ssh://$1" "${@:2}"
}

function _ssh_current_host {
    local host="${hosts[current_host]}"
    ssh -o "$ssh_master" -o "$ssh_path" -o "$ssh_persist" "ssh://$host" "$@"
}

function _execute_remote {
    _ssh_current_host "cd \"$current_pwd\"; $@"
}

function _execute_remote_pty {
    _ssh_current_host -t "cd \"$current_pwd\"; $@"
}

function _connect_host {
    local host_id="$1"
    local host="$2"
    ssh -fN -o "$ssh_master" -o "$ssh_path" -o "$ssh_persist" "ssh://$host" || return 1
    local home_dir=$(ssh -o "$ssh_master" -o "$ssh_path" -o "$ssh_persist" "ssh://$host" "pwd")

    # Directories must always end with /
    [ ${home_dir: -1} != '/' ] && home_dir="$home_dir/"

    hosts[host_id]="$host"
    hosts_pwd[host_id]="$home_dir"
    return 0
}

## Cursor functions ############################################################

function _cursor_to_file {
    for (( i=1; i<${#files[@]}; i++ )); do
        if [[ "${files[i]}" == "$1" ]]; then
            dc_cursor=$i
            return;
        fi
    done
}

## Bookmarks functions #########################################################

function _bookmark_add {
    local bookmark=$1
    local host="${hosts[current_host]}"
    if [ "$host" != "" ]; then
        bookmark="ssh://$host$1"
    fi
    echo "$(echo $bookmark | cat - $config_bookmarks)" > $config_bookmarks
}

function _bookmark_remove {
    unset files[$dc_cursor]
    files=("${files[@]}")
    sed -i "$((dc_cursor + 1))d" $config_bookmarks
}

## Marks functions #############################################################

function _marks_add {
    if [ -n "$current_file" ]; then
        files_marked[$current_file]=true
    fi
}
function _marks_add_all {
    if [ -n "$current_dir" ]; then
        render_loading
        for f in $current_dir/*; do
            files_marked["$f"]=true
        done
    fi
}

function _marks_toggle {
    # Check if selected
    if [ -n "$current_file" ]; then
        if [ ! -z "${files_marked[$current_file]}" ]; then
            files_marked[$current_file]=
        else
            files_marked[$current_file]=true
        fi
    fi
}

function _marks_clear {
    files_marked=()
}

function _marks_args {
    printf "\'%s\' " "${!files_marked[@]}"
}

function _marks_to_selection {
    if [ ${#files_marked[@]} -ne 0 ]; then
        local host="${hosts[current_host]}"
        if [ -z "$host" ]; then
            printf "%s\n" "${!files_marked[@]}" > $cache_selection
        else
            printf "ssh://$host/%s\n" "${!files_marked[@]}" > $cache_selection
        fi
        sed -i "s/ *#.*//g; /^$/d" $cache_selection
    fi
}
function _marks_to_selection_append {
    if [ ${#files_marked[@]} -ne 0 ]; then
        local host="${hosts[current_host]}"
        if [ -z "$host" ]; then
            printf "%s\n" "${!files_marked[@]}" >> $cache_selection
        else
            #printf "\"$host:'%s'\"\n" "${!files_marked[@]}" >> $cache_selection
            printf "ssh://$host/%s\n" "${!files_marked[@]}" > $cache_selection
        fi
        sed -i "s/ *#.*//g; /^$/d" $cache_selection
    fi
}

## Selection functions #########################################################

function _selection_remove {
    sed -i "$((dc_cursor + 1))d" $cache_selection
    files=($(cat $cache_selection))
}

function _selection_clear {
    > $cache_selection
}

## Setup Content Functions #####################################################

function set_action {
    mode="action"
    action=$1
    status_message=$2
}

function set_bookmarks {
    dc_cursor=0
    dc_lines=()
    # replace empty lines with ~ to make files array indexes consistent with file lines
    files=($(cat $config_bookmarks | sed "s/^$/~/"))
    mode="bookmarks"
    status_message="Stored bookmarks: 'Esc' to quit, '+' to see current mode actions"
}

function set_marks {
    if [ ${#files_marked[@]} -ne 0 ]; then
        dc_cursor=0
        dc_lines=()
        files=("${!files_marked[@]}")
        mode="marks"
        status_message="Marked files: 'Esc' to quit"
    fi
}

function set_selection {
    dc_cursor=0
    dc_lines=()
    files=($(cat $cache_selection))
    mode="selection"
    status_message="Selection: 'Esc' to quit, '+' to see current mode actions"
}

function set_filter {
    render_loading
    dc_cursor=0
    dc_lines=()
    files=($(printf "${files[*]}" | grep "$1"))
    mode="search"
    status_message="Filter results: 'Esc' to quit, '+' to see current mode actions"
}

function set_search {
    render_loading
    found=($(_execute "find $current_pwd -iname \"*$1*\""))
    dc_cursor=0
    dc_lines=()
    files=("${found[@]}")
    mode="search"
    status_message="Search results: 'Esc' to quit, '+' to see current mode actions"
}

function set_search_content {
    render_loading
    #TODO check multiple words
    found=($(_execute grep -Isr "$1"))
    dc_cursor=0
    dc_lines=($(printf "%s\n" "${found[@]}"))
    files=($(printf "%s\n" "${found[@]}" | sed "s/:.*//g"))
    mode="search"
    status_message="Search results: 'Esc' to quit, '+' to see current mode actions"
}

function set_host {
    for (( i=1; i<${#hosts[@]}; i++ )); do
        if [[ "${hosts[i]}" =~ "$1.*" ]]; then
            if _connect_host $i "${hosts[i]}"; then
                current_host=$i
                set_dir $2
                return 0
            else
                return 1
            fi
        fi
    done
    local host_index=${#hosts[@]}
    if _connect_host $host_index $1; then
        current_host=$host_index
        set_dir $2
    fi
}


function set_dir {
    local old_pwd=$current_pwd
    if ! _cd ${1:-""} > /dev/null 2>&1; then
        return 1
    fi
    
    dc_cursor=0
    current_dir=$current_pwd
    set_dir_refresh
    mode="dir"
    status_message="Directory mode: 'q' to quit, '+' to see current mode actions"
    _marks_clear

    if [[ "$old_pwd" != "$current_pwd" ]]; then
        _cursor_to_file $old_pwd
    elif [ -n "$current_file" ]; then
        _cursor_to_file $current_file
    fi
}

function set_dir_refresh {
    files=()
    dc_lines=()
    if [[ $dir_mode = "flat" ]]; then
        do_set_dir $current_pwd
    fi
    if [[ $dir_mode = "tree" ]]; then
        dir_level=0
        do_set_tree $current_pwd
    fi
}

function do_set_dir {
    local ls_arg="-Np -1 --group-directories-first"
    if [ $sort_by = "name" ]; then
        ls_arg=$ls_arg
    elif [ $sort_by = "size" ]; then
        ls_arg="$ls_arg -S"
    elif [ $sort_by = "modified" ]; then
        ls_arg="$ls_arg -tc"
    fi
    if $show_hidden; then
        ls_arg="$ls_arg -A"
    fi

    local pwd_esc="${1//\//\\/}"
    find_files="ls $ls_arg | sed 's/^/$pwd_esc/'"
    find_files_dc="ls -hl $ls_arg | sed 's/total.*//'"
    files=($(_execute "$find_files"))
    dc_lines=($(_execute "$find_files_dc"))
}

function do_set_tree {

    local dir_list=()
    local file_list=()

    local sort_cmd=
    local find_print_arg=

    if [ $sort_by = "name" ]; then
        find_print_arg="-printf '%p\n'"
        sort_cmd="|sort -ifd"
    elif [ $sort_by = "size" ]; then
        find_print_arg="-printf '%s\t%p\n'"
        sort_cmd="|sort -gr"
    elif [ $sort_by = "modified" ]; then
        find_print_arg="-printf '%T+\t%p\n'"
        sort_cmd="|sort -gr"
    fi

    grep_cmd=
    if ! $show_hidden; then
        grep_cmd="|grep -v '/\.'"
    fi

    find_dir="find -L \"$1\" -mindepth 1 -maxdepth 1 -type d $find_print_arg \
        $grep_cmd \
        $sort_cmd \
        | cut -f 2 \
        | sed 's:\$:/:'"
    find_file="find -L \"$1\" -mindepth 1 -maxdepth 1 -not -type d $find_print_arg \
        $grep_cmd \
        $sort_cmd \
        | cut -f 2"

    dir_list=($(_execute "$find_dir"))
    file_list=($(_execute "$find_file"))

    local prefix=$(printf "|%*s" $(( $dir_level )) )
    prefix=${prefix// /"   |"}

    # sort directories first
    local i=0
    for ((i=0; i<${#dir_list[*]}; i++)); do
        d="${dir_list[i]}"
        files+=($d)
        dc_lines+=($prefix${d##$1})
        is_expanded=${files_expanded["$d"]}
        if [ ! -z ${is_expanded:+x} ]; then
            ((dir_level++))
            do_set_tree ${dir_list[i]}
            ((dir_level--))
        fi
    done
    for ((i=0; i<${#file_list[*]}; i++)); do
        f="${file_list[i]}"
        files+=($f)
        dc_lines+=($prefix${f##*/})
    done
}

## Rendering Functions #########################################################

function row_nav {
    # store cursor position
    printf "\033[s"
    # move one line down, clear and print
    printf "\033[1B"
    printf "\033[2K"
    printf "%s" "$3$2$1$col_none"
}

function row_nav_end {
    # restore cursor position
    printf "\033[u"
    # set dc_cursor to beginning
    # printf "\e[1A"
    # printf "\e[200D"
}

function row {
    printf "$3$2%s$col_none\033[K\n" "$1"
    ((dc_canvas_height++))
}

function render {

    dc_canvas_height=0

    # Clamp rows to terminal height, specified as LINES
    if $fullscreen_mode; then
        dc_height=10000
    fi
    local files_size=${#files[@]}

    dc_height=$(( $dc_height+5 > $LINES ? $LINES-5 : $dc_height ))
    dc_height=$(( $dc_height < $PADDING ? $PADDING : $dc_height ))

    # Clamp cursor
    dc_cursor=$(( $dc_cursor < 0 ? 0 : $dc_cursor ))
    dc_cursor=$(( $dc_cursor > $files_size-1 ? $files_size-1 : $dc_cursor ))

    # Print header and navigation bar
    row "---------------------------------------------------------------------------------------"
    row "${hosts[$current_host]}$current_pwd" $col_nav
    row "---------------------------------------------------------------------------------------"

    dc_begin=$(( $dc_cursor - $dc_begin < $PADDING ? $dc_cursor - $PADDING : $dc_begin ))
    dc_begin=$(( $dc_begin < 0 ? 0 : $dc_begin ))
    dc_begin=$(( $dc_cursor - $dc_begin + $PADDING > $dc_height ? $dc_cursor + $PADDING - $dc_height : $dc_begin ))
    dc_begin=$(( $dc_begin < 0 ? 0 : $dc_begin ))

    for (( i=0; i<$dc_height; i++ )); do
        file_index=$((dc_begin + i))

        file=${files[file_index]}
        file_draw=${dc_lines[file_index]}

        color_fg=""
        color_bg=""

        # Check if directory
        case $file in 
            */)
            color_fg=$col_folder;;
            *) 
            # Check if char device
            if [ -c $file ]; then
                color_fg=$col_char
            # Check if socket
            elif [ -S $file ]; then
                color_fg=$col_socket
            # Check if executable
            elif [ -x $file ]; then
                color_fg=$col_exec
            fi
            # Check if link
            if [ -h $file ]; then
                color_fg=$col_link
            fi
        esac


        # Check if selected
        if [ ! -z "$file" ]; then
            local file_marked=${files_marked[$file]}
            if [ ! -z "$file_marked" ]; then
                color_bg=$col_select_bg
                color_fg=$col_select_fg
            fi
        fi
        # Check if current dc_cursor
        if [ $file_index -eq $dc_cursor ]; then
            if [[ $mode = "dir" ]]; then
                if [[ $dir_mode = "flat" ]]; then
                    current_file=$file
                    current_dir=$current_pwd
                fi
                if [[ $dir_mode = "tree" ]]; then
                    current_file=$file
                    current_dir=${file%/*}/
                fi
            else
                current_file=$file
            fi
            color_fg=$col_cursor
        fi
        row ${file_draw:-$file} $color_fg $color_bg
    done

    row "---------------------------------------------------------------------------------------"

    # Print bottom message without entering new line
    local message=$status_message
    [[ ! -z $notification_message ]] && message=$notification_message
    printf "%s\033[K" "$message"

    # clear notification message
    notification_message=""

    # render all with one print command to improve performance
    # printf "%b" "${dc_canvas[*]}"

    # set dc_cursor to beginning
    printf "\e[2000D"
    printf "\e[%sA" $dc_canvas_height
}

function render_loading {
    notification_message="Loading..."
    render
}

function render_clear {
    printf "\e[J"
}

function cursor_to_file {
    for (( i=0; i<${#files[@]}; i++ )); do
        local fname=${files[i]}
        [ ${fname: -1} == '/' ] && fname="${fname::-1}"
        fname=${fname##*/}
        if [[ "$fname" == "$1" ]]; then
            dc_cursor=$i
            return
        fi
    done
}

## Init/Exit/Run Functions #################################################

function _initialize() {
    #configure terminal
    terminal_setup
    terminal_fetch

    # initialize config
    mkdir -p $config_dir
    touch $config_bookmarks

    # initialize cache
    mkdir -p $cache_dir
    mkdir -p $cache_dir_remotes
    touch $cache_selection

    # register signal hooks
    trap _on_stop TSTP
    trap _on_resume CONT
    trap _on_terminate EXIT
    trap 'terminal_fetch; render' WINCH

    # register hosts
    hosts+=($(ls -1 $cache_dir_remotes))
    set_dir
}

# Terminal hooks

function _on_terminate(){
    render_clear
    terminal_restore
    printf "%s" "$current_pwd" > $cache_cd
    printf "%s" "${hosts[current_host]}" > $cache_host
    exit 0
}

function _on_stop(){
    render_clear
    terminal_restore
    kill -s SIGSTOP $$
}

function _on_resume(){
    terminal_setup
    echo Press enter to continue...
}

function _exit(){
    render_clear
    terminal_restore
    printf "%s" "$current_pwd" > $cache_cd
    printf "%s" "${hosts[current_host]}" > $cache_host
    exit 0
}

function _run {
    render_clear
    echo Executing subshell... Type 'exit' to return to fb.
    terminal_restore

    local host="${hosts[current_host]}"
    if [ "$host" = "" ]; then
        command cd "$@"
        eval "$SHELL"
    else
        _execute_remote_pty $SHELL
    fi

    _cd $current_pwd
    terminal_setup
}

function _run_with {
    render_clear
    terminal_restore
    if [ ${#files_marked[@]} -ne 0 ]; then
        eval 'printf "%s\n" "${!files_marked[@]}" | xargs -n 1 -I"{}" $1'
    elif [ -n "$current_file" ]; then
        eval 'printf "$current_file" | xargs -n 1 -I"{}" $1'
    fi
    terminal_setup
}

## Mode actions ############################################################

function _view_keys {
    if [[ $key = '+' ]]; then
        render_clear
        row_nav
        echo 'View commands: '
        echo ' ~        : Toggle tree view'
        echo ' @        : Connect to SSH host'
        echo ' Tab      : Switch between connected SSH hosts'
        echo ' J,K      : Increase/decrease drawing area'
        echo ' F        : Use all avaiable drawing space'
    fi

    # switch flat/tree folder view
    if [[ $key = '~' ]]; then 
        if [[ $dir_mode = "flat" ]]; then
            dir_mode="tree"
        else
            dir_mode="flat"
        fi
        set_dir
    fi
    if [[ $key = '@' ]]; then 
        local host="unknown"
        row_nav "Connect (user@hostname[:port]): "; _read_fb host
        set_host $host
        row_nav_end
    fi
    # Switch between hosts with tab key
    if [[ $key = $'\x09' ]]; then 
        current_host=$(( (current_host+1) % ${#hosts[@]} ))
        set_dir "${hosts_pwd[current_host]}"
    fi
    if [[ $key_l = 'wr' ]]; then 
        if [[ $mode = "selection" ]]; then
            set_selection
        fi
        if [[ $mode = "dir" ]]; then
            set_dir
        fi
        if [[ $mode = "bookmarks" ]]; then
            set_bookmarks
        fi
        key="consumed"
    fi
    # resize actions
    if [[ $key = 'J' ]]; then 
        render_clear
        dc_height=$((dc_height + 1))
    fi
    if [[ $key = 'K' ]]; then 
        render_clear
        dc_height=$((dc_height - 1))
    fi
    if [[ $key = 'F' ]]; then 
        if $fullscreen_mode; then
            render_clear
            fullscreen_mode=false
            dc_height=$HEIGHT
        else
            fullscreen_mode=true
        fi
    fi
}

function _action_keys {
    if [[ $key = '+' ]]; then
        render_clear
        row_nav
        echo 'Action commands: '
        echo ' y        : Confirm current action'
        echo ' n,q      : Cancel'
    fi
    if [[ $key = 'y' ]]; then 
        render_loading
        local host=${hosts[current_host]}
        local path=${host:+ssh://$host/}$current_dir
        #if [ -z "$host" ]; then
        #    path="${current_dir}"
        #else
        #    path="ssh://$host/$current_dir"
        #fi
        if [[ $action = "copy" ]]; then 
            for (( i=0; i<${#files[@]}; i++ )); do
                _copy "${files[i]}" "$path"
            done
        fi
        if [[ $action = "move" ]]; then 
            for (( i=0; i<${#files[@]}; i++ )); do
                _move "${files[i]}" "$path"
            done
        fi
        if [[ $action = "delete" ]]; then 
            for (( i=0; i<${#files[@]}; i++ )); do
                _delete "${files[i]}"
            done
        fi
        _marks_clear
        _selection_clear
        set_dir
    fi
    if [[ $key = 'n' || $key = 'q' ]]; then 
        set_dir
    fi
}

function _navigate_keys {

    if [[ $key = '+' ]]; then
        render_clear
        row_nav
        echo 'Movement commands: '
        echo ' Keys     : 'hjkl' or up, down, left, right arrows'
        echo ' Return   : Open file using xdg-open or enter folder'
        echo ' Backspace: Go back one folder'
        echo ' C^u      : Move up 10 files'
        echo ' C^d      : Move down 10 files'
        echo ' L,M,H    : Move low, middle, high'
        echo ' f[a-z]   : Go to file name starting with'
        echo ' .        : Repeat last "go to" action'
    fi

    # Repeat last go to action
    if [[ $key = '.' ]]; then 
        last_key='f'
        key=$last_goto_key
    fi
    # Go to action
    if [[ $last_key = 'f' ]]; then 
        last_goto_key=$key
        for (( i=$(( $dc_cursor + 1 )); i<${#files[@]}; i++ )); do
            local fname=${files[i]}
            [ ${fname: -1} == '/' ] && fname="${fname::-1}"
            fname=${fname##*/}
            if [[ "$fname" == "$key"* ]]; then
                dc_cursor=$i
                key="consumed"
                return
            fi
        done
        # if not found go again from beginning
        for (( i=0; i<$dc_cursor; i++ )); do
            local fname=${files[i]}
            [ ${fname: -1} == '/' ] && fname="${fname::-1}"
            fname=${fname##*/}
            if [[ "$fname" == "$key"* ]]; then
                dc_cursor=$i
                key="consumed"
                return
            fi
        done
        key="consumed"
        return
    fi

    # up/down
    if [[ $key = 'k' || $key = $'\e[A' ]]; then 
        dc_cursor=$((dc_cursor - 1))
    fi
    if [[ $key = 'j' || $key = $'\e[B' ]]; then 
        dc_cursor=$((dc_cursor + 1))
    fi

    # 10 up/down
    if [[ $key = $'\x15' ]]; then
        dc_cursor=$((dc_cursor - 10))
    fi
    if [[ $key = $'\x04' ]]; then
        dc_cursor=$((dc_cursor + 10))
    fi

    # Move high/middle/low
    if [[ $key = 'H' ]]; then
        dc_cursor=$dc_begin
    fi
    if [[ $key = 'M' ]]; then
        dc_cursor=$(( dc_begin + dc_height / 2))
    fi
    if [[ $key = 'L' ]]; then
        dc_cursor=$(( dc_begin + dc_height ))
    fi

    # Move top/bottom
    if [[ $key_l = 'gg' ]]; then
        dc_cursor=0
        key="consumed"
    fi
    if [[ $key = 'G' ]]; then
        local count=${#dc_lines[@]}
        dc_cursor=$(( count - 1 ))
    fi
}

function _navigate_tree_keys {
    # Expand current folder
    if [[ $key = 'l' || $key = $'\e[C' ]]; then
        if [ -n "$current_file" ]; then
            file="$current_file"
            if [ ${file: -1} = '/' -o -h $file ]; then
                if [ -z "${files_expanded["$file"]}" ]; then
                    files_expanded["$file"]=true
                    set_dir_refresh
                fi
            fi
            key="consumed"
        fi
    fi
    if [[ $key = 'h' || $key = $'\e[D' ]]; then
        if [ -n "$current_file" ]; then
            file="$current_file"
            if [ ${file: -1} = '/' -o -h $file ]; then
                if [ ! -z "${files_expanded["$file"]}" ]; then
                    files_expanded["$file"]=
                    set_dir_refresh
                fi
            fi
            key="consumed"
        fi
    fi
}

function _mark_keys {
    if [[ $key = "+" ]]; then
        render_clear
        row_nav
        echo 'Mark and selection commands: '
        echo ' v  : mark current '
        echo ' V  : mark all '
        echo ' c  : copy marked to clipboard as command arguments'
        echo ' y  : set marked files as selection '
        echo ' Y  : add marked files to selection'
        echo ' x  : clear selection and marks'
    fi
    # mark all
    if [[ $key = 'V' ]]; then
        _marks_add_all
    fi
    # mark current
    if [[ $key = 'v' ]]; then
        _marks_toggle 
        dc_cursor=$((dc_cursor + 1))
    fi
    # add marked files to selection
    if [[ $key = 'y' ]]; then
        _marks_to_selection
        _marks_clear
    fi
    # append marked files to selection
    if [[ $key = 'Y' ]]; then
        _marks_to_selection_append
        _marks_clear
    fi
    # clear marks and selection
    if [[ $key = 'x' ]]; then
        _marks_clear
        _selection_clear
        # switch to directory mode 
        if [[ $mode = "selection" ]]; then
            set_dir
        fi
    fi
    # copy marked to clipboard as command arguments
    if [[ $key_l = 'gc' ]]; then
        _marks_args | xsel -bi
        _marks_clear
    fi
}

function _search_keys {
    # Print commands
    if [[ $key = '+' ]]; then
        render_clear
        row_nav
        echo 'Search commands: '
        echo ' ?  : Find recursive'
        echo ' g? : Find content recursive'
    fi
    # recursive content search
    if [[ $key_l = 'g?' ]]; then
        local search=
        row_nav "Search content: "; _read_fb search
        row_nav_end
        [ $search ] && set_search_content $search
        key="consumed"
    fi
    # recursive search
    if [[ $key = '?' ]]; then
        local search=
        row_nav "Search: "; _read_fb search
        row_nav_end
        [ $search ] && set_search $search
    fi
}

function _common_keys {
    # Print commands
    if [[ $key = '+' ]]; then
        render_clear
        row_nav
        echo 'General: '
        echo ' I  : Edit file'
        echo ' i  : Edit file in a new window'
        echo ' t  : open new terminal window'
        echo ' e  : Execute command using current or marked files'
        echo ' /  : Filter '
        echo ' :  : Run shell command '
        echo ' b  : open/close bookmarks '
        echo ' B  : bookmark current '
    fi
    # Enter current folder
    if [[ $key = 'l' || $key = $'\e[C' || $key = '' ]]; then
        set_dir "$current_file"
    fi
    # Open current file
    if [[ $key = '' ]]; then
        if [ -f "$current_file" ]; then
            xdg-open "$current_file" &>/dev/null &
            disown
        fi
    fi
    # edit current file
    if [[ $key = 'I' ]]; then
        local host="${hosts[current_host]}"
        if [ -n "$current_file" ]; then
            if [ "$host" = "" ]; then
                /bin/bash -c "${APP_EDITOR} \"$current_file\""
            else
                local cmd="cd \"$current_pwd\"; ${APP_EDITOR} \"$current_file\""
                _ssh_current_host -t "$cmd"
            fi
            terminal_setup
        fi
    fi
    # edit current file in a new window
    if [[ $key = 'i' ]]; then
        local host="${hosts[current_host]}"
        if [ -n "$current_file" ]; then
            if [ "$host" = "" ]; then
                nohup ${APP_TERM} -e ${APP_EDITOR} "$current_file" >/dev/null 2>&1 &
            else
                local cmd="cd \"$current_pwd\"; ${APP_EDITOR} \"$current_file\""
                nohup ${APP_TERM} -e \
                    ssh -t -o "$ssh_path" -o "$ssh_master" -o $ssh_persist \
                    "ssh://$host" "$cmd" >/dev/null 2>&1 &
            fi
        fi
    fi
    # open terminal
    if [[ $key = 't' ]]; then
        local host="${hosts[current_host]}"
        if [ -z "$host" ]; then
            nohup ${APP_TERM} >/dev/null 2>&1 &
        else
            local cmd="cd \"$current_pwd\";"
            cmd=$cmd'$SHELL'
            nohup ${APP_TERM} -e \
                ssh -t -o "$ssh_path" -o "$ssh_master" -o $ssh_persist \
                "ssh://$host" "$cmd" >/dev/null 2>&1 &
        fi
    fi
    # run program using current file
    if [[ $key = 'e'  ]]; then
        local cmd=
        row_nav "Execute with (use {} as file arg): "; _read_fb cmd
        [ $cmd ] && _run_with $cmd 
    fi
    # open shell
    if [[ $key = ':' ]]; then
        _run
    fi
    # filter
    if [[ $key = '/' ]]; then
        #row_nav "Filter: "; _read_fb filter
        #row_nav_end
        #[ $filter ] && set_filter $filter

        files_backup=($(printf "${files[*]}"))

        # $1 - text input name
        # $2 - on confirm action
        # $3 - on terminate action
        # $4 - on change action
        _textinput "Filter: " '' 'set_dir' 'files=($(printf "${files_backup[*]}" | grep "$text"))'
    fi
    # show bookmarks
    if [[ $key = 'b' ]]; then
        set_bookmarks
    fi
    # add current dir or marked files to bookmarks
    if [[ $key = 'B' ]]; then
        if [ ${#files_marked[@]} -ne 0 ]; then
            for i in "${!files_marked[@]}"; do
                _bookmark_add "$i"
            done
            _marks_clear
            notification_message="Selection added to bookmarks"
        elif [ -n "$current_dir" ]; then
            _bookmark_add "$current_dir"
            notification_message="Current folder added to bookmarks"
        fi
    fi
    # show selection
    if [[ $key = 's'  ]]; then
        set_selection
    fi
}

function _search_mode_keys {
    if [[ $key = '+' ]]; then
        render_clear
        row_nav
        echo 'Search mode actions: '
        echo ' d  : delete file or folder '
    fi
    # Command delete selected files
    if [[ $key = 'd' ]]; then
        if [ ${#files_marked[@]} -ne 0 ]; then
            set_marks
            set_action "delete" "Delete listed files? (y/n)"
        else
            _marks_add
            set_marks
            set_action "delete" "Delete listed files? (y/n)"
        fi
    fi
}

function _bookmarks_mode_keys {
    if [[ $key = '+' ]]; then
        render_clear
        row_nav
        echo 'Bookmark mode actions: '
        echo ' i  : edit bookmarks '
        echo ' r  : remove bookmark '
        echo ' s  : quit bookmarks view '
    fi
    if [[ $key = 'i' ]]; then
        /bin/bash -c "${APP_EDITOR} \"$config_bookmarks\""
        terminal_setup
        set_bookmarks
    fi
    if [[ $key = 'r' ]]; then
        _bookmark_remove
    fi
    if [[ $key = 'l' || $key = $'\e[C' || $key = '' ]]; then
        local url="$current_file"
        if [[ $url =~ 'ssh' ]]; then
            local host=$(echo $url | sed -n 's|ssh://\([^/]*\).*|\1|p')
            local path=$(echo $url | sed -n 's|ssh://[^/]*\(.*\)|\1|p')
            set_host "$host" "$path"
            key="consume"
        fi
    fi
    if [[ $key = 'b' ]]; then
        key=$'\x1b'
    fi
}

function _selection_mode_keys {
    if [[ $key = '+' ]]; then
        render_clear
        row_nav
        echo 'Selection commands: '
        echo ' i : edit selection '
        echo ' p : copy listed files to current folder '
        echo ' m : move listed files to current folder '
        echo ' r : remove file from selection '
        echo ' d : delete listed files from filesystem '
        echo ' s : quit selection view '
    fi
    if [[ $key = 'i' ]]; then
        /bin/bash -c "${APP_EDITOR} \"$cache_selection\""
        terminal_setup
        set_selection
        key="consumed"
    fi
    if [[ $key = 'r' ]]; then
        _selection_remove
        key="consumed"
    fi
    if [[ $key = 'p' ]]; then
        set_marks
        set_action "copy" "Copy listed files to current folder (y/n)"
        key="consumed"
    fi
    if [[ $key = 'm' ]]; then
        set_marks
        set_action "move" "Move listed files to current folder (y/n)"
        key="consumed"
    fi
    if [[ $key = 'd' ]]; then
        set_marks
        set_action "delete" "delete listed files from filesystem (y/n)"
        key="consumed"
    fi
    if [[ $key = 's' ]]; then
        key=$'\x1b'
    fi
}

function _dir_mode_keys {
    # Print commands
    if [[ $key = '+' ]]; then
        render_clear
        row_nav
        echo 'Current directory commands: '
        echo ' a  : Show hidden files/folders'
        echo ' o  : create file or folder (to create folder end it with "/")'
        echo ' c  : rename current file or folder'
        echo ' p  : copy selection to current directory'
        echo ' m  : move selection to current directory'
        echo ' d  : delete marked files or folders'
    fi
    # Go back one folder
    if [[ $key = 'h' || $key = $'\e[D' || $key = $'\x7f' ]]; then
        set_dir ..
        key="consume"
    fi
    # show hiddne files
    if [[ $key = 'a' ]]; then
        if $show_hidden; then
            show_hidden=false
        else
            show_hidden=true
        fi
        set_dir .
        key="consume"
    fi
    # sort by name
    if [[ $key_l = 'wn' ]]; then
        sort_by="name"
        set_dir .
        key="consume"
    fi
    # sort by modified
    if [[ $key_l = 'wm' ]]; then
        sort_by="modified"
        set_dir .
        key="consume"
    fi
    # sort by size
    if [[ $key_l = 'ws' ]]; then
        sort_by="size"
        set_dir .
        key="consume"
    fi
    # Command create new file
    if [[ $key = 'o'  ]]; then
        row_nav "New file/foler name: "
        _read_fb name
        row_nav_end
        if [[ "$name" == */ ]]; then
            mkdir "$current_dir/$name"
            set_dir "$current_dir/$name"
        else
            touch $current_dir/$name
            set_dir_refresh 
            cursor_to_file $name
        fi
        key="consume"
    fi
    # Command paste selection
    if [[ $key = 'p'  ]]; then
        set_selection
        set_action "copy" "Paste listed files to '$(basename $current_dir)'? (y/n)"
        key="consume"
    fi
    # Command move selection
    if [[ $key = 'm'  ]]; then
        set_selection
        set_action "move" "Move listed files to '$(basename $current_dir)'? (y/n)"
        key="consume"
    fi
    # Command paste selection into
    if [[ $key = 'P' && $current_file == */ ]]; then
        set_selection
        current_dir=$current_file
        set_action "copy" "Paste listed files to '$(basename $current_dir)'? (y/n)"
        key="consume"
    fi
    # Command move selection into
    if [[ $key = 'M' && $current_file == */ ]]; then
        set_selection
        current_dir=$current_file
        set_action "move" "Move listed files to '$(basename $current_dir)'? (y/n)"
        key="consume"
    fi
    # Command rename current file
    if [[ $key = 'c' ]]; then
        if [ -n "$current_file" ]; then
            row_nav "New name: "
            _read_fb name
            terminal_restore
            rename -i $(basename "$current_file") "$name" "$current_file"
            terminal_setup
            row_nav_end

            set_dir_refresh
            key="consume"
        fi
    fi
    # Command delete selected files
    if [[ $key = 'd' ]]; then
        if [ ${#files_marked[@]} -ne 0 ]; then
            set_marks
            set_action "delete" "Delete listed files? (y/n)"
        else
            _marks_add
            set_marks
            set_action "delete" "Delete listed files? (y/n)"
        fi
        key="consume"
    fi
}

## Program execution ###########################################################

# Print last ssh destination and exit
if [[ $1 = "-h" ]]; then
    if [ -f $cache_host ]; then
        echo $(cat $cache_host)
    fi
    exit 0
fi

# Print last directory and exit
if [[ $1 = "-d" ]]; then
    if [ -f $cache_cd ]; then
        echo $(cat $cache_cd)
    fi
    exit 0
fi
# Start program with initial directory
if [[ -d $1 ]]; then
    cd $1
fi

# Initialize terminal, cache, hooks
_initialize

if [[ $1 = "-b" ]]; then
    # show bookmarks 
    set_bookmarks
fi

while :
do
    render

    # read key
    k1=
    k2=
    k3=
    read -sn1 k1
    if [[ $k1 == $'\e' || $k1 == $'\x' ]]; then
        read -sn1 -t 0.0001 k2
        read -sn1 -t 0.0001 k3
    fi
    key=${k1}${k2}${k3}
    key_l=$last_key${k1}${k2}${k3}

    # handle key actions
    _view_keys

    current_mode=$mode

    if [[ $current_mode = "action" ]]; then
        _navigate_keys
        _action_keys
    elif [[ $current_mode = "selection" ]]; then
        _navigate_keys
        _common_keys
        _mark_keys
        _selection_mode_keys
    elif [[ $current_mode = "search" ]]; then
        _navigate_keys
        _common_keys
        _mark_keys
        _search_keys
        _search_mode_keys
    elif [[ $current_mode = "bookmarks" ]]; then
        _navigate_keys
        _common_keys
        _bookmarks_mode_keys
    fi

    if [[ $current_mode = "dir" ]]; then
        if [[ $dir_mode = "tree" ]]; then
            _navigate_tree_keys
        fi
        _navigate_keys
        _common_keys
        _mark_keys
        _search_keys
        _dir_mode_keys
    fi
    # Restore dir mode
    # backspace, esc
    if [[ $key = $'\x7f' || $key = $'\x1b' ]]; then
        set_dir
    fi

    # quit 
    if [[ $key = q ]]; then
        _exit
    fi

    last_key=$key
done

terminal_restore
exit 0
