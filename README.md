# UsefulLinuxAliases
List of useful Linux Aliases to add into bashrc.  
Remember to refresh using `. ~/.bashrc` or what else you are editing.

#### Kill a list of process using their command name
```bash
killname() { if [ $# -eq 0 ]; then echo -e "No arguments specified. Usage:\nkillname commandname"; return 1; fi
for pid in `ps -eo pid,command | grep $1 | grep -v grep | awk '{print $1}'` ; do kill $pid; done }
```

#### Transfer file using curl and trasfer.sh public server
```bash
transfer() { if [ $# -eq 0 ]; then echo -e "No arguments specified. Usage:\necho transfer /tmp/test.md\ncat /tmp/test.md | transfer test.md"; return 1; fi 
tmpfile=$( mktemp -t transferXXX ); if tty -s; then basefile=$(basename "$1" | sed -e 's/[^a-zA-Z0-9._-]/-/g'); curl --progress-bar --upload-file "$1" "https://transfer.sh/$basefile" >> $tmpfile; else curl --progress-bar --upload-file "-" "https://transfer.sh/$1" >> $tmpfile ; fi; cat $tmpfile; rm -f $tmpfile; echo "";} 
```

#### See RAM used by a process
```bash
mem(){
  ps -eo rss,pid,euser,args:100 --sort %mem | grep -v grep | grep -i $@ | awk '{ sum += $1; printf $1/1024 "MB"; $1=""; print } END { print "Total memory usage: " sum/1024 "MB" }'
}
```

#### Slows and reset bandwidth
```bash
alias rallenta="sudo wondershaper enp2s0 2048 300"
alias reimposta="sudo wondershaper clear enp2s0"
```

#### mkdir then cd
```bash
mkcd() { mkdir "$1" && cd "$1"; }
```

#### What is my public ip
```bash
alias whatismyip="dig +short myip.opendns.com @resolver1.opendns.com"

```
#### Server web via docker (www in app/ and db in mysql/)
```bash
# A helper function to launch docker container using mattrayner/lamp with overrideable parameters
#
# $1 - Apache Port (optional)
# $2 - MySQL Port (optional - no value will cause MySQL not to be mapped)
function launchdockerwithparams {
    APACHE_PORT=80
    MYSQL_PORT_COMMAND=""
    if ! [[ -z "$1" ]]; then
        APACHE_PORT=$1
    fi
    if ! [[ -z "$2" ]]; then
        MYSQL_PORT_COMMAND="-p \"$2:3306\""
    fi
    docker run -i -t -p "$APACHE_PORT:80" $MYSQL_PORT_COMMAND -v ${PWD}/app:/app -v ${PWD}/mysql:/var/lib/mysql mattrayner/lamp:latest
}
alias launchdocker='launchdockerwithparams $1 $2'
alias ldi='launchdockerwithparams $1 $2'
```

#### Check differences between two repositories (for example if you want to perform a full Laravel upgrade)
```bash
function gitdifferences {
    git clone "$1" base || (echo "Usage: gitdifferences url_git_base url_git_new" && return 1)
    git clone "$2" new || (echo "Usage: gitdifferences url_git_base url_git_new" && return 1)
    cd base
    git remote add new ../new/
    git fetch new master:diff
    git diff diff
    # Then using a program like PhpStorm check "branches", click on "diff" branch then "Show Diff with Working Tree"
}
```

#### Exec a command inside a Docker container (if no command provided exec bash or sh)
```bash
function dockerexec {
    ! [[ -z "$2" ]] && docker exec -it $1 $2 || docker exec -it $1 bash || docker exec -it $1 sh || echo "Usage: dockerexec container command"
}
alias dsh='dockerexec $1 $2'
```

#### Clear all snap revisions
```bash
function snapclear {
    set -eu

    LANG=en_US.UTF-8 snap list --all | awk '/disabled/{print $1, $3}' |
    while read snapname revision; do
        sudo snap remove "$snapname" --revision="$revision"
    done
}
alias snapclear='snapclear'
```

### Build and push Docker image
```bash
dockerpush() {(
    [[ -z "$1" ]] && echo "Usage: $0 <name> [tag]" && exit 1
    [[ -z "$2" ]] && TAG='latest' || TAG="$2"
    [[ -z "`ls Dockerfile 2>/dev/null`" ]] && echo "Dockerfile not found in the current directory" && exit 1
    URL='nexus.example.org:10000'
    IMAGE="$1:$TAG"
    set -x
    docker build --tag "$IMAGE" . || exit 1
    docker tag "$IMAGE" "$URL/$IMAGE" || exit 1
    docker push "$URL/$IMAGE" || exit 1
)}
```

### Listing process and threads in a docker container
```bash
dockerthreads() {(
    [[ -z "$1" ]] && echo "Usage: $0 containername" && exit 1
    docker top $1 | tail -n +2 | awk '{print $2}' | while read p; do
    ps -o pid,user,cmd:255 -Tp $p | tail -n +2 | uniq -c
    done | awk '{gsub(/^\s+/, "", $0); print}{sum+=$1} END{print sum}'
)}
```

### Print usage of cpu of a command like the `time` command style
```bash
cputime() {
    cat <(grep 'cpu ' /proc/stat) <($@ && grep 'cpu ' /proc/stat) | awk -v RS="" '{print ($13-$2+$15-$4)*100/($13-$2+$15-$4+$16-$5)}'
}
```

### Removing snap from df output
```bash
alias df='df -x"squashfs"'
```
