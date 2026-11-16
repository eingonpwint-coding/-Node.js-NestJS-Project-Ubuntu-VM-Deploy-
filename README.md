# Node.js / NestJS Project ကို Ubuntu VM ပေါ် Deploy လုပ်နည်း

ဒီစာမျက်နှာမှာ Node.js သို့မဟုတ် NestJS project တစ်ခုကို GitHub ကနေ Ubuntu Virtual Machine ပေါ်သို့ deploy လုပ်ပုံကို အဆင့်လိုက် ဖော်ပြထားပါတယ်။

ဒီ Guide ကို အောက်ပါနေရာတွေမှာ အသုံးပြုနိုင်ပါတယ်။

- GitHub `README.md`
- Notion Deployment Note
- နောက်ထပ် Node.js/NestJS project များအတွက် Deployment Checklist
- Error ဖြစ်ချိန်မှာ Troubleshooting Reference

---

# Deployment Flow

Project တစ်ခုကို deploy လုပ်တဲ့အခါ အောက်ပါအစီအစဉ်အတိုင်း လုပ်ပါ။

```text
Project files စစ်ဆေးခြင်း
        ↓
Node.js version နှင့် Package Manager ရွေးချယ်ခြင်း
        ↓
Ubuntu VM ပြင်ဆင်ခြင်း
        ↓
GitHub နှင့် VM ချိတ်ဆက်ခြင်း
        ↓
Project Clone လုပ်ခြင်း
        ↓
Node.js နှင့် Dependencies Install လုပ်ခြင်း
        ↓
Build, Lint နှင့် Test လုပ်ခြင်း
        ↓
Development Mode စမ်းသပ်ခြင်း
        ↓
Production Build လုပ်ခြင်း
        ↓
systemd Service ပြုလုပ်ခြင်း
        ↓
API စမ်းသပ်စစ်ဆေးခြင်း
```

---

# ဒီ Guide မှာ အသုံးပြုထားသော Example Project

```text
Project Name: Personal Finance Manager API
Framework: NestJS
Operating System: Ubuntu Server 24.04 LTS
Node.js Version: 26.13.1
npm Version: 11.20.0
Application Port: 3000
Swagger URL: /docs
Test Endpoint: /categories
```

ကိုယ့် project မှာ သက်ဆိုင်ရာ username၊ repository၊ path နဲ့ port တွေကို ပြောင်းပေးရပါမယ်။

| Placeholder | Example |
|---|---|
| `<VM_USERNAME>` | `eingonpwint` |
| `<GITHUB_USERNAME>` | ကိုယ့် GitHub username |
| `<REPOSITORY>` | `personal-finance-manager-flameops` |
| `<PROJECT_PATH>` | `/home/eingonpwint/personal-finance-manager-flameops` |
| `<APP_NAME>` | `finance-api` |
| `<APP_PORT>` | `3000` |
| `<NODE_VERSION>` | `26.13.1` |

---

# Part 1 — Project ကို နားလည်အောင် အရင်စစ်ခြင်း

## Step 1 — Project Files များကို စစ်ပါ

Software များ install မလုပ်ခင် GitHub repository ထဲမှာ အောက်ပါ files များရှိမရှိ အရင်စစ်ပါ။

```text
README.md
package.json
.nvmrc
.gitignore
.env.example
package-lock.json
bun.lock
yarn.lock
pnpm-lock.yaml
Dockerfile
```

File တစ်ခုချင်းစီရဲ့ အဓိပ္ပာယ်က အောက်ပါအတိုင်းဖြစ်ပါတယ်။

| File | အသုံးပြုပုံ |
|---|---|
| `README.md` | Project install, build, test နှင့် run လုပ်နည်း |
| `package.json` | Dependencies, scripts, Node.js version နှင့် package manager |
| `.nvmrc` | အသုံးပြုရမည့် Node.js version |
| `.env.example` | လိုအပ်သော environment variables |
| `package-lock.json` | npm dependency versions များကို lock လုပ်ထားခြင်း |
| `bun.lock` | Bun dependency versions |
| `yarn.lock` | Yarn dependency versions |
| `pnpm-lock.yaml` | pnpm dependency versions |
| `Dockerfile` | Container ဖြင့် deploy လုပ်ရန် instructions |
| `.gitignore` | GitHub ပေါ် မတင်သင့်သော files များ |

Project folder ထဲမှာ အောက်ပါ commands များဖြင့် စစ်နိုင်ပါတယ်။

```bash
cat README.md
cat package.json
cat .nvmrc
```

`package.json` ထဲမှာ အောက်ပါအပိုင်းတွေကို အဓိက စစ်ဆေးရပါမယ်။

```json
{
  "scripts": {},
  "dependencies": {},
  "devDependencies": {},
  "engines": {},
  "packageManager": ""
}
```

အထူးသဖြင့် အောက်ပါအချက်တွေကို မှတ်ထားပါ။

