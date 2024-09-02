İlk olarak argocd yi helm ile kuruyoruz ki uygulamalarımızı argocd üzerinden takip edebilelim.

Keycloak, açık kaynaklı bir kimlik ve erişim yönetim sistemidir. Kullanıcı kimlik doğrulama, yetkilendirme ve kullanıcı yönetimi işlevlerini sağlar
Daha fazla bilgi için; https://www.keycloak.org/ ve https://github.com/bitnami/charts/tree/main/bitnami/keycloak adreslerine göz atabilirsiniz.

https://github.com/bitnami/charts/tree/main/bitnami/keycloak bu helm chart ı kullanarak argocd ile keycloak uygulamasını deploy ediyoruz.

Dikkat edilmesi gerekenler;

Argocd kurulumu;

helm upgrade -i argocd -n argocd argo/argo-cd -f values.yaml --version 5.19.15 bu komutu çalıştırıyoruz.

kubectl port-forward service/argocd-server -n argocd 8080:443 bu komut ile ya da lens gibi bir uygulama ile port-forward yaparak arayüze ulaşalım.

Her şey yolunda ise şimdi keycloak kurulum aşamasına geçebiliriz.

Keycloak kurulumu;

kubectl apply -f argocd.yaml komutu ile deploy edebilirsiniz. Öncesinde Configmap leri apply lamayı unutmayın.

configmap.yaml keycloak-config-cli configurasyonunu eğer true yaparsanız, kullanmanız gerekir. Bu kısım realm oluşturma gibi seçenekleri arayüz yerine configmap ile yapmanızı ve kayıt altında tutmanızı sağlar.

configmap2.yaml da ise https://github.com/aerogear/keycloak-metrics-spi kurulumu için gereklidir.

İlk olarak yapılması gereken adımlar;

git clone https://github.com/aerogear/keycloak-metrics-spi.git
cd keycloak-metrics-spi

mvn package

(hata verirse java versionu 21 olmalı)
java -version
brew install openjdk@21
export JAVA_HOME=$(/usr/libexec/java_home -v 21)
java -version


base64 original-keycloak-metrics-spi-6.0.1-SNAPSHOT.jar > keycloak-metrics-spi.base64

sonrasında base64 lü halini configmap2.yaml içerisine yapıştırıyoruz.

ve argocd.yaml da 

        extraVolumes:
          - name: keycloak-metrics-spi
            configMap:
              name: keycloak-metrics-spi
        extraVolumeMounts:
          - name: keycloak-metrics-spi
            mountPath: /opt/keycloak/providers/keycloak-metrics-spi.jar
            subPath: keycloak-metrics-spi.jar

bu ayarları yapıyoruz.
