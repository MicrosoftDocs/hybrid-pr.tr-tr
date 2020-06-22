---
title: Azure ve Azure Stack hub kullanarak analiz deseninin katmanlı verileri
description: Azure ve Azure Stack hub 'ı kullanarak hibrit bulut genelinde katmanlı veri çözümünü nasıl uygulayacağınızı öğrenin.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b671fa9f47fa51ab6e40633c04964957d613fec2
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84912013"
---
# <a name="tiered-data-for-analytics-pattern"></a>Analiz deseninin katmanlı verileri

Bu düzende, birden çok şirket içi ve bulut konumunda verileri hazırlamak, analiz etmek, işlemek, silmek ve depolamak için Azure Stack hub ve Azure 'ın nasıl kullanılacağı gösterilmektedir.

## <a name="context-and-problem"></a>Bağlam ve sorun

Modern teknolojideki kurumsal kuruluşlara yönelik sorunlardan biri, güvenli veri depolama, işleme ve çözümleme konularını ele taşıyor. Şunlar arasında dikkat edilecek noktalar:

- veri içeriği
- location
- Güvenlik ve gizlilik gereksinimleri
- erişim izinleri
- yapılması
- depolama alanı depolaması

Azure, Azure Stack hub ile birlikte, veri kaygılarını adresleyen ve düşük maliyetli çözümler sunmaktadır. Bu çözüm, dağıtılmış bir üretim veya lojistik şirket aracılığıyla en iyi şekilde ifade edilir.

Çözüm aşağıdaki senaryoya dayalıdır:

- Büyük bir çok dallı üretim organizasyonu.
- Hızlı ve güvenli veri depolama, işleme ve genel uzak konumlar arasında dağıtım, merkezi Merkez gereklidir.
- Çalışan ve makine etkinliği, Tesis bilgileri ve güvenli kalması gereken iş raporlama verileri. Verilerin uygun şekilde dağıtılması ve bölgesel uyumluluk ilkesini ve sektör düzenlemelerine uyması gerekir.

## <a name="solution"></a>Çözüm

Hem şirket içi hem de genel bulut ortamlarının kullanılması çok tesisli kuruluşların taleplerini karşılar. Azure Stack hub, yerel ve uzak verilerin toplanması, işlenmesi, depolanması ve dağıtılması için hızlı, güvenli ve esnek bir çözüm sunar. Bu model özellikle güvenlik, gizlilik, kurumsal ilke ve mevzuat gereksinimleri konumlar ve kullanıcılar arasında farklılık gösterebilse de kullanışlıdır.

![Analytics çözüm mimarisi için katmanlı veri deseninin](media/pattern-tiered-data-analytics/solution-architecture.png)

## <a name="components"></a>Bileşenler

Bu model aşağıdaki bileşenleri kullanır:

