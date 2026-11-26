Ubuntu VM ပေါ်တွင် Personal Finance Manager API Deploy လုပ်ခြင်း

ဤလမ်းညွှန်တွင် ၂၀၂၆ ခုနှစ် နိုဝင်ဘာလ ၂၆ ရက်နေ့၌ ပြုလုပ်ခဲ့သော deployment အဆင့်များကို မှတ်တမ်းတင်ထားသည်။ Project သည် NestJS API ဖြစ်ပြီး Ubuntu VirtualBox VM ပေါ်တွင် Bun၊ PostgreSQL၊ TypeORM Migration နှင့် Systemd ကို အသုံးပြုထားသည်။

အရေးကြီးသည်။ Database password၊ private key၊ token နှင့် အခြား secret များကို GitHub ပေါ် မတင်ရပါ။ ဤဖိုင်ရှိ <DB_PASSWORD> နေရာတွင် lab password ကို VM အတွင်း၌သာ အစားထိုးအသုံးပြုပါ။

Deployment ပြီးဆုံးသည့်အခါ ရရှိမည့်အခြေအနေ

VM boot တက်လာသည့်အခါ PostgreSQL အလိုအလျောက် start ဖြစ်မည်။

API ကို Systemd က အလိုအလျောက် start လုပ်ပေးမည်။

Systemd က PostgreSQL ကိုအရင် start လုပ်ပြီးမှ API ကို start လုပ်မည်။

Project အတွင်း .env file မဖန်တီးဘဲ environment variables များကို Systemd service ထဲတွင် သတ်မှတ်မည်။

Bun ဖြင့် compiled production build ကို run မည်။

VM reboot လုပ်ပြီးနောက်လည်း PostgreSQL ထဲရှိ data မပျက်ဘဲ ကျန်ရှိမည်။

Windows Postman မှ VirtualBox port forwarding ဖြင့် API ကို access လုပ်နိုင်မည်။

System Architecture

Windows Postman
    localhost:3001
          |
          | VirtualBox Port Forwarding
          v
Ubuntu VM 10.0.2.15:3000
          |
          v
Systemd -> Bun -> NestJS API -> PostgreSQL:5432

အသုံးပြုခဲ့သည့် Git Branch

Deployment ပြုလုပ်ရာတွင် အဟောင်း main branch ကို မသုံးဘဲ PostgreSQL code ပါရှိသော အောက်ပါ branch ကို အသုံးပြုခဲ့သည်။

teacher-postgresql

ဆရာ့ PostgreSQL changes များပါသည့် branch ကို ကိုယ့် GitHub repository သို့ push လုပ်ထားပြီး VM က origin/teacher-postgresql ကို fetch လုပ်ကာ local tracking branch အဖြစ် အသုံးပြုခဲ့သည်။

လက်ရှိ branch ကိုစစ်ရန်:

git branch --show-current
git status

Expected branch:

teacher-postgresql

1. PostgreSQL Install လုပ်ခြင်း

Ubuntu package list ကို update လုပ်ပြီး PostgreSQL ကို install လုပ်ပါ။

sudo apt update
sudo apt install -y postgresql postgresql-contrib

postgresql သည် အဓိက database server package ဖြစ်သည်။

postgresql-contrib တွင် PostgreSQL extension နှင့် utility အပိုများ ပါဝင်သည်။

postgres-contrib သည် package name အမှားဖြစ်သောကြောင့် မသုံးရပါ။

-y သည် installation confirmation ကို အလိုအလျောက် Yes ဖြေပေးသည်။

PostgreSQL ကို ယခုချက်ချင်း start လုပ်ပြီး VM boot တက်သည့်အခါ အလိုအလျောက် start ဖြစ်စေရန်:

sudo systemctl enable --now postgresql

Service နှင့် database listener ကို စစ်ရန်:

sudo systemctl status postgresql --no-pager
pg_isready

အောက်ပါအတိုင်းရလျှင် PostgreSQL က connection များကို လက်ခံနေပြီဖြစ်သည်။

/var/run/postgresql:5432 - accepting connections

2. PostgreSQL Role နှင့် Database ဖန်တီးခြင်း

PostgreSQL Linux account သို့ ပြောင်းပါ။

sudo -iu postgres

PostgreSQL console ထဲဝင်ပါ။

