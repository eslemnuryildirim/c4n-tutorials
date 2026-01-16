# Helm Eğitim Görevleri

---

## Task 1: Mevcut Paketleri Keşfetme ve Kurulum

### Görev
ArtifactHub üzerinden resmi bir nginx veya redis chart'ı bulup Minikube/k3s üzerine kurun.

### Ödev Detayı
1. Bitnami reposunu Helm'e ekleyin
2. helm install ile bir uygulama ayağa kaldırın
3. helm list ile durumu kontrol edin

### Yapılan İşlemler

#### 1. Bitnami Reposunu Helm'e Ekleme
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

#### 2. Nginx Uygulamasını Kurma
```bash
helm install my-nginx bitnami/nginx
```

#### 3. Durumu Kontrol Etme
```bash
helm list
helm status my-nginx
```

### helm status Çıktısı
```
NAME: my-nginx
LAST DEPLOYED: Wed Jan 14 06:24:02 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1

NOTES:
CHART NAME: nginx
CHART VERSION: 22.4.3
APP VERSION: 1.29.4

** Please be patient while the chart is being deployed **

NGINX can be accessed through the following DNS name from within your cluster:
    my-nginx.default.svc.cluster.local (port 80)

To access NGINX from outside the cluster:
    export SERVICE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].port}" services my-nginx)
    export SERVICE_IP=$(kubectl get svc --namespace default my-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    echo "http://${SERVICE_IP}:${SERVICE_PORT}"
```

### Soru: helm status komutundaki "Notes" bölümü ne işe yarar?

Notes bölümü şu amaçlara hizmet eder:

1. **Kurulum Sonrası Talimatlar**: Uygulamaya nasıl erişileceğini gösterir (port-forward, LoadBalancer IP alma vb.)
2. **Bağlantı Bilgileri**: Hangi URL veya port üzerinden erişileceğini belirtir
3. **Varsayılan Credentials**: Şifre gerektiren uygulamalarda kullanıcı bilgilerini veya nasıl alınacağını gösterir
4. **Önemli Uyarılar**: Güvenlik veya yapılandırma ile ilgili notları içerir

### Port Forwarding ile Erişim
Minikube'da LoadBalancer olmadığı için port-forward kullanıldı:

```bash
kubectl port-forward svc/my-nginx 8080:80
```

Çıktı:
```
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

Tarayıcıdan `http://localhost:8080` adresine gidildiğinde nginx karşılama sayfası görüntülendi.

![Nginx Welcome Page](screenshots/nginx-welcome.png)

---

## Task 2: Yapılandırma ve Values.yaml

### Görev
Kurduğunuz uygulamanın özelliklerini (örneğin replica sayısını veya servis tipini) dışarıdan müdahale ederek değiştirin.

### Ödev Detayı
1. values.yaml dosyasını export edin
2. Replica sayısını 1'den 3'e çıkarın
3. Uygulamanın 1 cpu, 2 gb memory ile çalışmasını sağlayın
4. helm upgrade komutuyla sistemi güncelleyin

### Yapılan İşlemler

#### 1. Default Values Dosyasını Export Etme
```bash
helm show values bitnami/nginx > default-values.yaml
```

#### 2. Özelleştirilmiş Values Dosyası Oluşturma
```yaml
replicaCount: 3

resources:
  limits:
    cpu: "1"
    memory: "2Gi"
  requests:
    cpu: "500m"
    memory: "1Gi"
```

![Values.yaml Yapılandırması](screenshots/values.yaml.png)

![Values.yaml Detay](screenshots/values.yaml1.png)

#### 3. Helm Upgrade ile Güncelleme
```bash
helm upgrade my-nginx bitnami/nginx -f values.yaml
```

### Upgrade Çıktısı
```
Release "my-nginx" has been upgraded. Happy Helming!
NAME: my-nginx
LAST DEPLOYED: Thu Jan 15 21:47:24 2026
NAMESPACE: default
STATUS: deployed
REVISION: 6
```

### Sonuç ve Gözlemler

Upgrade sonrası pod durumları:
```
NAME                        READY   STATUS    RESTARTS   AGE
my-nginx-5664ffbb99-xpvth   1/1     Running   0          39h
my-nginx-5664ffbb99-xtmch   1/1     Running   0          19s
my-nginx-7c5d7997-4xcjx     1/1     Running   0          6h43m
my-nginx-7c5d7997-8dz82     0/1     Pending   0          19s
my-nginx-fc5ccb965-9rlrk    0/1     Pending   0          69s
```

Bazı pod'lar "Pending" durumunda kaldı. Sebebi Minikube'un tek node üzerinde çalışması ve belirlenen kaynak gereksinimlerini (1 CPU, 2Gi memory) karşılayacak yeterli kapasitesinin olmaması.

`kubectl describe pod <pod-name>` komutuyla kontrol edildiğinde:
```
0/1 nodes are available: insufficient cpu.
```

Bu durum bir hata değil, Kubernetes'in kaynak yönetiminin doğru çalıştığının göstergesi. Scheduler, node'da yeterli kaynak olmadığı için pod'u schedule edemiyor ve Pending'de bekletiyor.

### Öğrenilen Kavramlar

