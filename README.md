# Continental Server Panel

**Continental Server Panel (CSP)**, Linux sunucularının merkezi olarak yönetilmesi, yapılandırılması, izlenmesi ve otomatikleştirilmesi amacıyla tasarlanacak **platform bağımsız, modüler ve genişletilebilir bir sunucu yönetim platformudur.**

CSP; işletim sistemi yönetimi, paket yönetimi, servis yönetimi, web sunucuları, PHP, veritabanları, container altyapısı, website yönetimi, SSL, backup, monitoring, job/queue/scheduler/worker sistemleri ve API/CLI erişimini tek bir mimari altında birleştirmeyi hedefler.

> **Bu doküman CSP'nin sıfırdan başlatılan yeni geliştirme sürecinin temel referansıdır.**
>
> Önceki CSP geliştirmeleri, prototipleri, mevcut dosyalar veya çalışan bileşenler bu dokümanda tanımlanan yeni mimarinin parçası olarak kabul edilmez.

---

# 1. Proje Durumu

| Alan | Durum |
|---|---|
| Proje | Continental Server Panel |
| Kısaltma | CSP |
| Geliştirme durumu | **Sıfırdan başlatılıyor** |
| Production | Hazır değil |
| İlk platform | Ubuntu Linux |
| İlk mimari hedef | Linux platform abstraction |
| PHP | 8.5+ hedefleniyor |
| Web UI | İlk aşamada geliştirilmeyecek |
| API | Temel mimariden sonra geliştirilecek |
| CLI | Temel yönetim arayüzlerinden biri olacak |
| Installer | Yeni mimariye göre sıfırdan tasarlanacak |

### Önemli

CSP'nin mevcut kod tabanında daha önce geliştirilmiş herhangi bir bileşenin bulunması, o bileşenin yeni mimaride doğru veya kullanılabilir olduğu anlamına gelmez.

Yeni geliştirme sürecinde:

1. Mimari yeniden tanımlanacaktır.
2. Temel sözleşmeler oluşturulacaktır.
3. Platform abstraction kurulacaktır.
4. Runtime yapısı oluşturulacaktır.
5. CLI ve servis katmanları oluşturulacaktır.
6. Test altyapısı kurulacaktır.
7. Daha sonra üst seviye özellikler geliştirilecektir.

---

# 2. Projenin Temel Amacı

CSP'nin amacı tek bir sunucuyu veya birden fazla sunucuyu merkezi olarak yönetebilen bir platform oluşturmaktır.

Uzun vadeli hedefler:

- İşletim sistemi bilgilerini yönetmek
- Sistem kaynaklarını izlemek
- Paket yönetmek
- Repository yönetmek
- Sistem servislerini yönetmek
- Nginx yönetmek
- PHP ve PHP-FPM yönetmek
- MariaDB/MySQL yönetmek
- Redis yönetmek
- Docker yönetmek
- Website oluşturmak ve yönetmek
- SSL sertifikalarını yönetmek
- Backup ve restore işlemleri yapmak
- Scheduled jobs çalıştırmak
- Queue sistemi sağlamak
- Scheduler çalıştırmak
- Worker çalıştırmak
- Monitoring sağlamak
- Authentication sağlamak
- Authorization sağlamak
- Audit logging sağlamak
- REST API sağlamak
- CLI sağlamak
- Gelecekte Web UI sağlamak
- Gelecekte farklı işletim sistemlerini desteklemek

---

# 3. Temel Mimari Prensip

CSP'nin en önemli prensibi:

> **Application Core işletim sistemi hakkında mümkün olduğunca az bilgiye sahip olmalıdır.**

Uygulama katmanı:


CLI
API
WEB
 │
 ▼
Application / Service Layer
 │
 ▼
Platform Contracts
 │
 ▼
Platform Adapter
 │
 ▼
Operating System


Platforma özgü işlemler Application Core içine dağıtılmayacaktır.

Örneğin:

php
if ($os === 'ubuntu') {
    shell_exec('apt install nginx');
}


yaklaşımı kullanılmayacaktır.

Bunun yerine:


PackageManager
      │
      ▼
PackageManagerInterface
      │
      ├── AptAdapter
      ├── DnfAdapter
      ├── ZypperAdapter
      ├── PacmanAdapter
      └── ApkAdapter


mimarisi kullanılacaktır.

---