psql

Application အတွက် database role ဖန်တီးပါ။

CREATE ROLE pfm_user WITH LOGIN;

Password ကို interactive နည်းဖြင့် သတ်မှတ်ပါ။

\password pfm_user

Password ရိုက်နေစဉ် screen ပေါ်တွင် မပေါ်သော်လည်း value ဝင်နေပါသည်။ Password ကို နှစ်ကြိမ်တူအောင် ရိုက်ရမည်။

Database ဖန်တီးပြီး pfm_user ကို owner အဖြစ် သတ်မှတ်ပါ။

CREATE DATABASE pfm_db OWNER pfm_user;

Role နှင့် database ကို စစ်ပါ။

\du
\l

PostgreSQL console မှ ထွက်ပါ။

\q

PostgreSQL Linux account မှ ubuntu account သို့ ပြန်ထွက်ပါ။

exit

Password authentication ကိုစမ်းပါ။

psql -h 127.0.0.1 -U pfm_user -d pfm_db -W

Connection အောင်မြင်ပြီးနောက် ထွက်ရန်:

\q

pfm_user သည် pfm_db ၏ owner ဖြစ်သောကြောင့် PostgreSQL SUPERUSER permission ပေးစရာမလိုပါ။

3. GitHub မှ PostgreSQL Branch ကို VM ထဲယူခြင်း

အရင် clone လုပ်ထားသော project directory ထဲဝင်ပါ။

cd /usr/local/bin/personal-finance-manager

ယခင် npm installation ကြောင့် untracked package-lock.json ဖိုင်ရှိပါက git status ဖြင့် သေချာစစ်ပြီး ထို untracked file ကိုသာ ဖျက်ပါ။

git status
rm package-lock.json

GitHub မှ နောက်ဆုံး branch information ကိုယူပါ။

git fetch origin
git branch -a

Remote PostgreSQL branch ကို track လုပ်သည့် local branch ဖန်တီးပြီး switch လုပ်ပါ။

git switch --track origin/teacher-postgresql

စစ်ဆေးရန်:

git branch --show-current
git status

Expected result:

teacher-postgresql
Your branch is up to date with 'origin/teacher-postgresql'.
nothing to commit, working tree clean

Local branch ရှိပြီးသားဖြစ်ပါက:

git switch teacher-postgresql
git pull

4. Bun Install လုပ်ခြင်း

Linux ပေါ်ရှိ Bun installer အတွက်လိုအပ်သော unzip ကို install လုပ်ပါ။

sudo apt install -y unzip

Official installation script ဖြင့် Bun ကို install လုပ်ပါ။

curl -fsSL https://bun.com/install | bash
source ~/.bashrc

Bun version နှင့် binary path ကိုစစ်ပါ။

bun --version
which bun

ဤ deployment တွင် ရရှိခဲ့သော path:

/home/ubuntu/.bun/bin/bun

npm install -g bun နည်းလမ်းအစား official script ကိုရွေးခဲ့သည်။ ထိုနည်းဖြင့် Bun သည် NVM ၏ Node version directory အောက်တွင် မရှိဘဲ တည်ငြိမ်သော path တစ်ခုရရှိသည်။ ထို absolute path ကို Systemd ExecStart တွင် တိုက်ရိုက်သုံးနိုင်သည်။

နည်းလမ်းနှစ်ခုလုံးကို တစ်ပြိုင်တည်းမသုံးသင့်ပါ။ ထိုသို့သုံးပါက PATH ထဲရှိ မည်သည့် Bun binary ကို run နေသည်ဆိုတာ ရှုပ်ထွေးနိုင်သည်။

5. Bun ဖြင့် Project Dependencies Install လုပ်ခြင်း

System ထဲရှိ npm program ကို uninstall လုပ်စရာမလိုပါ။ ယခင် npm installation က ဖန်တီးထားသည့် project dependency directory ကိုသာ ရှင်းပါ။

rm -rf ./node_modules

Repository ထဲတွင် Bun lock file ရှိကြောင်းစစ်ပါ။

ls -l bun.lock

bun.lock ထဲတွင် မှတ်တမ်းတင်ထားသည့် dependency versions အတိအကျအတိုင်း install လုပ်ပါ။

bun install --frozen-lockfile