- ဘယ် Node.js version လိုအပ်လဲ
- ဘယ် package manager အသုံးပြုထားလဲ
- Dependencies install command ကဘာလဲ
- Build command ကဘာလဲ
- Development start command ကဘာလဲ
- Production start command ကဘာလဲ
- Application port ဘယ်လောက်လဲ
- Database လိုအပ်လား
- Environment variables ဘာတွေလိုအပ်လဲ

---

## Step 2 — Package Manager ရွေးချယ်ပါ

Repository ထဲက lock file ကိုကြည့်ပြီး ဘယ် package manager အသုံးပြုရမလဲ ဆုံးဖြတ်နိုင်ပါတယ်။

| Lock File | Package Manager | Install Command |
|---|---|---|
| `package-lock.json` | npm | `npm ci` |
| `bun.lock` | Bun | `bun install --frozen-lockfile` |
| `yarn.lock` | Yarn | `yarn install --immutable` |
| `pnpm-lock.yaml` | pnpm | `pnpm install --frozen-lockfile` |

`package.json` ထဲမှာလည်း အောက်ပါအတိုင်း ရေးထားနိုင်ပါတယ်။

```json
"packageManager": "npm@11.20.0"
```

ဒါဆိုရင် ဒီ project က npm ကို သုံးရမယ်လို့ သတ်မှတ်ထားတာဖြစ်ပါတယ်။

### အရေးကြီးသော Rule

npm, Bun, Yarn နဲ့ pnpm ကို အကြောင်းပြချက်မရှိဘဲ ရောမသုံးပါနဲ့။

Project က သတ်မှတ်ထားတဲ့ package manager တစ်မျိုးတည်းကို အသုံးပြုပါ။

### npm Project ဖြစ်ပြီး `package-lock.json` မရှိရင်

npm project ဖြစ်တယ်ဆိုတာ သေချာမှသာ အောက်ပါ command ဖြင့် lock file ဖန်တီးပါ။

```bash
npm install --package-lock-only
npm ci
```

ဖန်တီးလိုက်တဲ့ `package-lock.json` ကို repository ထဲမှာ commit လုပ်ထားသင့်ပါတယ်။ ဒါမှသာ environment အားလုံးမှာ dependency versions တူတူ install ဖြစ်မှာပါ။

---

# Part 2 — Ubuntu VM ပြင်ဆင်ခြင်း

## Step 3 — Ubuntu VM ဖန်တီးပါ

VirtualBox မှာ small API project တစ်ခုအတွက် အောက်ပါ configuration ကို အသုံးပြုနိုင်ပါတယ်။

| Setting | Recommended Value |
|---|---|
| Operating System | Ubuntu Server 24.04 LTS |
| RAM | 4 GB |
| CPU | 2 Cores |
| Disk | 30 GB Dynamically Allocated |
| Network | NAT |
| OpenSSH | Ubuntu install လုပ်ချိန်မှာ ထည့်သွင်းပါ |
| Username | Normal non-root user |
| Hostname | Project နာမည်နှင့်ဆိုင်သော နာမည် |

Ubuntu installation ပြီးသွားရင် install လုပ်စဉ်က ဖန်တီးထားတဲ့ username နဲ့ password ကိုသုံးပြီး login ဝင်ပါ။

---

## Step 4 — OS-Level Dependencies Install လုပ်ပါ

Ubuntu VM ထဲမှာ package list update လုပ်ပါ။

```bash
sudo apt update
```

Git, curl နဲ့ OpenSSH Server ကို install လုပ်ပါ။

```bash
sudo apt install -y git curl openssh-server
```

Install ဖြစ်မဖြစ် စစ်ဆေးပါ။

```bash
git --version
curl --version
systemctl status ssh
```

SSH service မ run သေးရင် အောက်ပါ command ကို သုံးပါ။

```bash
sudo systemctl enable --now ssh
```

Ubuntu packages အားလုံး update လုပ်ချင်ရင်:

```bash
sudo apt upgrade -y
sudo reboot
```

## Dependency Level များ

| Dependency Level | Examples | အသုံးပြုပုံ |
|---|---|---|
| OS-Level | Git, curl, OpenSSH | VM ပြင်ဆင်ခြင်း၊ GitHub access နှင့် remote login |
| Runtime | Node.js, npm | JavaScript application run ခြင်း |
| Project-Level | NestJS, RxJS, Swagger | Application အလုပ်လုပ်ရန် |
| Development | TypeScript, Jest, ESLint | Build, test, lint နှင့် format |
| Transitive | `glob`, `@parcel/watcher` | တခြား dependencies များက လိုအပ်သော packages |

---

## Step 5 — VirtualBox Port Forwarding ပြုလုပ်ပါ

VM ကို power off လုပ်ထားပြီး VirtualBox မှာ အောက်ပါနေရာသို့ သွားပါ။

```text
VirtualBox
→ Settings
→ Network
→ Adapter 1
→ NAT
→ Advanced
→ Port Forwarding
```

### SSH Port Forwarding Rule

| Field | Value |
|---|---|
| Name | `SSH` |
| Protocol | `TCP` |
| Host IP | `127.0.0.1` |
| Host Port | `2222` |
| Guest IP | Blank |
| Guest Port | `22` |