# 4. İlk Hedef Platform

İlk geliştirme platformu:


Ubuntu Linux


İlk aşamada hedef:


Ubuntu
 ├── apt
 ├── systemd
 ├── filesystem
 ├── process
 ├── network
 └── security


İlk Ubuntu desteği stabil hale gelmeden diğer dağıtımların geliştirilmesi öncelikli değildir.

---

# 5. Gelecekte Desteklenecek Platformlar

Planlanan platformlar:


Ubuntu
Debian
RHEL
Rocky Linux
AlmaLinux
Fedora
openSUSE
Arch Linux
Alpine Linux
Windows


Platform destek sırası kesin değildir.

Yeni platform ekleme prensibi:


Platform Contract
       │
       ▼
Platform Adapter
       │
       ▼
Platform Tests
       │
       ▼
CI


Yeni bir platform eklemek için Application Core değiştirilmemelidir.

---

# 6. Platform Layer

Platform Layer CSP'nin temel mimari katmanıdır.

Platform Layer'ın görevi işletim sistemi ayrıntılarını üst katmanlardan gizlemektir.

Temel bileşenler:


Platform
├── OS
├── Package Manager
├── Repository
├── Service Manager
├── Filesystem
├── Process
├── Network
├── User
├── Permission
├── Firewall
└── System


---

# 7. Platform Detection

İlk Linux platform tespiti `/etc/os-release` üzerinden yapılacaktır.

Örnek:


OS_FAMILY=debian
OS_ID=ubuntu
OS_VERSION=...
ARCH=amd64
PACKAGE_MANAGER=apt
INIT_SYSTEM=systemd


Platform tespiti sonucunda uygulamanın kullanacağı bir platform profili oluşturulacaktır.

---

# 8. Platform Profile

Hedef yapı:


PlatformProfile
├── osFamily
├── osId
├── osVersion
├── architecture
├── packageManager
├── serviceManager
├── initSystem
├── configPaths
├── binaryPaths
├── logPaths
└── capabilities


Capabilities örnekleri:


nginx
php
php-fpm
mariadb
redis
docker
systemd
firewall
selinux
apparmor


---

# 9. Package Manager

Application Layer gerçek package manager komutlarını bilmeyecektir.

Örneğin:


csp package install nginx


Ubuntu:


apt install nginx


RHEL:


dnf install nginx


Arch:


pacman -S nginx


gibi adapter tarafından uygulanacaktır.

Temel sözleşme:


PackageManagerInterface

install()
remove()
update()
upgrade()
search()
info()
isInstalled()
version()


---

# 10. Service Manager

CSP servis işlemleri için platform bağımsız bir sözleşme sağlayacaktır.

Örneğin:


csp service status nginx
csp service start nginx
csp service stop nginx
csp service restart nginx
csp service reload nginx


Linux/systemd:


systemctl


OpenRC:


rc-service


Windows:


Windows Services / PowerShell


kullanılabilir.

Hedef:


ServiceManagerInterface
├── SystemdAdapter
├── OpenRCAdapter
└── WindowsServiceAdapter


---

# 11. Filesystem Abstraction

Platforma özgü yollar Application Core içerisinde hard-code edilmeyecektir.

Örneğin:


Nginx config
PHP config
PHP-FPM socket
Service files
Logs
Runtime
Temporary files


platform abstraction üzerinden çözülecektir.

Hedef:


FilesystemProvider
├── nginxConfigPath()
├── nginxSitesPath()
├── phpConfigPath()
├── phpFpmPath()
├── servicePath()
├── logPath()
└── runtimePath()


---

# 12. Repository Provider

Repository işlemleri abstraction üzerinden gerçekleştirilecektir.


RepositoryProvider
├── add()
├── remove()
├── enable()
├── disable()
├── refresh()
└── list()


Ubuntu:


APT repository
PPA
Ubuntu repository


RHEL:


DNF repository
RPM repository
EPEL


gibi kaynaklar adapter tarafından yönetilecektir.

---

# 13. Application Core

Application Core platform bağımsız olacaktır.

Temel bileşenler:


Application
Container
Router
Request
Response
Config
Database
Logger
Event Dispatcher
Service Provider
Exception Handler
Validator


Hedef:


Application
├── Container
├── Router
├── Request
├── Response
├── Config
├── Database
├── Logger
├── Events
└── Service Providers


---