--frozen-lockfile သည် Bun အား bun.lock ကို ပြောင်းလဲခြင်းမပြုရန် တားမြစ်ထားသည်။ package.json နှင့် bun.lock မကိုက်ညီပါက မတူညီသော dependency versions များကို အသံတိတ် generate မလုပ်ဘဲ installation ကို error ဖြင့် ရပ်မည်။

နားလည်ရန်:

package.json = Project က ဘယ် packages တွေလိုအပ်သလဲ
bun.lock     = ဘယ် version အတိအကျကို သုံးမလဲ
node_modules = Install ပြီးသား packages တွေထားတဲ့ directory

Working tree ကိုစစ်ပါ။

git status

6. Migration အတွက် ယာယီ Environment Variables သတ်မှတ်ခြင်း

Server ပေါ်ရှိ project အတွင်း .env file မဖန်တီးပါ။ Migration တစ်ကြိမ် run ရန် လက်ရှိ shell session ထဲတွင် variables များကို ယာယီ export လုပ်ပါ။

export DATABASE_URL='postgresql://pfm_user:<DB_PASSWORD>@127.0.0.1:5432/pfm_db'
export DATABASE_SSL='false'
export PORT='3000'

ဤ exported variables များသည် လက်ရှိ shell session ပိတ်သွားသည့်အခါ ပျောက်သွားမည်။ Application အမြဲတမ်း run ရန်လိုအပ်သော values များကို နောက်ပိုင်း Systemd service ထဲတွင် သတ်မှတ်မည်။

Password တွင် @, :, /, # ကဲ့သို့ special characters ပါရှိပါက URL encoding လုပ်ရန်လိုနိုင်သည်။

7. Application Build နှင့် Database Migration

NestJS application ကို Bun ဖြင့် build လုပ်ပါ။

bun run build

Pending TypeORM migrations များကိုစစ်ပါ။

bunx typeorm migration:show -d dist/database/data-source.js

Migration ကို run ပါ။

bun run migrate:run

Migration အောင်မြင်ပါက categories table၊ indexes၊ constraints နှင့် typeorm_migrations history table တို့ကို ဖန်တီးပေးမည်။

Tables များကို စစ်ရန်:

psql -h 127.0.0.1 -U pfm_user -d pfm_db -W -c '\dt'

Package Script နှင့် ပတ်သက်သည့်မှတ်ချက်

Deployment ပြုလုပ်ချိန်တွင် package.json ၏ wrapper script ထဲ၌ npm command ကျန်နေသေးသည်။ ဥပမာ:

"migrate": "npm run build && npm run migrate:run"

npm ကို မခေါ်ဘဲ Bun သာသုံးနိုင်ရန် အောက်ပါ command နှစ်ခုကို သီးခြား run ခဲ့သည်။

bun run build
bun run migrate:run

Source repository ထဲရှိ npm-based wrapper ကို ရည်ရွယ်ချက်ရှိရှိ update လုပ်ပြီး test မလုပ်ရသေးမီ ထို wrapper ကို မသုံးသင့်ပါ။

8. PostgreSQL Password Authentication Error ဖြေရှင်းခြင်း

အောက်ပါ error သည် DATABASE_URL ထဲရှိ password နှင့် PostgreSQL pfm_user ၏ password မကိုက်ညီခြင်းကို ဆိုလိုသည်။

password authentication failed for user "pfm_user"
code: 28P01

PostgreSQL administrator ဖြင့် lab password ကို ပြန်သတ်မှတ်ပါ။

sudo -u postgres psql -c "ALTER ROLE pfm_user WITH LOGIN PASSWORD '<DB_PASSWORD>';"

Migration ပြန်မလုပ်မီ connection ကို အရင်စမ်းပါ။

psql -h 127.0.0.1 -U pfm_user -d pfm_db -W -c '\conninfo'

Connection အောင်မြင်ပြီးမှ environment variable ကို password အမှန်ဖြင့် ပြန်သတ်မှတ်ကာ migration ကို ပြန် run ပါ။

9. Systemd Application Service ဖန်တီးခြင်း

Systemd unit file ကိုဖန်တီးပါ။

sudo nano /etc/systemd/system/pfm-api.service

အောက်ပါ configuration ကိုထည့်ပြီး <DB_PASSWORD> နေရာတွင် password အမှန်ကို VM ပေါ်၌သာ အစားထိုးပါ။