### Application Port Forwarding Rule

| Field | Value |
|---|---|
| Name | `API` |
| Protocol | `TCP` |
| Host IP | `127.0.0.1` |
| Host Port | `3000` |
| Guest IP | Blank |
| Guest Port | `3000` |

Project က port `3000` မဟုတ်ဘဲ တခြား port အသုံးပြုရင် သက်ဆိုင်ရာ port ကို ပြောင်းပေးပါ။

Host IP ကို `127.0.0.1` ထားခြင်းကြောင့် Windows host computer ကနေသာ application ကို ခေါ်နိုင်မှာဖြစ်ပါတယ်။

---

## Step 6 — Windows PowerShell ကနေ VM ထဲ SSH ဖြင့် ဝင်ပါ

Windows PowerShell မှာ:

```powershell
ssh -p 2222 <VM_USERNAME>@127.0.0.1
```

ဥပမာ:

```powershell
ssh -p 2222 eingonpwint@127.0.0.1
```

Login ဝင်ပြီးရင် prompt က အောက်ပါအတိုင်း မြင်ရနိုင်ပါတယ်။

```text
eingonpwint@finance-api-vm:~$
```

ဒီ prompt ကိုမြင်ရင် အခုရိုက်မယ့် Linux commands တွေက Ubuntu VM အတွင်းမှာ run မှာဖြစ်ပါတယ်။

---

# Part 3 — GitHub နှင့် VM ချိတ်ဆက်ခြင်း

## Step 7 — VM ထဲမှာ GitHub SSH Key ဖန်တီးပါ

Ubuntu VM ထဲမှာ အောက်ပါ command ကို run ပါ။

```bash
ssh-keygen -t ed25519 -C "PROJECT-VM"
```

Save location မေးရင် default location ကို အသုံးပြုဖို့ `Enter` နှိပ်ပါ။

Public key ကိုပြရန်:

```bash
cat ~/.ssh/id_ed25519.pub
```

GitHub မှာ အောက်ပါနေရာကို သွားပါ။

```text
GitHub Profile
→ Settings
→ SSH and GPG keys
→ New SSH key
```

`cat ~/.ssh/id_ed25519.pub` က ပြတဲ့ public key စာကြောင်းအပြည့်ကို GitHub ထဲ paste လုပ်ပါ။

ပြီးရင် VM ကနေ GitHub connection ကို စမ်းပါ။

```bash
ssh -T git@github.com
```

အောင်မြင်ရင် အောက်ပါပုံစံနဲ့ message မြင်ရပါမယ်။

```text
Hi <GITHUB_USERNAME>! You've successfully authenticated...
```

### သတိပြုရန်

Public key:

```text
~/.ssh/id_ed25519.pub
```

ကို GitHub မှာ ထည့်ရပါတယ်။

Private key:

```text
~/.ssh/id_ed25519
```

ကို ဘယ်သူ့ကိုမှ မပေးရပါဘူး။ GitHub ပေါ်လည်း မတင်ရပါဘူး။

---

## Step 8 — GitHub Repository Clone လုပ်ပါ

GitHub repository မှာ:

```text
Code → SSH
```

ကိုရွေးပြီး SSH URL ကို copy လုပ်ပါ။

VM ထဲမှာ:

```bash
git clone git@github.com:<GITHUB_USERNAME>/<REPOSITORY>.git
```

Project folder ထဲဝင်ပါ။

```bash
cd <REPOSITORY>
```

Project ကို စစ်ဆေးပါ။

```bash
ls -la
git remote -v
git status
```

### ဖြစ်နိုင်သော Git Clone Errors

| Error | ဖြစ်နိုင်သောအကြောင်းရင်း |
|---|---|
| `Permission denied (publickey)` | GitHub မှာ SSH public key မထည့်ရသေး |
| `Repository not found` | Repository URL မှား သို့မဟုတ် access မရှိ |
| `Could not resolve hostname` | Internet သို့မဟုတ် DNS ပြဿနာ |
| `destination path already exists` | နာမည်တူ project folder ရှိပြီးသား |

---

# Part 4 — Node.js Runtime ပြင်ဆင်ခြင်း

## Step 9 — NVM Install လုပ်ပါ

NVM က project တစ်ခုချင်းစီအတွက် လိုအပ်တဲ့ Node.js version ကို install လုပ်ပြီး ပြောင်းသုံးနိုင်အောင် ကူညီပေးပါတယ်။