# 14. CLI

CSP'nin ilk kullanıcı arayüzlerinden biri CLI olacaktır.

Ana komut:

bash
csp


İlk hedef komut grupları:


csp platform
csp system
csp package
csp service
csp monitor
csp website
csp nginx
csp php
csp mysql
csp redis
csp docker
csp ssl
csp backup
csp job


Örnek:

bash
csp platform info
csp platform detect

csp package install nginx

csp service status nginx
csp service restart nginx


CLI doğrudan OS komutlarına bağlı bir yapı olmayacaktır.

CLI:


CLI
 │
 ▼
Command
 │
 ▼
Service Layer
 │
 ▼
Platform Layer


mimarisini kullanacaktır.

---

# 15. API

API, CLI ile aynı Service Layer'ı kullanacaktır.

Temel mimari:


CLI ─────┐
         │
API ─────┼──► Service Layer
         │
WEB ─────┘
             │
             ▼
       Platform Layer


Hedef API:


/api/v1/


Planlanan alanlar:


/api/v1/auth
/api/v1/server
/api/v1/platform
/api/v1/services
/api/v1/packages
/api/v1/websites
/api/v1/databases
/api/v1/php
/api/v1/nginx
/api/v1/docker
/api/v1/ssl
/api/v1/backups
/api/v1/jobs
/api/v1/monitoring


API ilk geliştirme aşamasında değildir.

---

# 16. Web Panel

Web UI, backend ve platform mimarisi stabil hale gelmeden geliştirilmeyecektir.

Hedef:


Browser
   │
   ▼
Web UI
   │
   ▼
REST API
   │
   ▼
Service Layer
   │
   ▼
Platform Layer
   │
   ▼
Operating System


Web UI hiçbir zaman doğrudan shell veya işletim sistemi API'larına erişmeyecektir.

---

# 17. Runtime

CSP runtime ile installer birbirinden tamamen ayrılacaktır.

Temel kural:

> `install/` runtime dependency olmayacaktır.

Runtime:


lib/


altında bulunacaktır.

Installer:


install/


altında bulunacaktır.

Runtime kodunun installer içerisindeki library'lere bağımlı olması yasaktır.

---

# 18. CSP Root

Tek canonical runtime root kullanılacaktır.

Production:


/opt/csp


Development ortamında repository root kullanılabilir.

Ancak uygulama içerisinde ikinci veya alternatif canonical root mekanizmaları oluşturulmayacaktır.

Temel değişken:

bash
CSP_ROOT


olacaktır.

---

# 19. Job System

Job sistemi CSP'nin temel altyapılarından biri olacaktır.

Hedef lifecycle:


Definition
    │
    ▼
Pending
    │
    ▼
Running
    │
    ├──► Completed
    │
    └──► Failed


Queue:


storage/queue/
├── pending/
├── running/
├── completed/
└── failed/


Job sisteminde ileride:

- Queue locking
- Retry
- Timeout
- Concurrency
- Cancellation
- Priority
- Cleanup
- History retention
- Output limits

desteklenecektir.

---

# 20. Scheduler

Scheduler job üretiminden sorumlu olacaktır.

Temel görevler:

1. Job definition okumak
2. Schedule kontrol etmek
3. Cron expression değerlendirmek
4. Duplicate execution önlemek
5. Queue'ya job eklemek
6. Execution state oluşturmak

İleri aşamalarda:

- Cron parser
- Cron validation
- Timezone
- Duplicate prevention
- Missed job policy
- Retry
- Timeout
- Metrics
- Health
- Graceful shutdown

eklenecektir.

---

# 21. Worker

Worker queue içerisindeki job'ları çalıştıracaktır.

Temel lifecycle:


Pending
   │
   ▼
Running
   │
   ▼
Execute
   │
   ├──► Completed
   │
   └──► Failed


Worker için hedefler:

- Process isolation
- Queue locking
- Timeout
- Retry
- Exit code
- Output limits
- Concurrency
- Health
- Metrics
- Graceful shutdown

---

# 22. Daemon System

Linux tarafında daemon mimarisi systemd üzerinden tasarlanacaktır.

Hedef servisler:


cspd
csp-monitor
csp-scheduler
csp-worker


Ancak daemon geliştirmesi temel runtime ve platform abstraction tamamlanmadan başlatılmayacaktır.

---

# 23. Installer