- **values.yaml**: Helm chart'larını özelleştirmek için kullanılan yapılandırma dosyası
- **helm upgrade**: Mevcut bir release'i yeni değerlerle güncellemek için kullanılır
- **Resource Limits**: Pod'ların kullanabileceği maksimum kaynak miktarını belirler
- **Pending Status**: Node'da yeterli kaynak olmadığında pod'ların bekleme durumu

---

## Task 3: Kendi Chart'ını Oluşturma (Deep Dive)

### Görev
Çok basit bir "Hello World" (Flask veya Node.js olabilir) uygulamasını Helm Chart haline getirin.

### Ödev Detayı
1. helm create my-app komutuyla iskeleti oluşturun
2. Gereksiz dosyaları temizleyin
3. templates/ klasörü altındaki YAML dosyalarında Go Templating kullanarak dinamik alanlar oluşturun
4. helm lint komutuyla chart'ın standartlara uygunluğunu test edin

### Yapılan İşlemler

#### 1. Flask Uygulaması Oluşturma
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from Helm Chart!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

#### 2. Dockerfile Oluşturma
```dockerfile
FROM python:3.9-slim
WORKDIR /app
RUN pip install flask
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```

#### 3. Helm Chart İskeleti Oluşturma
```bash
helm create my-app
```

#### 4. values.yaml Yapılandırması
```yaml
replicaCount: 2

image:
  repository: my-hello-app
  pullPolicy: IfNotPresent
  tag: "v1.0.0"

service:
  type: ClusterIP
  port: 5000

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi

app:
  message: "Hello from My Helm Chart!"
```

#### 5. Go Templating Kullanımı (deployment.yaml)
```yaml
containers:
- name: {{ .Chart.Name }}
  image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
  imagePullPolicy: {{ .Values.image.pullPolicy }}
  ports:
  - name: http
    containerPort: 5000
  env:
  - name: APP_MESSAGE
    value: {{ .Values.app.message | quote }}
  resources:
    {{- toYaml .Values.resources | nindent 10 }}
```

#### 6. Helm Lint ile Test
```bash
helm lint
```

Çıktı:
```
==> Linting .
[INFO] Chart.yaml: icon is recommended
1 chart(s) linted, 0 chart(s) failed
```

#### 7. Chart'ı Deploy Etme ve Test
```bash
helm install my-release ./my-app
curl $(minikube service my-release-my-app --url)
```
```
Hello from Helm Chart!
```

![Task 3 - Hello World Çıktısı](screenshots/task3-helloworld.png)

### Karşılaşılan Sorun: ImagePullBackOff

İlk deploy denemesinde `ImagePullBackOff` hatası alındı:

```
NAME                                 READY   STATUS             RESTARTS   AGE
my-release-my-app-d94869d65-rs9bl   0/1     ImagePullBackOff   0          26s
```

![ImagePullBackOff Hatası](screenshots/task3-imagepullbackoff.png)

#### Sorunun Sebebi
VM'de `docker build` komutu çalıştırıldığında image VM'nin Docker daemon'ında oluşuyor. Ancak Minikube kendi Docker daemon'ını kullanıyor ve bu image'ı göremiyordu.

```
┌─────────────────────────────────┐
│         VM (vagrant)            │
│  ┌──────────────────────────┐  │
│  │  Docker Daemon (VM)      │  │  ← Build edilen yer
│  │  • my-hello-app:v1.0.0   │  │
│  └──────────────────────────┘  │
│                                 │
│  ┌──────────────────────────┐  │
│  │  Minikube                │  │
│  │  ┌────────────────────┐  │  │
│  │  │ Docker Daemon      │  │  │  ← Kubernetes'in baktığı yer
│  │  │ (Minikube içinde)  │  │  │
│  │  │ • IMAGE YOK! ❌    │  │  │
│  │  └────────────────────┘  │  │
│  └──────────────────────────┘  │
└─────────────────────────────────┘
```

#### Çözüm
Minikube'nin Docker daemon'ına geçip image'ı orada build etmek:

```bash
# Minikube'nin Docker'ına geç
eval $(minikube docker-env)

# Image'ı Minikube içinde build et
cd ~/c4n-tutorials/helm-tasks/task3
docker build -t my-hello-app:v1.0.0 .

# Image'ın Minikube'de olduğunu kontrol et
docker images | grep my-hello-app
```

values.yaml düzeltmesi:
```yaml
image:
  repository: my-hello-app
  pullPolicy: Never    # IfNotPresent yerine Never
  tag: "v1.0.0"
```

Helm'i tekrar kur:
```bash
helm uninstall my-release
helm install my-release . --set image.pullPolicy=Never
```

Bu değişikliklerden sonra pod'lar başarıyla Running durumuna geçti.

### Öğrenilen Kavramlar

- **helm create**: Yeni bir Helm chart iskeleti oluşturur
- **Go Templating**: `{{ .Values.xxx }}` syntax'ı ile dinamik değerler tanımlanır
- **helm lint**: Chart'ın Helm standartlarına uygunluğunu kontrol eder
- **Chart.yaml**: Chart metadata'sını içerir (isim, versiyon, açıklama)
- **values.yaml**: Varsayılan yapılandırma değerlerini içerir
- **eval $(minikube docker-env)**: Terminal'i Minikube'nin Docker daemon'ına bağlar
- **imagePullPolicy: Never**: Kubernetes'in image'ı registry'den çekmemesini sağlar
