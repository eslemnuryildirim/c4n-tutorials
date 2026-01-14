Görev: ArtifactHub üzerinden resmi bir nginx veya redis chart'ı bulup Minikube/k3s üzerine kurun
Ödev Detayı: 1. Bitnami reposunu Helm'e ekleyin. 2. helm install ile bir uygulama ayağa kaldırın. 3. helm list ile durumu kontrol edin.
Soru: helm status komutu çıktısındaki "Notes" bölümü ne işe yarar?

2: Yapılandırma ve "Values.yaml"
Görev: Kurduğunuz uygulamanın özelliklerini (örneğin replica sayısını veya servis tipini) dışarıdan müdahale ederek değiştirin.
Ödev Detayı: 1. values.yaml dosyasını export edin. 2. Replica sayısını 1'den 3'e çıkarın.
3. Uygulamanın 1 cpu, 2 gb memory ile çalışmasını sağlayın.
4. helm upgrade komutuyla sistemi güncelleyin.

Modül 3: Kendi Chart'ını Oluşturma (Deep Dive)
İşin mutfağına girme zamanı. Bir uygulamanın nasıl paketlendiğini görmeleri gerekiyor.
Görev: Çok basit bir "Hello World" (Flask veya Node.js olabilir) uygulamasını Helm Chart haline getirin.
Ödev Detayı: 1. helm create my-app komutuyla iskeleti oluşturun. 2. Gereksiz dosyaları temizleyin. 3. templates/ klasörü altındaki YAML dosyalarında Go Templating kullanarak (örneğin: {{ .Values.image.repository }}) dinamik alanlar oluşturun. 4. helm lint komutuyla yazdıkları chart'ın standartlara uygunluğunu test etsinler.

4: Release Yönetimi ve Geri Dönüş (Rollback)
Görev: Uygulamanın imaj versiyonunu bilerek "yanlış/bozuk" bir versiyonla güncelleyin ve sistemin çöküşünü izleyip geri dönün.
Ödev Detayı: 1. Hatalı bir imaj etiketiyle upgrade yapın.
2. kubectl get pods ile hatayı (ImagePullBackOff) görün. 3. helm history ile geçmişi listeleyin. 4. helm rollback komutuyla çalışan son versiyona saniyeler içinde geri dönün.

5: Final Projesi (Full Stack)
Proje: Bir WordPress sitesini, veritabanı (MariaDB/MySQL) ile birlikte Helm kullanarak kurun.
Ekstra Zorluk: Veritabanı şifresini düz metin olarak değil, bir Secret objesi üzerinden Helm ile deploy edin. (secret.yaml).
2x Ekstra Zorluk:  Worldpress'e DNS tanımlayın ve HTTPS (SSL sertifikası ile) çalışmasını sağlayın.
Not: Sertifika satın almayın!
