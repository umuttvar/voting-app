# voting-app
A voting app deployed on Kubernetes with 5 microservices. Jenkins run checks and ArgoCD handles automatic deployments.


---------------------------------------------------------------------------------------------------------------------------------------------------------------------

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

## Neler yapıldı

- Tüm Kubernetes dosyalarını (Deployment, Service, StatefulSet, Secret) sıfırdan, `k8s/` klasöründe yazdım
- PostgreSQL için, Pod yeniden başlasa bile verinin kaybolmaması amacıyla, `volumeClaimTemplates` içeren bir StatefulSet kullandım,
- Veritabanı kimlik bilgilerini, Kubernetes Secret olarak sakladım, `envFrom` ile servislere enjekte ettim
- Servislerin çökmesi durumunda sorun yaşanılmamaıs adına sabit IP yerine tamamen Kubernetes DNS'i üzerinden birbirine bağladım
- Tüm sistemi, local bir `kind` cluster'ına, `kubectl apply -f k8s/` ile deploy ettim

## Karşılaştığım ve çözdüğüm gerçek sorunlar

Deploy sürecinde karşılaştığım ve çözdüğüm gerçek problemler:

- **Disk doluluğu ** — kullanılmayan Docker image/volume'leri, `docker system prune` ile temizledim,
- **StatefulSet'in CrashLoopBackOff vermesi** — sorunun, eski/bozuk bir PersistentVolumeClaim'den kaynaklandığını tespit ettim, Pod ve PVC'yi  silip StatefulSet'in temiz bir şekilde yeniden oluşturmasını sağladım,
- **Worker'ın "Waiting for db" mesajında takılı kalması** —  asıl sebebin Secret'taki şifre ile worker image'ının beklediği şifre arasındaki uyumsuzluk olduğunu, projenin orijinal `docker-stack.yml` dosyasını inceleyerek buldum,

## Kullanılan teknolojiler

Kubernetes · Docker · PostgreSQL · Redis · kubectl
