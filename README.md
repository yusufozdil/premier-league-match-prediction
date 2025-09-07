# Premier League Maç Sonucu Tahmin Modeli

## Projeye Genel Bakış

Bu proje, İngiltere Premier League futbol maçlarının sonuçlarını tahmin etmek için geliştirilmiş istatistiksel bir makine öğrenmesi modelini içermektedir. Projenin temel yaklaşımı, maçın nihai sonucunu (Ev Sahibi Galibiyeti, Beraberlik, Deplasman Galibiyeti) doğrudan bir sınıflandırma problemi olarak ele almak yerine, her takımın **atması beklenen gol sayısını** tahmin etmektir. Bu yöntem, spor analizlerinde yaygın olarak kullanılan Poisson dağılımına dayanır ve daha esnek ve isabetli tahminler üretme potansiyeline sahiptir.

Model, takımların geçmiş performanslarını, göreceli güçlerini ve mevcut form durumlarını analiz ederek, gelecekteki maçlar için olasılıksal tahminler sunar.

---

## Metodoloji

Modelin çekirdeğini, her takım için ayrı ayrı gol beklentisi üreten iki uzman `XGBoost Regressor` modeli oluşturur.

1.  **Ev Sahibi Gol Uzmanı:** Ev sahibi takımın hücum performansı ve rakip takımın deplasman savunma istatistiklerine dayanarak ev sahibinin atması beklenen gol sayısını tahmin eder.
2.  **Deplasman Gol Uzmanı:** Deplasman takımının hücum performansı ve ev sahibi takımın iç saha savunma istatistiklerine dayanarak deplasman takımının atması beklenen gol sayısını tahmin eder.

Bu iki modelden elde edilen beklenen gol sayıları, Poisson olasılık kütle fonksiyonu kullanılarak tüm olası maç skorlarının (0-0, 1-0, 0-1, 1-1, vb.) gerçekleşme olasılıklarını hesaplamak için kullanılır. Son olarak, bu skor olasılıkları toplanarak maçın nihai sonucu için **Ev Sahibi Galibiyeti, Beraberlik ve Deplasman Galibiyeti** olasılıkları elde edilir.

### Özellik Mühendisliği (Feature Engineering)

Modelin isabetliliğini artırmak için ham veriden çeşitli anlamlı özellikler türetilmiştir:

*   **Dinamik ELO Reytingi:** Her takım için bir ELO puanı hesaplanır. Bu sistem, sadece maç sonucuna değil, aynı zamanda **gol farkına** da duyarlıdır. Büyük farklı galibiyetler, ELO puanlarında daha anlamlı değişikliklere yol açarak takımların o anki gücünü daha hassas bir şekilde yansıtır.

*   **Form Metrikleri:** Her maçtan önce, takımların oynadığı son 5 maçtaki performansını yansıtan metrikler hesaplanır. Bunlar arasında atılan/yenilen gol ortalamaları ve toplanan puanlar bulunur.

*   **Sezonluk Hücum ve Savunma Gücü:** Her takımın ev ve deplasman performansları, ligin o sezonki genel gol ortalamalarıyla karşılaştırılarak göreceli bir "Hücum Gücü" ve "Savunma Gücü" skoru oluşturulur.

### Model Optimizasyonu

Her iki gol uzmanı model de `RandomizedSearchCV` kullanılarak **hiperparametre optimizasyonuna** tabi tutulmuştur. Bu süreç, modelin `n_estimators`, `max_depth`, `learning_rate` gibi iç ayarlarının en iyi kombinasyonunu otomatik olarak bularak, gol tahminlerindeki isabet oranını en üst düzeye çıkarmayı hedefler.

---

## Dosya Yapısı

*   `epl_match_predict.ipynb`: Projenin tüm adımlarını içeren ana Jupyter Notebook dosyası. Veri işleme, özellik mühendisliği, model eğitimi, optimizasyon ve interaktif tahmin fonksiyonu bu dosyada yer almaktadır.
*   `epl_final.csv`: Modelin eğitimi ve testi için kullanılan, 2000-2025 yılları arasındaki Premier League maç verilerini içeren veri seti.
*   `README.md`: Bu dosya.

---

## Kurulum ve Kullanım

Bu projeyi kendi bilgisayarınızda çalıştırmak için aşağıdaki adımları izleyebilirsiniz.

### Gerekli Kütüphaneler

Projenin çalışması için aşağıdaki Python kütüphanelerinin kurulu olması gerekmektedir:

* pandas
* numpy
* scikit-learn
* xgboost
* matplotlib


Bu kütüphaneleri pip kullanarak yükleyebilirsiniz:
```bash
pip install pandas numpy scikit-learn xgboost matplotlib
```

### Projeyi Çalıştırma

1.  Bu repoyu klonlayın veya dosyaları indirin.
2.  `epl_match_predict.ipynb` dosyasını Jupyter Notebook veya uyumlu bir ortamda (JupyterLab, VS Code vb.) açın.
3.  Not defterindeki hücreleri **yukarıdan aşağıya doğru sırayla** çalıştırın.
4.  Not defterinin en sonundaki interaktif tahmin hücresine geldiğinizde, ekrandaki talimatları izleyerek istediğiniz takımlar için maç sonucu olasılıklarını alabilirsiniz.

---

## Sonuçlar

Yapılan testler sonucunda model, her üç maç sonucunu da (Ev Sahibi Galibiyeti, Beraberlik, Deplasman Galibiyeti) dengeli bir şekilde tahmin edebilme yeteneği göstermiştir. Özellikle, basit sınıflandırma modellerinin en büyük zafiyeti olan "beraberlik" sonucunu tahmin etmedeki başarısı, seçilen Poisson yaklaşımının doğruluğunu kanıtlamaktadır.

Modelin test seti üzerindeki performansı, genel olarak %45-50 civarında bir doğruluk oranına ulaşmıştır. Bu oran, futbol gibi oldukça değişken ve tahmin edilmesi zor bir sporda, sadece geçmiş istatistiksel veriler kullanılarak elde edilmiş başarılı bir sonuç olarak değerlendirilmektedir. Modelin en büyük gücü, tek bir sonuç dayatmak yerine, her bir olası sonuç için olasılıksal bir dağılım sunarak kullanıcıya daha zengin bir analiz imkanı tanımasıdır.

## Gelecekteki Geliştirmeler

Bu model, gelecekte aşağıdaki yöntemlerle daha da geliştirilebilir:

*   **Daha Granüler Veri:** Oyuncu sakatlıkları, cezalı oyuncular, takımların kullandığı formasyonlar ve maç içi daha detaylı istatistikler (beklenen gol - xG, topla oynama yüzdeleri vb.) gibi verilerin eklenmesi.
*   **Farklı Model Mimarileri:** XGBoost'a alternatif olarak Gradient Boosting, Random Forest veya daha karmaşık sinir ağı (neural network) modellerinin denenmesi.
*   **Web Arayüzü:** Modelin tahmin fonksiyonunun Streamlit veya Flask gibi araçlar kullanılarak basit bir web uygulamasına dönüştürülmesi.
