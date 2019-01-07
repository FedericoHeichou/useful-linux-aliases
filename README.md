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