[Unit]
Description=Personal Finance Manager API
Requires=postgresql.service
After=postgresql.service

[Service]
Type=simple
User=ubuntu
Group=ubuntu
WorkingDirectory=/usr/local/bin/personal-finance-manager

Environment="NODE_ENV=production"
Environment="PORT=3000"
Environment="DATABASE_URL=postgresql://pfm_user:<DB_PASSWORD>@127.0.0.1:5432/pfm_db"
Environment="DATABASE_SSL=false"

ExecStart=/home/ubuntu/.bun/bin/bun dist/main.js

Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target

အဓိက dependency directives များ:

Requires=postgresql.service
After=postgresql.service

Requires=postgresql.service သည် API service start လုပ်သည့်အခါ PostgreSQL service ကိုပါ start လုပ်ပေးသည်။

After=postgresql.service သည် PostgreSQL startup job ပြီးမှ API startup job ကို ဆက်လုပ်စေသည်။

User=ubuntu သည် application ကို root အဖြစ်မဟုတ်ဘဲ ubuntu user အဖြစ် run စေသည်။

WorkingDirectory သည် application directory ကို သတ်မှတ်သည်။

ExecStart သည် Bun ဖြင့် compiled dist/main.js ကို run သည်။

Restart=on-failure သည် application error ဖြင့်ရပ်သွားပါက ပြန် start လုပ်ပေးသည်။

Nano တွင် save နှင့် exit လုပ်ရန်:

Ctrl+O နှိပ်ပါ။

Enter နှိပ်ပါ။

Ctrl+X နှိပ်ပါ။

Systemd configuration ကို reload လုပ်ပြီး service ကို boot အချိန်တွင် auto-start ဖြစ်အောင် enable လုပ်ကာ ယခုချက်ချင်း start လုပ်ပါ။

sudo systemctl daemon-reload
sudo systemctl enable --now pfm-api.service

Application service ကိုစစ်ပါ။

sudo systemctl status pfm-api.service --no-pager

Expected status:

Active: active (running)

Error ရှိပါက logs ကြည့်ရန်:

sudo journalctl -u pfm-api.service -n 50 --no-pager

10. VM အတွင်းမှ API Test လုပ်ခြင်း

Categories endpoint ကိုစမ်းပါ။

curl -i http://127.0.0.1:3000/categories

Expected status:

HTTP/1.1 200 OK

Response [] ရရှိခြင်းသည် error မဟုတ်ပါ။ Database table ထဲတွင် category data မရှိသေးခြင်းကို ဆိုလိုသည်။

Swagger ကိုစမ်းပါ။

curl -I http://127.0.0.1:3000/docs

Expected status:

HTTP/1.1 200 OK

VM IP address ကိုကြည့်ရန်:

hostname -I

ဤ deployment တွင် VirtualBox NAT address သည်:

10.0.2.15

ဖြစ်သည်။

11. VirtualBox Port Forwarding သတ်မှတ်ခြင်း

VM သည် VirtualBox NAT networking ကိုအသုံးပြုထားသည်။ VirtualBox ရှိ Settings > Network > Adapter 1 > Advanced > Port Forwarding တွင် အောက်ပါ rule ကိုထည့်ပါ။

Setting

Value

Name

pfm-api

Protocol

TCP

Host IP

127.0.0.1

Host Port

3001

Guest IP

10.0.2.15

Guest Port

3000

ဤ mapping ၏အဓိပ္ပာယ်:

Windows localhost:3001 -> Ubuntu VM 10.0.2.15:3000

Windows/Postman ဘက်မှ port 3001 ကိုသုံးသော်လည်း VM အတွင်းရှိ application က port 3000 ကို နားထောင်နေခြင်းဖြစ်သည်။

12. Windows Postman မှ API Test လုပ်ခြင်း

Postman တွင် GET request ပို့ပါ။

GET http://localhost:3001/categories

Windows browser မှ Swagger ဖွင့်ရန်:

http://localhost:3001/docs

Category အသစ်ဖန်တီးရန်:

POST http://localhost:3001/categories

Postman တွင် Body > raw > JSON ကိုရွေးပြီး ဥပမာ request body ထည့်ပါ။

{
  "name": "Food"
}

