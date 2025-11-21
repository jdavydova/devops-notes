## Linux

🔸 Основи командного рядка:
- `ls`, `cd`, `pwd`, `mkdir`, `rm`, `cp`, `mv`
- `chmod`, `chown`, `ps`, `top`, `kill`

🔸 Шпаргалка:
- [Linux Command Cheat Sheet (PDF)](https://files.fosswire.com/2007/08/fwunixref.pdf)

🔸 Мої нотатки:
- # Як підключитися по SSH

connecting to your Ubuntu Virtual Machine in VirtualBox via SSH is something every DevOps engineer should know!
Let’s go step-by-step 👇

🧭 1. Check if SSH Server is installed on Ubuntu

Open the Ubuntu VM terminal and run:

sudo apt update
sudo apt install -y openssh-server


Then check that it’s running:

sudo systemctl status ssh


✅ You should see “active (running)”.

If not — start and enable it:

sudo systemctl enable ssh
sudo systemctl start ssh

🌐 2. Find the Ubuntu VM’s IP address

Run:

ip a


Look for something like:

inet 192.168.56.101/24 brd 192.168.56.255 scope global dynamic eth0


👉 The part before /24 (for example 192.168.56.101) is your Ubuntu’s IP address.

🧱 3. Configure VirtualBox Network (important)

For SSH to work, you need your host (macOS or Windows) to reach that IP.

✅ Option A — NAT + Port Forwarding

Stop the VM if it’s running.

Go to VirtualBox → Settings → Network

Select Attached to: NAT

Click Advanced → Port Forwarding

Add a new rule:

Name	Protocol	Host Port	Guest Port
SSH	TCP	2222	22

Then start the VM again.

✅ Option B — Bridged Adapter

If you want the VM to appear as a real device in your network (same LAN IP range), choose:

Attached to: Bridged Adapter


→ Then your Ubuntu will get an IP like 192.168.x.x directly reachable from your host.

🔐 4. Connect via SSH (from host terminal)

Now, on your macOS or Windows terminal, run:

🧩 If using NAT + Port Forwarding:
ssh -p 2222 yourusername@127.0.0.1


(Replace yourusername with your Ubuntu login name — e.g., ubuntu or julia.)

🧩 If using Bridged Adapter:
ssh yourusername@192.168.56.101


(Use the IP from ip a output.)

🧠 5. Verify Connection

If all is good, you’ll see:

The authenticity of host ... can't be established.
Are you sure you want to continue connecting (yes/no)?


Type:

yes


Then enter your Ubuntu password 🔑

You’re now connected via SSH 🎉

🧩 6. (Optional) Check SSH connectivity from inside VM

To confirm SSH is reachable:

sudo ss -tlnp | grep ssh


You should see something like:

LISTEN 0 128 0.0.0.0:22 ...


- # Як створити користувача

✅ Create the user without creating a same-named group

Use useradd with --gid to assign an existing group, or just tell adduser to skip creating the group.

Option 1 — Create user and assign existing admin group

    sudo useradd -m -g admin admin
    sudo passwd admin

Option 2 — Create user with its own group, then add to admin group

This avoids the conflict entirely:

    sudo adduser adminuser
    sudo usermod -aG admin adminuser


(Use any username except admin.)

✅ Check whether user exists

Run:

    getent passwd admin


If it returns nothing — the user truly doesn’t exist.

Check the group:

    getent group admin


This will show the existing admin group.

⭐ Recommended (Ubuntu best practice)

Ubuntu doesn’t use the admin group anymore.
Modern Ubuntu uses sudo group.

Better approach:

    sudo adduser admin
    sudo usermod -aG sudo admin


But since the group admin blocks creation, instead:

    sudo adduser admin2
    sudo usermod -aG sudo admin2

If you REALLY want username = admin

Here is the clean method:

    sudo groupdel admin
    sudo adduser admin
    sudo usermod -aG sudo admin


⚠️ Only do this if nothing important uses the admin group.
- Щоб увімкнути SSH-вхід для користувача admin, потрібно зробити 2 речі:

✅ 1. Переконатися, що у admin є доступ через SSH-ключ або пароль

SSH-доступ працює автоматично, якщо в користувача є:

пароль
або

SSH ключі в ~/.ssh/authorized_keys

Перевір, чи існує папка .ssh у admin:

   ls -la /home/admin


Якщо немає папки .ssh, то ключі не скопійовані → треба зробити крок 2.

✅ 2. Дозволити SSH для admin (скопіювати ключі root)

Найпростіше — скопіювати SSH ключі з root у admin:

sudo mkdir -p /home/admin/.ssh
sudo cp /root/.ssh/authorized_keys /home/admin/.ssh/
sudo chown -R admin:admin /home/admin/.ssh
sudo chmod 700 /home/admin/.ssh
sudo chmod 600 /home/admin/.ssh/authorized_keys


Це увімкне SSH-доступ тим самим ключем, яким ти заходиш як root.

Тепер спробуй увійти:

ssh admin@SERVER_IP