Installer yeni CSP mimarisine göre sıfırdan tasarlanacaktır.

Installer:


Environment
     │
     ▼
OS Detection
     │
     ▼
Platform Detection
     │
     ▼
Package Manager Detection
     │
     ▼
Service Manager Detection
     │
     ▼
Repository
     │
     ▼
Required Packages
     │
     ▼
CSP Runtime
     │
     ▼
System Integration


Installer prensipleri:

- Idempotent olmalıdır.
- Root kontrolü yapmalıdır.
- Platform detection yapmalıdır.
- Unsupported platform durumunu açıkça belirtmelidir.
- Package manager kontrolü yapmalıdır.
- Repository durumunu kontrol etmelidir.
- Hataları açıkça raporlamalıdır.
- Güvenli command execution kullanmalıdır.
- Runtime koduna installer bağımlılığı oluşturmamalıdır.

---

# 24. Server Management

Sunucu servisleri mümkün olduğunca ortak bir sözleşme kullanacaktır.

Temel işlemler:


install
status
start
stop
restart
reload
configure
validate
logs
uninstall


İlk hedef servisler:


Nginx
PHP
MariaDB
Redis
Docker


---

# 25. Website Management

Hedef CLI:

bash
csp website create example.com
csp website delete example.com
csp website list
csp website status example.com
csp website enable example.com
csp website disable example.com


Website modeli:


Website
├── Domain
├── Document Root
├── Nginx Configuration
├── PHP-FPM
├── SSL
├── Logs
├── Permissions
└── Database


Platform-specific path bilgileri Platform Layer üzerinden sağlanacaktır.

---

# 26. PHP Management

Hedef:

bash
csp php list
csp php install 8.5
csp php remove 8.5
csp php default 8.5
csp php extensions
csp php fpm


PHP abstraction:


PHPService
├── install()
├── remove()
├── versions()
├── defaultVersion()
├── extensions()
├── fpm()
└── validate()


PHP paket isimleri, binary yolları ve FPM servisleri platform adapter tarafından belirlenmelidir.

---

# 27. Nginx Management

Hedef:

bash
csp nginx install
csp nginx status
csp nginx start
csp nginx stop
csp nginx restart
csp nginx reload
csp nginx config-test
csp nginx logs


Nginx platform entegrasyonu:


NginxService
     │
     ▼
Platform Layer
     │
     ├── Package Manager
     ├── Filesystem
     └── Service Manager


---

# 28. Database Management

İlk hedef database server:


MariaDB / MySQL


Hedef:

bash
csp mysql status
csp mysql database list
csp mysql database create
csp mysql database delete
csp mysql user list
csp mysql user create
csp mysql backup
csp mysql restore


Application database abstraction gelecekte farklı database sistemlerini destekleyebilecek şekilde tasarlanacaktır.

---

# 29. Redis

Hedef:

bash
csp redis status
csp redis restart
csp redis health
csp redis memory


Redis yönetimi server management mimarisinin bir parçası olacaktır.

---

# 30. Docker

Hedef:

bash
csp docker status
csp docker ps
csp docker images
csp docker start
csp docker stop
csp docker restart


Gelecekte:


Containers
Images
Networks
Volumes
Compose


yönetilecektir.

---

# 31. SSL

SSL yönetiminde ilk hedef Let's Encrypt entegrasyonudur.

Hedef:

bash
csp ssl issue example.com
csp ssl renew example.com
csp ssl revoke example.com
csp ssl list
csp ssl status example.com


---

# 32. Backup

Planlanan backup türleri:


System Backup
Website Backup
Database Backup
Configuration Backup
Docker Backup
Full Server Backup


Lifecycle:


Create
  │
  ▼
Compress
  │
  ▼
Checksum
  │
  ▼
Store


Storage hedefleri:


Local
Remote
Object Storage


---

# 33. Monitoring

Temel metrikler:


CPU
Memory
Swap
Disk
Network
Load
Processes
Services
PHP-FPM
Nginx
MariaDB
Redis
Docker


Monitoring platform-specific metric provider üzerinden çalışacaktır.

---

# 34. Security

Security CSP'nin temel tasarım gereksinimlerinden biridir.

Özellikle command execution kritik kabul edilir.

Yanlış:

bash
bash -c "$user_input"


Doğru yaklaşım:


User Input
    │
    ▼
Validation
    │
    ▼