GET request ကို ထပ်ပို့ပြီး သိမ်းထားသည့် category ကို PostgreSQL မှ ပြန်ပေးကြောင်းစစ်ပါ။

Example response:

[
  {
    "id": "28127683-a66b-41a4-9ac8-393ec359a734",
    "name": "Food",
    "totalAmount": 50.5
  }
]

Amount ထည့်သည့် endpoint နှင့် request body ကို Swagger /docs မှ အတိအကျကြည့်နိုင်သည်။

13. PostgreSQL ထဲရှိ Data ကို တိုက်ရိုက်စစ်ခြင်း

PostgreSQL categories table ထဲရှိ records များကို ပြပါ။

sudo -u postgres psql -d pfm_db -c 'SELECT id, name, total_amount, created_at FROM categories;'

Migration history ကိုပြပါ။

sudo -u postgres psql -d pfm_db -c 'SELECT * FROM typeorm_migrations;'

Postman မှထည့်ခဲ့သည့် data ကို table ထဲတွင်တွေ့ရလျှင် API မှ database သို့ data သိမ်းခြင်း အောင်မြင်သည်။

14. Reboot ပြီးနောက် Automatic Startup စစ်ခြင်း

VM ကို restart လုပ်ပါ။

sudo reboot

sudo reboot ၏အဓိပ္ပာယ်:

sudo သည် command ကို administrator permission ဖြင့် run သည်။

reboot သည် operating system ကို restart လုပ်သည်။

Windows တွင် Restart နှိပ်ခြင်းနှင့် တူသည်။

SSH အသုံးပြုနေပါက VM restart လုပ်နေစဉ် အောက်ပါ message ရခြင်းသည် ပုံမှန်ဖြစ်သည်။

Connection to 127.0.0.1 closed by remote host.

VM ပြန်တက်လာပြီးနောက် SSH ဖြင့်ပြန်ဝင်ကာ services နှစ်ခုကိုစစ်ပါ။

systemctl is-active postgresql
systemctl is-active pfm-api.service

Expected output:

active
active

Application ကို manual start မလုပ်ဘဲ API ကို ပြန်စမ်းပါ။

curl http://127.0.0.1:3000/categories

Postman မှလည်း GET request ကိုပြန်ပို့ပါ။

GET http://localhost:3001/categories

Reboot မလုပ်မီ ဖန်တီးထားသော data ပြန်ထွက်လာပါက အောက်ပါအချက်များ အောင်မြင်သည်။

PostgreSQL auto-start

API auto-start

Systemd dependency/order

Systemd environment variables

PostgreSQL data persistence

Development Mode နှင့် Production Mode

Development mode တွင် ပုံမှန်အားဖြင့် အောက်ပါ command ကိုသုံးသည်။

bun run start:dev

Development mode သည် TypeScript source code ကို watch လုပ်ပြီး file ပြောင်းလဲသည့်အခါ process ကို reload လုပ်ပေးသည်။ Terminal နှင့် ချိတ်ဆက်ထားလေ့ရှိပြီး production deployment အတွက် မရည်ရွယ်ပါ။

ဤ VM သည် production-style runtime ကိုအသုံးပြုထားသည်။

NODE_ENV=production
Bun က compiled dist/main.js ကို run သည်
Systemd က process ကို manage လုပ်သည်
Failure ဖြစ်ပါက Systemd က restart လုပ်သည်
VM reboot ပြီးပါက Systemd က auto-start လုပ်သည်

ထို့ကြောင့်:

Development mode          = မဟုတ်ပါ
Production runtime mode   = ဟုတ်ပါသည်
Fully production-hardened = မဖြစ်သေးပါ

ဤ setup သည် deployment lab အတွက် သင့်လျော်သည်။ Public production system အတွက် ထပ်မံလိုအပ်နိုင်သည်များ:

Strong database password

Secret manager သို့မဟုတ် protected environment file

Nginx သို့မဟုတ် အခြား reverse proxy

HTTPS/TLS certificate

Firewall rules

Health check နှင့် monitoring

PostgreSQL backup plan

အသုံးများသော Service Commands

API status ကြည့်ရန်:

sudo systemctl status pfm-api.service --no-pager

API service တစ်ခုတည်း restart လုပ်ရန်:

sudo systemctl restart pfm-api.service

API ကို stop/start လုပ်ရန်:

