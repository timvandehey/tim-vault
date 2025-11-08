![](https://miro.medium.com/v2/resize:fit:875/1*tWFrw-NcoSKdSdXRCDpHWQ.png)

Boost Your Terminal Productivity

[

![Ashish Singh](https://miro.medium.com/v2/resize:fill:64:64/1*LwwW8iuLsLBasPSo9kaZPw.jpeg)



](https://ashishnoob.medium.com/?source=post_page---byline--42fbbcb32b90---------------------------------------)

> Boost Your Terminal Productivity with These Time-Saving Tricks

Linux mastery isn’t about memorizing every command — it’s about working smarter, not harder. Whether you’re a developer, DevOps engineer, or sysadmin, these 30 Linux shortcuts will save you hours, reduce errors, and make you a terminal power user.

Let’s dive in — categorized for easy navigation with real-world examples and pro tips!

## 🚀 Why Learn Linux Shortcuts?

- Speed up workflows — Reduce repetitive typing.
- Avoid mistakes — Fewer typos = fewer headaches.
- Impress colleagues — Effortlessly run complex one-liners.
- Become terminal-fluent — Essential for cloud, DevOps, and scripting.

**_Pro Tip:_** Bookmark this guide and add your favorites to `~/.bashrc` or `~/.zshrc` for permanent access!

## 🧭 Navigation & Command Line Mastery

**1.** `**cd -**`

Switch to your last directory

cd /var/log    
cd /home/user    
cd -  

**2.** `**cd !$**`

Use the last argument from the previous command

mkdir /projects/new-app    
cd !$  

**3.** `**!!**` – Rerun the Last Command

Ever forget `sudo`? Fix it instantly!

apt update    
sudo !!  

**4.** `**!command**` – Repeat a Specific Past Command

!vim     
!ssh   

**5.** `**Ctrl + R**` – Reverse Command Search

Fuzzy-search your history → Start typing to find past commands.

**6.** `**Ctrl + A**` **/** `**Ctrl + E**`

Jump to the start/end of the line (Faster than holding → key!)

**7.** `**Ctrl + U**` **/** `**Ctrl + K**`

Delete text from cursor to start/end of line

**8.** `**Alt + .**` – Insert Last Argument

scp file.txt user@server:/path/    
vim !$  

**9.** `**pushd**` **&** `**popd**` – Directory Bookmarks

pushd /etc/nginx    
  
popd  

**10.** `**Ctrl + W**` – Delete Previous Word

Faster than backspacing!

## 🔍 System Monitoring & Debugging

**11.** `**df -h**` – Check Disk Space

Human-readable output (No more byte math!)

**12.** `**du -sh ***` – Folder Sizes

See what’s eating your disk:

du -sh /var/* | sort -h  

**13.** `**htop**` – Better Than `top`

Colorful, interactive process viewer (Install with `apt install htop`).

**14.** `**lsof -i :3000**` – Find Port Users

Who’s blocking your app’s port?

**15.** `**kill -9 $(lsof -t -i:3000)**`

Nuke a process hogging a port

**16.** `**watch -n 1 free -h**`

Live RAM usage updates every second

**17.** `**journalctl -fu nginx**` – Follow Service Logs

Debug crashes in real time

## 🌐 Networking Shortcuts

**18.** `**curl -I example.com**` – Check HTTP Headers

Verify caching, redirects, and server info.

**19.** `**ss -tuln**` – Modern `netstat`

All listening ports (`-t` = TCP, `-u` = UDP).

**20.** `**rsync -avhP src/ dest/**` – Fast File Transfers

Resumes interrupted copies (`-P` = progress).

**21.** `**scp -r ~/projects user@server:/backup**`

Securely copy folders over SSH.

**22.** `**nmap -sP 192.168.1.0/24**` – Scan Local Network

Find all connected devices.

## ⚡ Process & Job Control

**23.** `**command & disown**` – Detach a Background Job

python long_script.py & disown  

24. `**nohup command &**` – Another Background Trick

Saves output to `nohup.out`.

25. `**Ctrl + Z**` **→** `**bg**` – Pause/Resume Jobs

Ctrl + Z    
bg  

**26.** `**crontab -e**` – Schedule Tasks

Example (run at 2 AM daily):

0 2 * * * /backup.sh

## 📦 File & Package Management

**27.** `**find . -name "*.log"**` – Locate Files

Case-insensitive search:

find . -iname "*.Log"

**28.** `**grep -rnw . -e "error"**` – Search File Contents

Recursive + line numbers.

**29.** `**tar -czvf backup.tar.gz /data**` – Compress

Exclude files:

tar -czvf backup.tar.gz --exclude="*.tmp" /data

**30.** `**alias ll='ls -lah'**` – Create Shortcuts

Make permanent by adding to `~/.bashrc`.

## 💡 Pro Tips for Power Users

- Combine commands with pipes (`|`):

ps aux | grep nginx | awk '{print $2}' | xargs kill -9

- Use `man` pages (e.g., `man grep`) for deep dives.
- Version control your `~/.bashrc` to sync aliases across machines.

## 🚀 Final Thoughts

These 30 shortcuts will 10x your Linux efficiency. Start with 5 favorites, then gradually add more.

> 🚀 Share this guide with your team if you found this helpful!  
> 🔔 Follow to stay updated.  
> 🌟 Enjoyed this series? Give it 50 claps!