Allowlist
    │
    ▼
Command Builder
    │
    ▼
Platform Adapter
    │
    ▼
Executor


Raw user input hiçbir zaman doğrudan shell command olarak çalıştırılmayacaktır.

---

# 35. Authentication

Gelecekte:


Users
Roles
Permissions
Sessions
API Tokens
Password Hashing
Password Reset


desteklenecektir.

Örnek roller:


admin
operator
readonly


Authentication sistemi API ve Web UI geliştirilmeden önce sağlam bir temel olarak tasarlanacaktır.

---

# 36. Authorization

Authorization middleware üzerinden uygulanacaktır.

Hedef:


Authentication
      │
      ▼
Authorization
      │
      ▼
Permission Check
      │
      ▼
Controller


Örnek izinler:


server.read
server.manage
website.manage
database.manage
system.manage


---

# 37. Audit Logging

Kritik işlemler audit log ile kayıt altına alınacaktır.

Örnek alanlar:


User
Action
Resource
IP
Timestamp
Result


Örnek işlem:


admin
website.delete
example.com
<ip>
<timestamp>
success


---

# 38. Storage

Standart runtime storage yapısı:


storage/
├── backups/
├── cache/
├── jobs/
├── logs/
├── queue/
├── run/
├── sessions/
└── tmp/


Runtime dizinleri installer tarafından oluşturulacaktır.

---

# 39. Testing

Test sistemi baştan kurulacaktır.

Hedef yapı:


tests/
├── Unit/
├── Feature/
├── Integration/
├── Platform/
└── System/


Platform testleri özellikle önemlidir.

Örneğin:


tests/Platform/
├── Ubuntu/
├── Debian/
├── RHEL/
├── Fedora/
└── Windows/


Yeni bir platform adapter'ı test edilmeden desteklenen platform olarak kabul edilmeyecektir.

---

# 40. Static Analysis

PHP:


PHPUnit
PHPStan
PHP-CS-Fixer


Bash:


ShellCheck


kullanılacaktır.

CI sistemi ilerleyen aşamada bu kontrolleri otomatik çalıştıracaktır.

---

# 41. Kodlama Standartları

PHP:

- PSR-12
- Strict Types
- Type declarations
- Return types
- Dependency Injection
- SOLID
- Interface-driven design

Bash:

bash
set -Eeuo pipefail


kullanılmalıdır.

Ayrıca:

- Proper quoting
- Explicit error handling
- ShellCheck
- Unsafe `eval` kullanımından kaçınma
- Raw user input çalıştırmama

zorunludur.

---

# 42. Temel Klasör Yapısı

İlk hedeflenen repository yapısı:


continental-server-panel/
│
├── app/
│   ├── Core/
│   ├── Contracts/
│   ├── Controllers/
│   ├── Services/
│   ├── Repositories/
│   ├── Models/
│   ├── Middleware/
│   ├── Validators/
│   ├── Exceptions/
│   └── Support/
│
├── api/
│   └── v1/
│
├── bootstrap/
│
├── cli/
│   └── csp/
│
├── config/
│
├── database/
│   ├── migrations/
│   └── seeders/
│
├── lib/
│
├── platform/
│   ├── contracts/
│   ├── detect/
│   └── linux/
│       ├── common/
│       └── ubuntu/
│
├── install/
│
├── routes/
│
├── storage/
│
├── system/
│
├── templates/
│
├── tests/
│
├── resources/
│
├── public/
│
├── scripts/
│
├── composer.json
├── LICENSE
└── README.md


Bu yapı başlangıç mimarisidir.

İhtiyaç oluştuğunda değiştirilebilir ancak değişiklikler mimari prensiplere uygun olmalıdır.

---

# 43. Geliştirme Sırası

CSP sıfırdan aşağıdaki sırayla geliştirilecektir.

## Faz 0 — Repository Foundation

İlk hedef:

- Repository temizliği
- README
- LICENSE
- Temel klasör yapısı
- Development conventions

Çıktı:


Clean Project Foundation


---

## Faz 1 — Platform Foundation

- OS detection
- Architecture detection
- Platform profile
- Package manager detection
- Service manager detection
- Filesystem abstraction
- Platform contracts
- Ubuntu adapter

Çıktı:


Working Ubuntu Platform Layer


---

## Faz 2 — Runtime Foundation