sudo systemctl stop pfm-api.service
sudo systemctl start pfm-api.service

Recent API logs ကြည့်ရန်:

sudo journalctl -u pfm-api.service -n 50 --no-pager

API logs ကို live follow လုပ်ရန်:

sudo journalctl -u pfm-api.service -f

Systemd unit configuration ကိုပြရန်:

sudo systemctl cat pfm-api.service

PostgreSQL status ကြည့်ရန်:

sudo systemctl status postgresql --no-pager

VM တစ်ခုလုံး restart မလုပ်ဘဲ API service ကိုသာ ပြန်စလိုပါက sudo systemctl restart pfm-api.service ကိုသုံးပါ။

ဆရာ့ရှေ့တွင် Demonstration လုပ်ရန် Checklist

1. Architecture ကိုရှင်းပြရန်

PostgreSQL နှင့် NestJS API ကို Systemd က manage လုပ်ထားပါတယ်။ API service က PostgreSQL ကို require လုပ်ပြီး Database service startup ပြီးမှ API start ဖြစ်အောင် သတ်မှတ်ထားပါတယ်။ Project .env file မသုံးဘဲ application environment variables ကို Systemd ထဲမှာ သတ်မှတ်ထားပါတယ်။

2. Services ကိုပြရန်

systemctl is-active postgresql
systemctl is-active pfm-api.service

နှစ်ခုလုံး active ဖြစ်ရမည်။

3. Systemd Configuration ကိုပြရန်

sudo systemctl cat pfm-api.service

ဆရာ့ကို အောက်ပါအချက်များပြနိုင်သည်။

Requires=postgresql.service

After=postgresql.service

User=ubuntu

WorkingDirectory=...

Environment=...

ExecStart=/home/ubuntu/.bun/bin/bun dist/main.js

Restart=on-failure

Shared screen ပေါ်တွင် real database password မပေါ်စေရန် သတိထားပါ။

4. API နှင့် Database ကိုပြရန်

Postman မှ:

GET http://localhost:3001/categories

Category အသစ်တစ်ခု POST လုပ်ပြီး GET request ပြန်ပို့ပါ။ ထို့နောက် database ထဲတွင် တကယ်ဝင်ကြောင်း ပြပါ။

sudo -u postgres psql -d pfm_db -c 'SELECT id, name, total_amount, created_at FROM categories;'

5. Auto-start နှင့် Persistence ကိုပြရန်

sudo reboot

VM ပြန်တက်လာသောအခါ application ကို manual start မလုပ်ဘဲ services နှစ်ခုကိုစစ်၍ Postman GET request ပြန်ပို့ပါ။ Data ကျန်ရှိနေခြင်းက auto-start နှင့် persistence အလုပ်လုပ်ကြောင်း သက်သေပြသည်။

README ကို GitHub ပေါ်တင်ခြင်း

ဤဖိုင်ကို repository ထဲထည့်ပြီး teacher-postgresql branch ပေါ်တွင်ရှိကြောင်း အရင်စစ်ပါ။

git branch --show-current
git status

ပြီးနောက် commit နှင့် push လုပ်ပါ။

git add VM_DEPLOYMENT_README.md
git commit -m "docs: add Burmese Ubuntu VM deployment guide"
git push origin teacher-postgresql

Push မလုပ်မီ အောက်ပါတို့မပါကြောင်း နောက်ဆုံးစစ်ပါ။

Real database password

Private SSH key

Access token

API secret

Personal credential

Repository တွင် မူလ README.md ရှိပြီးသားဖြစ်ပါက မဖျက်ဘဲ အောက်ပါ link ကို ထည့်နိုင်သည်။

## Deployment Guide

[Ubuntu VM Deployment with Bun, PostgreSQL and Systemd](VM_DEPLOYMENT_README.md)

အကျဉ်းချုပ်

Deployment flow အပြည့်အစုံ:

Install PostgreSQL
        ↓
Create pfm_user and pfm_db
        ↓
Fetch and switch to teacher-postgresql branch
        ↓
Install Bun
        ↓
Install dependencies from bun.lock
        ↓
Build the NestJS application
        ↓
Run TypeORM migration
        ↓
Create and enable the Systemd service
        ↓
Test from the VM and Windows Postman
        ↓
Reboot and verify automatic startup and data persistence