| Katman | Bileşen | Description |
|----------|-----------|-------------|
| Azure | Depolama | [Azure depolama](/azure/storage/) hesabı, bir sterile veri tüketimi uç noktası sağlar. Azure Depolama, Microsoft’un modern veri depolama senaryolarına yönelik bulut depolama çözümüdür. Azure depolama, veri nesneleri için yüksek düzeyde ölçeklenebilir bir nesne deposu ve bulut için bir dosya sistemi hizmeti sunar. Ayrıca, güvenilir mesajlaşma ve NoSQL Mağazası için bir mesajlaşma deposu sağlar. |
| Azure Stack hub 'ı | Depolama | Birden çok hizmet için [Azure Stack hub depolama](/azure-stack/user/azure-stack-storage-overview) hesabı kullanılır:<br><br>- Ham veri depolama için **BLOB depolama** . BLOB depolama, bir belge, medya dosyası veya uygulama yükleyicisi gibi herhangi bir tür metin veya ikili veri içerebilir. Her blob bir kapsayıcı altında düzenlenir. Kapsayıcılar, nesne gruplarına güvenlik ilkeleri atamak için kullanışlı bir yol sağlar. Depolama hesabı herhangi bir sayıda kapsayıcı içerebilir ve bir kapsayıcı depolama hesabının 500 TB 'lik kapasite sınırına kadar herhangi bir sayıda blob içerebilir.<br>- Veri arşivi için **BLOB depolama** . Seyrek Erişimli veri arşivleme için düşük maliyetli depolama avantajları vardır. Seyrek Erişimli verilerin örnekleri arasında yedeklemeler, medya içeriği, bilimsel veriler, uyumluluk ve arşiv verileri sayılabilir. Genel olarak, seyrek erişilen tüm veriler seyrek erişimli depolama olarak değerlendirilir. Erişim sıklığı ve Bekletme dönemi gibi özniteliklere göre verileri katmanlama. Müşteri verilerine seyrek erişilebilir ancak sık erişimli veriler için benzer gecikme ve performans gerekir.<br>- İşlenen veri depolaması için **kuyruk depolaması** . Kuyruk depolama, uygulama bileşenleri arasında bulut mesajlaşmasını sağlar. Uygulamaları ölçeklendirmek için tasarlarken, uygulama bileşenleri genellikle birbirinden bağımsız olarak ölçeklendirilebilecek şekilde ayrılır. Kuyruk depolama, bulutta, masaüstünde, şirket içi sunucuda veya mobil bir cihazda çalışıp çalışmadığını, uygulama bileşenleri arasındaki iletişim için zaman uyumsuz mesajlaşma sağlar. Kuyruk depolama ayrıca zaman uyumsuz görevlerin yönetilmesini ve süreç iş akışlarının oluşturulmasını destekler. |
| | Azure İşlevleri | [Azure işlevleri](/azure/azure-functions/) hizmeti, [Azure App Service Azure Stack hub](/azure-stack/operator/azure-stack-app-service-overview) kaynak sağlayıcısı tarafından sağlanır. Azure Işlevleri, çeşitli olaylara yanıt olarak kodunuzu basit, sunucusuz bir ortamda yürütmenize olanak tanır. Azure Işlevleri, tercih ettiğiniz programlama dilini kullanarak bir VM oluşturmak veya bir Web uygulaması yayımlamak zorunda kalmadan talebe uyacak şekilde ölçeklendirilir. İşlevleri için çözüm tarafından kullanılır:<br><br>- **Veri alma**<br>- **Veri sterilleme.** El ile tetiklenen işlevler, zamanlanmış veri işleme, temizleme ve arşivleme işlemleri gerçekleştirebilir. Örnekler, gecelik müşteri listesi itilen ve aylık rapor işleme içerebilir.|

## <a name="issues-and-considerations"></a>Sorunlar ve dikkat edilmesi gerekenler

Bu çözümü nasıl uygulayacağınıza karar verirken aşağıdaki noktaları göz önünde bulundurun:

### <a name="scalability"></a>Ölçeklenebilirlik

Veri hacmi ve işleme taleplerini karşılamak için Azure Işlevleri ve depolama çözümleri ölçeği. Azure ölçeklenebilirlik bilgileri ve hedefleri için bkz. [Azure Storage ölçeklenebilirlik belgeleri](/azure/storage/common/storage-scalability-targets).

### <a name="availability"></a>Kullanılabilirlik

Depolama, bu model için birincil kullanılabilirlik açısından önemlidir. Büyük veri hacmi işleme ve dağıtımı için hızlı bağlantılar aracılığıyla bağlantı gereklidir.

### <a name="manageability"></a>Yönetilebilirlik

Bu çözümün yönetilebilirlik, kaynak denetimi ve görevlendirmede kullanılan yazma araçlarına bağlıdır.

## <a name="next-steps"></a>Sonraki adımlar

Bu makalede tanıtılan konular hakkında daha fazla bilgi edinmek için:

- Bkz. [Azure depolama](/azure/storage/) ve [Azure işlevleri](/azure/azure-functions/) belgeleri. Bu model, Azure depolama hesapları ve Azure Işlevleri için Azure ve Azure Stack Hub üzerinde çok fazla kullanım sağlar.
- En iyi uygulamalar hakkında daha fazla bilgi edinmek ve ek sorulara yanıt almak için bkz. [karma uygulama tasarımı konuları](overview-app-design-considerations.md) .
- Ürünlerin ve çözümlerin tamamı hakkında daha fazla bilgi edinmek için [ürün ve çözümlerin Azure Stack ailesine](/azure-stack) bakın.

Çözüm örneğini test etmeye hazırsanız, [analiz çözümü Dağıtım Kılavuzu ' na yönelik katmanlı veriler](https://aka.ms/tiereddatadeploy)' e geçin. Dağıtım Kılavuzu, bileşenlerinin dağıtılması ve test edilmesi için adım adım yönergeler sağlar.