- CSP_ROOT
- Bootstrap
- Runtime configuration
- Logging
- Error handling
- Command execution
- Process handling
- Runtime storage

Çıktı:


Stable Runtime


---

## Faz 3 — PHP Core

- Application
- Container
- Config
- Request
- Response
- Router
- Exception system
- Validator
- Service Provider
- Logger

Çıktı:


Stable PHP Core


---

## Faz 4 — CLI

- CLI bootstrap
- Command parser
- Command registry
- Command handler
- Arguments
- Options
- Validation
- Output formatter

Çıktı:


Working CSP CLI


---

## Faz 5 — Database

- Connection
- Configuration
- Migration
- Seeder
- Transaction
- Repository abstraction
- Database exceptions

Çıktı:


Stable Database Layer


---

## Faz 6 — Job / Queue

- Job definition
- Queue
- State model
- Execution lifecycle
- Locking
- Retry
- Timeout
- Cleanup
- History

Çıktı:


Reliable Job System


---

## Faz 7 — Scheduler

- Cron parsing
- Schedule validation
- Timezone
- Duplicate prevention
- Missed job policy
- Queue integration
- Graceful shutdown

Çıktı:


Reliable Scheduler


---

## Faz 8 — Worker

- Queue consumption
- Process execution
- Exit code
- Timeout
- Retry
- Concurrency
- Output handling
- Graceful shutdown
- Health

Çıktı:


Reliable Worker


---

## Faz 9 — Service Layer

- Server service abstraction
- Nginx service
- PHP service
- Database service
- Redis service
- Docker service

Çıktı:


Server Management Core


---

## Faz 10 — API

- API bootstrap
- Routing
- Controllers
- Middleware
- Authentication
- Authorization
- Validation
- Error responses
- Pagination
- Rate limiting

Çıktı:


CSP REST API


---

## Faz 11 — Authentication & Authorization

- Users
- Roles
- Permissions
- Login
- Logout
- Sessions
- Password hashing
- API tokens
- Authorization middleware

Çıktı:


Secure Access Layer


---

## Faz 12 — Website Management

- Website model
- Domain
- Document root
- Nginx configuration
- PHP-FPM
- SSL
- Permissions
- Logs

Çıktı:


Website Management


---

## Faz 13 — Backup

- Website backup
- Database backup
- Configuration backup
- Full backup
- Compression
- Checksum
- Restore
- Retention
- Scheduling

---

## Faz 14 — Monitoring

- CPU
- Memory
- Disk
- Network
- Processes
- Services
- PHP-FPM
- Nginx
- MariaDB
- Redis
- Docker
- Historical metrics

---

## Faz 15 — Platform Expansion

Sırasıyla:


Ubuntu
   │
   ├── Debian
   │
   ├── RHEL
   │
   ├── Rocky Linux
   │
   ├── AlmaLinux
   │
   └── Fedora


Daha sonra:


openSUSE
Arch
Alpine
Windows


---

## Faz 16 — Web UI

Web UI ancak:


Runtime
Platform
CLI
Service Layer
Database
API
Authentication
Authorization


yeterince stabil olduktan sonra geliştirilecektir.

---

# 44. Development Philosophy

CSP geliştirilirken şu prensipler korunacaktır:

1. Önce mimari, sonra özellik.
2. Önce abstraction, sonra adapter.
3. Core'a OS bilgisi sızdırılmayacak.
4. Platform-specific kod adapter içinde tutulacak.
5. Runtime ve installer ayrılacak.
6. CLI, API ve Web aynı Service Layer'ı kullanacak.
7. Raw shell execution sınırlandırılacak.
8. Her kritik işlem test edilecek.
9. Yeni platform eklemek Core değişikliği gerektirmeyecek.
10. Çalışıyor olması tek başına doğru mimari anlamına gelmeyecek.
11. Teknik borç README'de "tamamlandı" olarak gösterilmeyecek.
12. Bir özellik test edilmeden stabil kabul edilmeyecek.

---

# 45. Definition of Done

Bir CSP bileşeni ancak aşağıdaki koşullar sağlandığında tamamlanmış kabul edilir:


Implementation
     │
     ▼
Unit Tests
     │
     ▼
Integration Tests
     │
     ▼
Platform Tests
     │
     ▼
Error Handling
     │
     ▼
Logging
     │
     ▼
Security Review
     │
     ▼
