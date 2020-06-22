---
title: Azure ve Azure Stack Edge kullanarak hisse senedi algılama dışı
description: Azure ve Azure Stack Edge hizmetlerini kullanarak stok algılamayı nasıl uygulayacağınızı öğrenin.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 865f63bc4234e50ed169aa29cefdb1886750594c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911989"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Kenar düzeninde hisse senedi algılama

Bu düzende, raflar 'in bir Azure Stack Edge veya Azure IoT Edge cihaz ve ağ kameralarını kullanarak hisse senedi öğeleri olup olmadığını nasıl belirleyebileceği gösterilmektedir.

## <a name="context-and-problem"></a>Bağlam ve sorun

Fiziksel perakende mağazalarında, müşteriler bir öğeyi arayacağı, rafında yer almıyor. Ancak öğe, deponun arkasında ve geri eklenmemiş olabilir. Mağazaların personelini daha verimli bir şekilde kullanmak ve öğelerin yeniden kullanıma ihtiyacı olduğunda otomatik olarak bilgilendirmek ister.

## <a name="solution"></a>Çözüm

Çözüm örneği her depoda Azure Stack Edge gibi bir uç cihaz kullanır ve bu da depodaki kameralardan verileri verimli bir şekilde işler. Bu iyileştirilmiş tasarım, mağazaların buluta yalnızca ilgili olayları ve görüntüleri göndermesini sağlar. Tasarım bant genişliğini, depolama alanını kaydeder ve müşterinin gizliliğini sağlar. Çerçeveler her bir kameradan okunmasıyla, bir ML modeli görüntüyü işler ve stok dışı alanlardan herhangi birini döndürür. Görüntü ve stok alanlarının tümü yerel bir Web uygulamasında görüntülenir. Bu veriler, Power BI Öngörüler göstermek için zaman serisi Insight ortamına gönderilebilir.

![Uçtan uca çözüm mimarisi](media/pattern-out-of-stock-at-edge/solution-architecture.png)

Çözümün nasıl çalıştığı aşağıda verilmiştir:

1. Görüntüler, HTTP veya RTSP üzerinden bir ağ kamerasından yakalanır.
2. Görüntü yeniden boyutlandırılır ve çıkarım sürücüsüne gönderilir. Bu, herhangi bir hisse senedi görüntüsü olup olmadığını anlamak için ML modeliyle iletişim kurar.
3. ML modeli, stok dışı alanlardan herhangi birini döndürür.
4. Inyavanı sürücü, Ham görüntüyü bir blob 'a yükler (belirtilmişse) ve sonuçları modelden Azure IoT Hub ve bir sınırlayıcı kutu işlemcisine gönderir.
5. Sınırlama kutusu işlemcisi görüntüye sınırlayıcı kutular ekler ve görüntü yolunu bellek içi veritabanında önbelleğe alır.
6. Web uygulaması görüntüleri sorgular ve alındıkları sırada gösterir.
7. IoT Hub iletiler Time Series Insights toplanmaktadır.
8. Power BI, Time Series Insights verileri ile zaman içinde stok öğelerinden oluşan etkileşimli bir rapor görüntüler.


## <a name="components"></a>Bileşenler

Bu çözüm aşağıdaki bileşenleri kullanır:

| Katman | Bileşen | Description |
|----------|-----------|-------------|
| Şirket içi donanım | Ağ kamerası | Bir ağ kamerası, görüntüleri çıkarımı için sağlamak üzere HTTP veya RTSP beslemeli bir akışa sahip olmalıdır. |
| Azure | Azure IoT Hub | [Azure IoT Hub](/azure/iot-hub/) , sınır cihazları için cihaz sağlamayı ve mesajlaşma işlemini gerçekleştirir. |
|  | Azure Zaman Serisi Görüşleri | [Azure Time Series Insights](/azure/time-series-insights/) , IoT Hub iletileri görselleştirme için depolar. |
|  | Power BI | [Microsoft Power BI](https://powerbi.microsoft.com/) , stok dışı olaylara yönelik iş odaklı raporlar sağlar. Power BI, çıktıyı Azure Stream Analytics görüntülemek için kullanımı kolay bir pano arabirimi sağlar. |
| Azure Stack Edge veya<br>Azure IoT Edge cihaz | Azure IoT Edge | [Azure IoT Edge](/azure/iot-edge/) , şirket içi kapsayıcılar için çalışma zamanını düzenler ve cihaz yönetimini ve güncelleştirmelerini işler.|
| | Azure proje Brainwave | Azure Stack Edge cihazında, [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) , ml ınklesi hızlandırmak için alan-programlanabilir kapı dizileri (FPGAs) kullanır.|

## <a name="issues-and-considerations"></a>Sorunlar ve dikkat edilmesi gerekenler

Bu çözümü nasıl uygulayacağınıza karar verirken aşağıdaki noktaları göz önünde bulundurun:

### <a name="scalability"></a>Ölçeklenebilirlik

Çoğu makine öğrenimi modeli, belirtilen donanıma bağlı olarak, saniye başına yalnızca belirli sayıda karede çalıştırılabilir. ML ardışık düzeninin yedeklemediğinden emin olmak için kameranızdan en iyi örnek hızı öğrenin. Farklı donanım türleri farklı sayıda kameraları ve kare hızını işleymeyecektir.

### <a name="availability"></a>Kullanılabilirlik

Sınır cihazının bağlantıyı kaybetmesi durumunda ne olabileceğini göz önünde bulundurmanız önemlidir. Time Series Insights ve Power BI panosundan hangi verilerin kaybolabileceğini göz önünde bulundurun. Sağlanan örnek çözüm, yüksek oranda kullanılabilir olacak şekilde tasarlanmamıştır.

### <a name="manageability"></a>Yönetilebilirlik

Bu çözüm, çok sayıda cihaza ve konuma yayılabilir ve bu da bir süre sonra olabilir. Azure 'un IoT Hizmetleri, yeni konumları ve cihazları otomatik olarak çevrimiçi duruma getirebilir ve güncel tutar. Uygun veri idare yordamlarının de ardından gelmelidir.

### <a name="security"></a>Güvenlik

Bu model potansiyel olarak hassas verileri işler. Anahtarların düzenli olarak döndürüldüğü ve Azure depolama hesabı ve yerel paylaşımlarının izinlerinin doğru ayarlandığından emin olun.

## <a name="next-steps"></a>Sonraki adımlar

Bu makalede tanıtılan konular hakkında daha fazla bilgi edinmek için:
- Bu düzende [Azure IoT Edge](/azure/iot-edge/), [Azure IoT Hub](/azure/iot-hub/)ve [Azure Time Series Insights](/azure/time-series-insights/)dahil birden çok IoT ile ilgili hizmet kullanılır.
- Microsoft Project Brainwave hakkında daha fazla bilgi edinmek için bkz. [Blog duyurusu](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) ve [Project Brainwave videosu ile Azure hızlandırılmış Machine Learning](https://www.youtube.com/watch?v=DJfMobMjCX0)kullanıma alma.
- En iyi uygulamalar hakkında daha fazla bilgi edinmek ve ek sorulara yanıt almak için bkz. [karma uygulama tasarımı konuları](overview-app-design-considerations.md) .
- Ürünlerin ve çözümlerin tamamı hakkında daha fazla bilgi edinmek için [ürün ve çözümlerin Azure Stack ailesine](/azure-stack) bakın.

Çözüm örneğini test etmeye hazırsanız, [analiz çözümü Dağıtım Kılavuzu ' na yönelik katmanlı veriler](https://aka.ms/edgeinferencingdeploy)' e geçin. Dağıtım Kılavuzu, bileşenlerinin dağıtılması ve test edilmesi için adım adım yönergeler sağlar.
