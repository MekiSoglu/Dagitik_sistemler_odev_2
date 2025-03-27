emergency_simulation dosyasında bulunan dosyada MPI ve OpenMP kullanılarak acil durum similasyonu yapılmaktadır . 
 Similasyonda bir şehirde zaman akışı içerisinde (olay_tipi ve mesafe) verilerine göre (İtfaye , Ambulans , Polis ) kategorilerin de 1000 acil durum olayı geçekleşiyor . Bu olaylara müdahale edebilmesi için 100 ekip bulunuyor. 
 MPI ile bu 100 kişik ekip 4 farklı rank ile aynı aynda kordine ediliyor . Her gelen acil durum ihbarında OpenMP ile 2 farklı görev için threatlar çalışıyor 


 Paralelleştirme Yapısı:

MPI ile ekipler 4 işlemciye bölünerek yönetilir.

Her işlem kendi olaylarını OpenMP kullanarak paralel işler:

1. Görev: Müsait ekipleri belirler. 

2. Görev: Olay yerine en yakın ekibi seçer ve ekibin müsaitlik durumunu günceller . Müsaitlik durumu müdahale süresi bitince tekrar güncellenir .



Kodu MPI ve OpenMP desteği ile derle:  mpicc -fopenmp -o emergency_simulation emergency_simulation.c -lm

MPI kullanarak simülasyonu başlat: mpirun -np 4 ./emergency_simulation

OpenMP aktif olarak çalıştırmak için thread sayısını belirle: export OMP_NUM_THREADS=4
mpirun -np 4 ./emergency_simulation


Örnek Çıktı :

ISTATISTIKLER
Toplam olay sayisi      : 1000
Başarıyla müdahale      : 573
Başarı oranı            : 57.30%
Toplam müdahale süresi  : 372.60 saniye
Ortalama müdahale süresi: 0.65 saniye
Toplam mesafe           : 14903.98 birim

Toplam yürütme süresi (MPI_Wtime() ile): 0.007153 saniye





DockerFile ve docker_compose_yml

Dockerfile:
projemiz için özel bir çalışma ortamı oluşturur.
Bu ortamda, gerekli programlar ve kütüphaneler otomatik olarak yüklenir ve kodun çalıştırılması için uygun bir ortam hazırlanır.

Amaç:
 Gerekli tüm bağımlılıkları otomatik yükler.
 MPI ve OpenMP desteği olan bir Ubuntu ortamı oluşturur.
 Acil durum simülasyonu için kodu otomatik olarak derler ve çalıştırmaya hazır hale getirir.


 docker-compose.yml:
Docker'ı kullanarak projemizi tek komutla çalıştırmamıza yardımcı olur.
Bu dosya, Docker konteynerini nasıl çalıştıracağımızı tanımlar ve çalıştırma ayarlarını düzenlememizi sağlar.

Amaç:
 Simülasyonu tek komutla başlatmamızı sağlar.
 Simülasyonun çıktılarının bilgisayarımızda saklanmasını sağlar.
 OpenMP için thread (iş parçacığı) sayısını ayarlamamıza imkan tanır.



 MPI ve OpenMp karşılaştılıması 


 MPI :

 emergency % mpirun -np 4 ./emergency_simulation   
 
 çıktı : Toplam yürütme süresi (MPI_Wtime() ile): 0.007153 saniye


 OpenMP ve MPI  :

OMP_NUM_THREADS=4
mpirun -np 4 ./emergency_simulation

 çıktı : Toplam yürütme süresi (MPI_Wtime() ile): 0.003361 saniye


 Farkın nedeni :

  MPI Tek Başına Çalışırken (0.0071 saniye):

MPI, 4 farklı işlem (rank) oluşturarak olayları farklı bölgelere dağıtır.

Her işlem, olayları sırayla işler, ancak her olay için tek bir işlemci çekirdeği çalışır.

  MPI + OpenMP Kullanıldığında (0.0033 saniye):

MPI yine 4 işlem oluşturur, ancak her işlem içinde OpenMP ile 4 farklı thread (iş parçacığı) çalışır.

Böylece, aynı anda daha fazla olay işlenebilir ve toplam süre yarıya iner.