Documentation
     │
     ▼
Done


Sadece kodun yazılmış olması "tamamlandı" anlamına gelmez.

---

# 46. Production Readiness

CSP production-ready olarak kabul edilmeden önce en azından:

- Platform abstraction
- Runtime
- Installer
- CLI
- Database
- Job system
- Scheduler
- Worker
- Authentication
- Authorization
- API
- Logging
- Audit logging
- Backup
- Monitoring
- Security
- Testing
- Upgrade mechanism
- Rollback mechanism

alanlarında yeterli stabilite sağlanmalıdır.

---

# 47. İlk Geliştirme Hedefi

CSP'nin sıfırdan geliştirilmesindeki ilk teknik hedef:


Clean Repository
      │
      ▼
Platform Contracts
      │
      ▼
Ubuntu Detection
      │
      ▼
Ubuntu Platform Adapter
      │
      ▼
Runtime Foundation
      │
      ▼
CSP CLI


İlk başarı kriteri büyük bir panel oluşturmak değildir.

İlk başarı kriteri:

> **CSP'nin temiz, test edilebilir ve platform bağımsız bir temel üzerinde çalışmasıdır.**

---

# 48. Son Hedef

Nihai mimari:


                    CONTINENTAL SERVER PANEL
                              │
             ┌────────────────┼────────────────┐
             │                │                │
            CLI              API              WEB
             │                │                │
             └────────────────┼────────────────┘
                              │
                              ▼
                       SERVICE LAYER
                              │
                              ▼
                      APPLICATION CORE
                              │
                              ▼
                     PLATFORM CONTRACTS
                              │
             ┌────────────────┼────────────────┐
             │                │                │
        Linux Adapter    Windows Adapter   Future
             │
       ┌─────┼───────────────┐
       │     │               │
    Ubuntu Debian          RHEL
       │     │               │
      apt   apt            dnf
       │     │               │
   systemd systemd       systemd


CSP'nin uzun vadeli amacı:

> **İşletim sistemi ayrıntılarını üst katmanlardan izole ederek farklı platformları tek bir yönetim modeli altında birleştiren güvenilir bir sunucu yönetim platformu oluşturmaktır.**

---

# 49. Proje Başlangıç Kuralı

Bu README'nin yayınlanmasıyla birlikte CSP geliştirme süreci aşağıdaki varsayımla başlayacaktır:


CSP = NEW PROJECT


Önceki:

- installer kodları
- scheduler kodları
- worker kodları
- job state yapıları
- daemon yapıları
- PHP core
- CLI
- deployment mekanizmaları
- systemd servisleri
- runtime kütüphaneleri
- mevcut testler

yeni mimarinin doğrulanmış parçaları olarak kabul edilmeyecektir.

Gerekli görülen parçalar daha sonra **yeniden tasarlanarak** oluşturulabilir.

---

# 50. İlk Milestone

İlk milestone:

## CSP Foundation 0.1

Hedef:


Repository
    │
    ├── Platform Contracts
    ├── Ubuntu Adapter
    ├── OS Detection
    ├── Runtime Bootstrap
    ├── Configuration
    ├── Logger
    ├── Command Executor
    ├── Basic CLI
    └── Tests


Bu milestone tamamlanmadan:


Scheduler
Worker
Website Management
Web UI
Backup
Monitoring


gibi üst seviye özelliklere geçilmeyecektir.

---

# 51. Özet

CSP sıfırdan şu sırayla kurulacaktır:


1. Repository Foundation
          ↓
2. Platform Contracts
          ↓
3. Ubuntu Adapter
          ↓
4. Runtime
          ↓
5. PHP Core
          ↓
6. CLI
          ↓
7. Database
          ↓
8. Job / Queue
          ↓
9. Scheduler
          ↓
10. Worker
          ↓
11. Service Layer
          ↓
12. API
          ↓
13. Authentication
          ↓
14. Server Management
          ↓
15. Website Management
          ↓
16. Backup
          ↓
17. Monitoring
          ↓
18. Platform Expansion
          ↓
19. Web UI


Bu sıra CSP'nin yeni başlangıç geliştirme planıdır.

**Amaç mevcut sistemi düzeltmek değil, doğru mimariyi baştan kurmaktır.**

---

# Lisans

CSP için lisans kararı proje başlangıç aşamasında ayrıca belirlenecektir.