NVM install လုပ်ရန်:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.41.0/install.sh | bash
```

NVM ကို current terminal မှာ load လုပ်ပါ။

```bash
source ~/.bashrc
```

NVM version စစ်ပါ။

```bash
nvm --version
```

---

## Step 10 — Project လိုအပ်သော Node.js Version Install လုပ်ပါ

Project folder ထဲမှာ `.nvmrc` ရှိရင်:

```bash
cat .nvmrc
nvm install
nvm use
```

Node.js နဲ့ npm version စစ်ပါ။

```bash
node -v
npm -v
```

ဥပမာ result:

```text
v26.13.1
11.20.0
```

နောက်တစ်ကြိမ် terminal အသစ်ဖွင့်ပြီး project folder ထဲပြန်ဝင်တဲ့အခါ:

```bash
cd <PROJECT_PATH>
nvm use
```

လုပ်လိုက်ရင် `.nvmrc` ထဲမှာ သတ်မှတ်ထားတဲ့ Node.js version ကို ရွေးပေးပါမယ်။

### `.nvmrc` မရှိရင်

`package.json` ရဲ့ `engines` နဲ့ `README.md` ကို စစ်ပါ။

ပြီးရင် လိုအပ်တဲ့ version ကို install လုပ်ပါ။

```bash
nvm install <NODE_VERSION>
nvm use <NODE_VERSION>
```

### `nvm: command not found` ဖြစ်ရင်

```bash
source ~/.bashrc
```

လုပ်ပြီး:

```bash
nvm --version
```

ပြန်စမ်းပါ။

---

# Part 5 — Project Configuration နှင့် Dependencies

## Step 11 — Environment Variables စစ်ပါ

Project folder ထဲမှာ hidden files အပါအဝင် files အားလုံးကြည့်ပါ။

```bash
ls -la
```

`.env.example` ရှိရင် `.env` file ဖန်တီးပါ။

```bash
cp .env.example .env
nano .env
```

Project ပေါ်မူတည်ပြီး အောက်ပါ variables တွေလိုအပ်နိုင်ပါတယ်။

```dotenv
PORT=3000
DATABASE_URL=
API_KEY=
JWT_SECRET=
REDIS_URL=
```

`.env` file ကို Git က ignore လုပ်ထားမထား စစ်ပါ။

```bash
git check-ignore .env
```

Result မှာ:

```text
.env
```

လို့ပြရင် `.env` ကို Git ignore လုပ်ထားတာ မှန်ပါတယ်။

### GitHub ပေါ် မတင်သင့်သောအရာများ

- Password
- API Key
- Access Token
- JWT Secret
- Database Password
- Private SSH Key
- Production `.env` file

ဒီ Personal Finance Manager API မှာ လက်ရှိအနေအထားအရ:

```text
Default Port: 3000
Database: မရှိ
Data Storage: In-memory
Environment file: လိုအပ်မှုမရှိသေး
```

ဒါပေမယ့် နောက်ပိုင်း code ပြောင်းလဲမှုရှိနိုင်တာကြောင့် လက်ရှိ `README.md` နဲ့ `package.json` ကို အမြဲပြန်စစ်ပါ။

---

## Step 12 — Project Dependencies Install လုပ်ပါ

Project ရဲ့ package manager နှင့် ကိုက်ညီသော command တစ်မျိုးတည်းကို သုံးပါ။

### npm Project

`package-lock.json` ရှိရင်:

```bash
npm ci
```

npm project ဖြစ်တာ သေချာပြီး `package-lock.json` မရှိရင်:

```bash
npm install --package-lock-only
npm ci
```

### Bun Project

```bash
bun install --frozen-lockfile
```

### Yarn Project

```bash
corepack enable
yarn install --immutable
```

### pnpm Project

```bash
corepack enable
pnpm install --frozen-lockfile
```

Node.js dependencies install ဖြစ်မဖြစ် စစ်ရန်:

```bash
ls node_modules
```

## `npm ci` နှင့် `npm install` ကွာခြားချက်

| Command | အသုံးပြုသင့်သောနေရာ |
|---|---|
| `npm ci` | Deployment နှင့် clean, reproducible installation |
| `npm install` | Dependency အသစ်ထည့်ခြင်း သို့မဟုတ် lock file ဖန်တီးခြင်း |

အောက်ပါ command ကို အကြောင်းပြချက်မရှိဘဲ မလုပ်သင့်ပါ။

```bash
npm audit fix --force
```

ဒီ command က dependency versions တွေကို အတင်းပြောင်းနိုင်တာကြောင့် project ပျက်သွားနိုင်ပါတယ်။

---

# Project Dependencies များကို နားလည်ခြင်း

## Production Dependencies

ဒါတွေက application run ဖို့ တကယ်လိုအပ်တဲ့ packages တွေဖြစ်ပါတယ်။

| Package | အသုံးပြုပုံ |
|---|---|
| `@nestjs/common` | NestJS decorators, pipes, guards နှင့် common functions |
| `@nestjs/core` | NestJS application ရဲ့ အဓိက runtime |
| `@nestjs/platform-express` | Express-based HTTP server |
| `@nestjs/swagger` | Swagger/OpenAPI documentation |
| `class-validator` | Request DTO validation |
| `class-transformer` | Request data ကို typed objects ပြောင်းခြင်း |
| `reflect-metadata` | NestJS decorators အတွက် metadata |
| `rxjs` | NestJS အသုံးပြုသော reactive programming library |

ဥပမာ `@nestjs/core` မရှိရင် application တစ်ခုလုံး start လုပ်လို့မရပါဘူး။

```text
Cannot find module '@nestjs/core'
```

`@nestjs/swagger` မရှိရင် Swagger documentation ကို run လို့မရနိုင်ပါဘူး။

```text
Cannot find module '@nestjs/swagger'
```

---

## Development Dependencies

ဒါတွေက development, build, format နဲ့ testing အတွက် အသုံးပြုတဲ့ packages တွေဖြစ်ပါတယ်။

| Package | အသုံးပြုပုံ |
|---|---|
| `@nestjs/cli` | `nest build`, `nest start` commands |
| `@nestjs/schematics` | Module, controller, service files generate လုပ်ရန် |
| `@nestjs/testing` | NestJS automated testing |
| `typescript` | TypeScript ကို JavaScript ပြောင်းရန် |
| `jest` | Automated testing framework |
| `ts-jest` | TypeScript tests ကို Jest နဲ့ run ရန် |
| `supertest` | HTTP API endpoints testing |
| `eslint` | Code quality စစ်ဆေးရန် |
| `typescript-eslint` | TypeScript code ကို ESLint ဖြင့် စစ်ရန် |
| `prettier` | Code formatting |
| `@types/*` | TypeScript type definitions |

`@nestjs/cli` မရှိရင်:

```text
nest: command not found
```

TypeScript မရှိရင်:

```text
tsc: command not found
```

Jest မရှိရင်:

```text
jest: command not found
```

ဖြစ်နိုင်ပါတယ်။

---

## Transitive Dependencies

ကိုယ်က `package.json` ထဲမှာ တိုက်ရိုက်မထည့်ထားပေမယ့် တခြား package တစ်ခုက လိုအပ်လို့ အလိုအလျောက် install ဖြစ်လာတဲ့ packages တွေဖြစ်ပါတယ်။

ဥပမာ:

```text
glob
@parcel/watcher
@scarf/scarf
```

ဒါကြောင့် `package.json` ထဲမှာ direct dependencies အရေအတွက် နည်းပေမယ့်:

```text
added 583 packages
```

လို့ပြနိုင်ပါတယ်။

Dependency tree ကြည့်ရန်:

```bash
npm list
```

Package တစ်ခုကို ဘာကြောင့် install လုပ်ထားတာလဲ စစ်ရန်:

```bash
npm explain <PACKAGE_NAME>
```

ဥပမာ:

```bash
npm explain glob
```

---

# Part 6 — Code Quality နှင့် Application Testing

## Step 13 — Project Scripts စစ်ပါ

Project မှာ run လို့ရတဲ့ scripts တွေကိုကြည့်ရန်:

```bash
npm run
```

NestJS project တစ်ခုမှာ အောက်ပါ scripts တွေရှိနိုင်ပါတယ်။

```bash
npm run build
npm run typecheck
npm run lint
npm test
npm run format:check
```

`package.json` ထဲမှာ တကယ်ရှိတဲ့ scripts တွေကိုသာ run ပါ။

ဒီ project အတွက် scripts တွေရှိရင်:

```bash
npm run build
npm run typecheck
npm run lint
npm test
npm run format:check
```

အားလုံး pass ဖြစ်သင့်ပါတယ်။

Error တက်ရင်:

1. နောက်ဆုံး error summary ကိုသာမကြည့်ဘဲ ပထမဆုံး error ကိုကြည့်ပါ။
2. Warning နဲ့ Error ကို ခွဲကြည့်ပါ။
3. ပထမဆုံး root cause ကို အရင်ဖြေရှင်းပါ။
4. Fail ဖြစ်တဲ့ command ကို ပြန် run ပါ။
5. ပြီးရင် checks အားလုံးကို ပြန် run ပါ။

---

## Step 14 — Development Mode ဖြင့် စမ်းပါ

Development server start လုပ်ရန်:

```bash
npm run start:dev
```

NestJS application start အောင်မြင်ရင် အောက်ပါစာသားမျိုး တွေ့ရပါမယ်။

```text
Found 0 errors
Nest application successfully started
```

Application foreground မှာ run နေတာကြောင့် terminal prompt ပြန်မပေါ်တာ ပုံမှန်ဖြစ်ပါတယ်။

Application ကို ရပ်ရန်:

```text
Ctrl + C
```

---

## Step 15 — VM အတွင်းကနေ API အရင်စမ်းပါ

Application ကို run ထားပြီး Windows PowerShell အသစ်တစ်ခုဖွင့်ပါ။

```powershell
ssh -p 2222 <VM_USERNAME>@127.0.0.1
```

VM ထဲဝင်ပြီး:

```bash
curl http://127.0.0.1:3000/categories
```

စမ်းပါ။

ဒီ command အလုပ်လုပ်ရင်:

- Application က run နေတယ်
- Port `3000` က listen လုပ်နေတယ်
- Endpoint က response ပြန်ပေးနိုင်တယ်

ဆိုတာ သိနိုင်ပါတယ်။

---

## Step 16 — Windows Browser သို့မဟုတ် Postman ကနေ စမ်းပါ

Swagger URL:

```text
http://127.0.0.1:3000/docs
```

Postman Request:

```http
GET http://127.0.0.1:3000/categories
```

Postman မှာ:

1. Method dropdown ကနေ `GET` ရွေးပါ။
2. URL box ထဲမှာ URL တစ်ခုတည်း ထည့်ပါ။
3. URL box ထဲမှာ `GET http://...` လို့ မရေးပါနဲ့။

### Connection Problem ကို ခွဲစစ်နည်း

VM အတွင်းက `curl` မရရင်:

- Application run နေမနေစစ်ပါ
- Application logs စစ်ပါ
- Port listen လုပ်နေမနေစစ်ပါ

VM အတွင်းက `curl` ရပြီး Windows ကနေ မရရင်:

- VirtualBox port forwarding စစ်ပါ
- Host IP နဲ့ port စစ်ပါ
- Postman URL စစ်ပါ

---

# Part 7 — Production Deployment

## Step 17 — Development Server ကို ရပ်ပါ

Development server run နေတဲ့ terminal မှာ:

```text
Ctrl + C
```

နှိပ်ပါ။

Port `3000` ကို တခြား process တစ်ခုသုံးနေမနေ စစ်ပါ။

```bash
sudo ss -ltnp | grep :3000
```

Development server နဲ့ production server ကို port တစ်ခုတည်းမှာ တစ်ချိန်တည်း run လို့မရပါဘူး။

---

## Step 18 — Production Build လုပ်ပါ

Project ကို build လုပ်ရန်:

```bash
npm run build
```

Build output ကို စစ်ပါ။

```bash
ls dist
```

ဒီ project မှာ အောက်ပါ file ရှိသင့်ပါတယ်။

```text
dist/main.js
```

Build မလုပ်ထားဘဲ production start လုပ်ရင်:

```text
Cannot find module 'dist/main.js'
```

ဆိုတဲ့ error တက်နိုင်ပါတယ်။

---

## Step 19 — Production Mode ကို Manual စမ်းပါ

```bash
npm run start:prod
```

Application start အောင်မြင်ရင်:

```text
Nest application successfully started
```

လို့ပြနိုင်ပါတယ်။

VM အတွင်းကနေ API စမ်းပါ။

```bash
curl http://127.0.0.1:3000/categories
```

Windows Postman ကနေလည်း ပြန်စမ်းပါ။

```http
GET http://127.0.0.1:3000/categories
```

Manual production test အောင်မြင်မှ systemd service ပြုလုပ်ပါ။

---

# Part 8 — Application ကို အမြဲ Run နေစေရန်

## Step 20 — systemd Service ပြုလုပ်ပါ

systemd service ပြုလုပ်ထားရင်:

- SSH session ပိတ်သွားလည်း API ဆက် run နေမယ်
- VM reboot ဖြစ်ရင် API အလိုအလျောက် start ဖြစ်မယ်
- Application crash ဖြစ်ရင် ပြန် start လုပ်နိုင်မယ်
- Logs ကို `journalctl` နဲ့ ကြည့်နိုင်မယ်

### Node.js Path ကိုရှာပါ

Project folder ထဲဝင်ပြီး Node.js version ရွေးပါ။

```bash
cd <PROJECT_PATH>
nvm use
which node
```

ဥပမာ result:

```text
/home/eingonpwint/.nvm/versions/node/v26.13.1/bin/node
```

ဒီ path ကို အတိအကျ မှတ်ထားပါ။

---

## systemd Service File ဖန်တီးပါ

```bash
sudo nano /etc/systemd/system/<APP_NAME>.service
```

ဥပမာ:

```bash
sudo nano /etc/systemd/system/finance-api.service
```

အောက်ပါ configuration ကို paste လုပ်ပြီး username နဲ့ path တွေကို ကိုယ့် project အလိုက် ပြောင်းပါ။

```ini
[Unit]
Description=Personal Finance Manager API
After=network.target

[Service]
Type=simple
User=<VM_USERNAME>
WorkingDirectory=<PROJECT_PATH>
Environment=NODE_ENV=production
Environment=PORT=3000
ExecStart=/home/<VM_USERNAME>/.nvm/versions/node/v<NODE_VERSION>/bin/node <PROJECT_PATH>/dist/main.js
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

ဒီ project အတွက် concrete example:

```ini
[Unit]
Description=Personal Finance Manager API
After=network.target

[Service]
Type=simple
User=eingonpwint
WorkingDirectory=/home/eingonpwint/personal-finance-manager-flameops
Environment=NODE_ENV=production
Environment=PORT=3000
ExecStart=/home/eingonpwint/.nvm/versions/node/v26.13.1/bin/node /home/eingonpwint/personal-finance-manager-flameops/dist/main.js
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### Nano မှာ Save လုပ်နည်း

```text
Ctrl + O
Enter
Ctrl + X
```

---

## Service ကို Enable နှင့် Start လုပ်ပါ

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now finance-api
```

Service status စစ်ပါ။

```bash
sudo systemctl status finance-api
```

Live logs ကြည့်ရန်:

```bash
journalctl -u finance-api -f
```

နောက်ဆုံး log 100 ကြောင်းကြည့်ရန်:

```bash
journalctl -u finance-api -n 100 --no-pager
```

### Service Control Commands

```bash
sudo systemctl start finance-api
sudo systemctl stop finance-api
sudo systemctl restart finance-api
sudo systemctl status finance-api
```

---

## Reboot ပြီးလည်း Auto Start ဖြစ်မဖြစ် စစ်ပါ

VM ကို reboot လုပ်ပါ။

```bash
sudo reboot
```

VM ပြန်တက်လာရင် SSH ပြန်ဝင်ပါ။

```powershell
ssh -p 2222 <VM_USERNAME>@127.0.0.1
```

Service status စစ်ပါ။

```bash
sudo systemctl status finance-api
```

API စမ်းပါ။

```bash
curl http://127.0.0.1:3000/categories
```

Service status က `active (running)` ဖြစ်ပြီး API response ပြန်လာရင် deployment အောင်မြင်ပါတယ်။

---

# Part 9 — Code Update အသစ် Deploy လုပ်နည်း

## Step 21 — GitHub က Code အသစ် Pull လုပ်ပြီး Deploy လုပ်ခြင်း

နောက်တစ်ကြိမ် GitHub repository မှာ code update ရှိရင်:

```bash
cd <PROJECT_PATH>
git status
git pull
nvm use
npm ci
npm run build
sudo systemctl restart <APP_NAME>
sudo systemctl status <APP_NAME>
```

ဒီ project အတွက်:

```bash
cd ~/personal-finance-manager-flameops
git status
git pull
nvm use
npm ci
npm run build
sudo systemctl restart finance-api
sudo systemctl status finance-api
```

API ပြန်စမ်းပါ။

```bash
curl http://127.0.0.1:3000/categories
```

Logs စစ်ပါ။

```bash
journalctl -u finance-api -n 100 --no-pager
```

### သတိပြုရန်

`git status` မှာ local changes ရှိနေရင် `git pull` မလုပ်ခင် အဲဒီ changes တွေကို သေချာစစ်ပါ။

လိုအပ်တဲ့ changes တွေကို အမှတ်မထင် overwrite မလုပ်ပါနဲ့။

---

# Part 10 — Git Commit Checklist

Commit မလုပ်ခင်:

```bash
git status
git diff
```

အောက်ပါ files နဲ့ data များကို ပုံမှန်အားဖြင့် GitHub ပေါ် မတင်သင့်ပါ။

```text
node_modules/
.env
dist/
Private SSH keys
Passwords
API keys
Access tokens
Temporary logs
```

npm lock file ကို မှန်ကန်စွာ ဖန်တီးထားရင်:

```bash
git add package-lock.json
git commit -m "Add npm dependency lock file"
git push
```

Push မလုပ်ခင်:

```bash
git status
```

နဲ့ ဘယ် files တွေ GitHub ပေါ် တင်တော့မလဲ ပြန်စစ်ပါ။

---

# Part 11 — Troubleshooting

API ချိတ်မရရင် အောက်ပါအစီအစဉ်အတိုင်း စစ်ပါ။

```text
1. VM run နေလား
2. Windows ကနေ SSH ဝင်လို့ရလား
3. Application သို့မဟုတ် Service run နေလား
4. Port 3000 listen လုပ်နေလား
5. VM အတွင်းက curl အလုပ်လုပ်လား
6. VirtualBox port forwarding မှန်လား
7. Windows browser/Postman URL မှန်လား
```

## Diagnostic Commands

```bash
sudo systemctl status finance-api
journalctl -u finance-api -n 100 --no-pager
sudo ss -ltnp | grep :3000
curl http://127.0.0.1:3000/categories
```

## Common Errors

| Error | ဖြစ်နိုင်သောအကြောင်းရင်း | ဖြေရှင်းနည်း |
|---|---|---|
| `Permission denied (publickey)` | GitHub SSH key ပြဿနာ | Public key ကို GitHub မှာ register လုပ်ပါ |
| `nvm: command not found` | NVM မ load ရသေး | `source ~/.bashrc` လုပ်ပါ |
| `EBADENGINE` | Node.js/npm version မှား | `nvm use` လုပ်ပြီး `.nvmrc` စစ်ပါ |
| `npm ci` lock-file error | Lock file မရှိ သို့မဟုတ် မကိုက် | Package manager နှင့် lock file ပြန်စစ်ပါ |
| `nest: command not found` | Dependencies မ install ရသေး | Project folder ထဲမှာ `npm ci` လုပ်ပါ |
| `Cannot find module '@nestjs/core'` | Runtime dependencies မရှိ | `npm ci` ပြန်လုပ်ပြီး install errors စစ်ပါ |
| `Cannot find module 'dist/main.js'` | Production build မရှိ | `npm run build` လုပ်ပါ |
| `EADDRINUSE` | Port ကို တခြား process သုံးနေ | `sudo ss -ltnp \| grep :3000` ဖြင့်စစ်ပါ |
| `ECONNREFUSED` | Application က port မှာ listen မလုပ် | Service status နဲ့ logs စစ်ပါ |
| VM `curl` ရပြီး Windows က မရ | Port forwarding မှား | VirtualBox NAT rules စစ်ပါ |
| HTTP `400` | Request body သို့မဟုတ် validation မှား | DTO နှင့် Swagger schema စစ်ပါ |
| HTTP `404` | Route သို့မဟုတ် ID မှား | Endpoint URL စစ်ပါ |
| HTTP `409` | Duplicate data | Existing records နဲ့ uniqueness rules စစ်ပါ |

---

## Node.js Version ပြောင်းပြီး Service မတက်ရင်

NVM က Node.js version တစ်ခုချင်းစီကို folder တစ်ခုစီမှာ install လုပ်ပါတယ်။

ဒါကြောင့် Node.js version upgrade လုပ်လိုက်ရင် systemd service ထဲက `ExecStart` path အဟောင်းဖြစ်သွားနိုင်ပါတယ်။

Node.js path အသစ်ရှာပါ။

```bash
nvm use
which node
```

Service file ပြန်ဖွင့်ပါ။

```bash
sudo nano /etc/systemd/system/finance-api.service
```

`ExecStart` မှာ Node.js path အသစ်ပြောင်းပြီး:

```bash
sudo systemctl daemon-reload
sudo systemctl restart finance-api
sudo systemctl status finance-api
```

လုပ်ပါ။

---

# Part 12 — အသုံးများသော Commands အတိုချုပ်

## Daily Development

```bash
cd <PROJECT_PATH>
nvm use
npm ci
npm run start:dev
```

## Manual Production Test

```bash
cd <PROJECT_PATH>
nvm use
npm ci
npm run build
npm run start:prod
```

## Production Update

```bash
cd <PROJECT_PATH>
git status
git pull
nvm use
npm ci
npm run build
sudo systemctl restart <APP_NAME>
sudo systemctl status <APP_NAME>
```

## API Health Check

```bash
curl http://127.0.0.1:<APP_PORT>/categories
```

## Service Diagnostics

```bash
sudo systemctl status <APP_NAME>
journalctl -u <APP_NAME> -n 100 --no-pager
sudo ss -ltnp | grep :<APP_PORT>
```

## VM ကို လုံခြုံစွာပိတ်ရန်

```bash
sudo poweroff
```

---

# Final Deployment Checklist

- [ ] `README.md` ကိုဖတ်ပြီးပြီ
- [ ] `package.json` ကိုစစ်ပြီးပြီ
- [ ] Node.js version ကိုသိပြီးပြီ
- [ ] Package manager နဲ့ lock file ကိုသေချာပြီ
- [ ] Environment variables ကိုစစ်ပြီးပြီ
- [ ] Database သို့မဟုတ် external services လိုမလို သိပြီးပြီ
- [ ] Ubuntu VM ပြင်ဆင်ပြီးပြီ
- [ ] SSH access စမ်းပြီးပြီ
- [ ] GitHub public key register လုပ်ပြီးပြီ
- [ ] Repository clone လုပ်ပြီးပြီ
- [ ] Correct Node.js version install/use လုပ်ပြီးပြီ
- [ ] Project dependencies install လုပ်ပြီးပြီ
- [ ] Build, lint, typecheck နဲ့ tests pass ဖြစ်ပြီးပြီ
- [ ] Development mode စမ်းပြီးပြီ
- [ ] VM အတွင်း `curl` ဖြင့်စမ်းပြီးပြီ
- [ ] Windows browser/Postman ဖြင့်စမ်းပြီးပြီ
- [ ] Production build လုပ်ပြီးပြီ
- [ ] Production mode ကို manual စမ်းပြီးပြီ
- [ ] systemd service ဖန်တီးပြီးပြီ
- [ ] VM reboot ပြီး service auto-start ဖြစ်မဖြစ် စမ်းပြီးပြီ
- [ ] Secrets နဲ့ private keys တွေ GitHub ပေါ် မပါကြောင်းစစ်ပြီးပြီ

---

# Project-Specific Notes

ကိုယ့် project ရဲ့ အချက်အလက်တွေကို အောက်မှာ မှတ်တမ်းတင်နိုင်ပါတယ်။

```text
Project Name:
GitHub Repository:
Git Branch:
VM Hostname:
VM Username:
Project Path:
Node.js Version:
Package Manager:
Package Manager Version:
Application Port:
Service Name:
Health Check URL:
Swagger URL:
Database:
External Services:
Environment Variables:
Deployment Date:
Last Updated Date:
Verified By:
```

---

# အဓိက မှတ်ထားရမည့် Deployment အစီအစဉ်

> **Project instructions ဖတ်ပါ → Runtime version ရွေးပါ → Package manager ရွေးပါ → Environment configure လုပ်ပါ → Dependencies install လုပ်ပါ → Build/Test လုပ်ပါ → Development mode စမ်းပါ → Production build စမ်းပါ → systemd service ပြုလုပ်ပါ → API ကို အတည်ပြုပါ။**
