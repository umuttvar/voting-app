# Voting app on Kubernetes
A voting app deployed on Kubernetes with 5 microservices. Jenkins run checks and ArgoCD handles automatic deployments.

##Architecture
- **Vote** - web UI where users voting
- **Redis** - temporary queue that holds incoming votes
- **Worker** - reads votes from redis, writes them to PostgreSQL service
- **PostgreSQL** - database
- **Result** - shows voting results
- **Prometheus** — monitoring

##CI/CD 
- **ArgoCD** watches the `k8s/` folder on GitHub and automatically
  syncs any change to the cluster — no manual `kubectl apply`
  needed after the initial setup

## What I built

- Wrote all Kubernetes files (Deployment, Service, StatefulSet, Secret) from scratch in `k8s/`
- Used a StatefulSet with `volumeClaimTemplates` for PostgreSQL to guarantee persistent storage across Pod restarts
- Used Kubernetes Secrets for database credentials, injected via `envFrom`
- Connected services purely through Kubernetes DNS (Service names), no hardcoded IPs
- Deployed everything to a local `kind` cluster with `kubectl apply -f k8s/`
- - Added a Jenkins pipeline for basic manifest validation


## Debugging highlights

Real issues I hit and resolved while deploying:

- **Disk full (ENOSPC)** — cleaned up unused Docker images/volumes with `docker system prune`
- **StatefulSet CrashLoopBackOff** — traced to a stale PersistentVolumeClaim, fixed by deleting the Pod and PVC and letting the StatefulSet recreate them
- **Worker stuck on "Waiting for db"** — used `kubectl logs`, `pg_isready`, and `getent hosts` to rule out network/DNS issues, then found the root cause: a password mismatch between my Secret and the value the worker image expects (found by reading the original project's `docker-stack.yml`)
- **Stuck PVC in "Terminating"** — removed a leftover finalizer with `kubectl patch`

## Tech stack

Kubernetes · Docker · Jenkins · ArgoCD · Prometheus · PostgreSQL · Redis

----------------------------------------------------------------------------------------------------------------------------

# Kubernetes'te Oylama Uygulaması

Kubernetes üzerinde çalışan, çok servisli bir oylama uygulaması. 
5 container'lı servisten oluşuyor (Vote, Redis, Worker, PostgreSQL, Result)
Tüm servisler DockerHub üzerindeki voting-app image'larından çekildi, Dockerfile yazılmadı. 

## Mimari

- **Vote** - kullanıcıların oy verdiği Web arayüzü
- **Redis** — gelen oyları geçiçi olarak tutan kuyruk
- **Worker** — Redis'ten oyları okuyup PostgreSQL'e yazan servis
- **PostgreSQL** — kalıcı veri deposu
- **Result** — oylama sonuçlarının gösterildiği arayüz
- **Prometheus** — izleme (monitoring)

## CI/CD

- **Jenkins**, her push'ta Kubernetes manifest'lerini doğruluyor
- **ArgoCD**, GitHub'daki `k8s/` klasörünü izliyor, herhangi bir 
  değişikliği otomatik olarak cluster'a uyguluyor (GitOps) — ilk 
  kurulumdan sonra elle `kubectl apply` yapmaya gerek yok

## Neler yapıldı

- Tüm Kubernetes dosyalarını (Deployment, Service, StatefulSet, Secret) sıfırdan, `k8s/` klasöründe yazdım
- PostgreSQL için, Pod yeniden başlasa bile verinin kaybolmaması amacıyla, `volumeClaimTemplates` içeren bir StatefulSet kullandım,
- Veritabanı kimlik bilgilerini, Kubernetes Secret olarak sakladım, `envFrom` ile servislere enjekte ettim
- Servislerin çökmesi durumunda sorun yaşanılmamaıs adına sabit IP yerine tamamen Kubernetes DNS'i üzerinden birbirine bağladım
- Tüm sistemi, local bir `kind` cluster'ına, `kubectl apply -f k8s/` ile deploy ettim
- ArgoCD'de, bu repoyu izleyen ve otomatik senkronize eden bir 
  Application kurdum
- Manifest'leri doğrulayan basit bir Jenkins pipeline'ı ekledim

## Karşılaştığım ve çözdüğüm gerçek sorunlar

Deploy sürecinde karşılaştığım ve çözdüğüm gerçek problemler:

- **Disk doluluğu ** — kullanılmayan Docker image/volume'leri, `docker system prune` ile temizledim,
- **StatefulSet'in CrashLoopBackOff vermesi** — sorunun, eski/bozuk bir PersistentVolumeClaim'den kaynaklandığını tespit ettim, Pod ve PVC'yi  silip StatefulSet'in temiz bir şekilde yeniden oluşturmasını sağladım,
- **Worker'ın "Waiting for db" mesajında takılı kalması** —  asıl sebebin Secret'taki şifre ile worker image'ının beklediği şifre arasındaki uyumsuzluk olduğunu, projenin orijinal `docker-stack.yml` dosyasını inceleyerek buldum,

## Kullanılan teknolojiler

Kubernetes · Docker · Jenkins · ArgoCD · Prometheus · PostgreSQL · Redis
