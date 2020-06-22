---
title: Azure ve Azure Stack hub kullanan bir betfall algılama kalıbı
description: Retail Store trafiğini çözümlemek için bir AI tabanlı bir ıtfall algılama çözümü uygulamak üzere Azure ve Azure Stack hub 'ı nasıl kullanacağınızı öğrenin.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 0bf07bb38537f530a0adb3569c43d53af13b8d56
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84912002"
---
# <a name="footfall-detection-pattern"></a>Betfall algılama kalıbı

Bu düzende, perakende mağazalarındaki ziyaretçi trafiğini çözümlemek için bir AI tabanlı bir kantfall algılama çözümü uygulamaya yönelik bir genel bakış sunulmaktadır. Çözüm, Azure, Azure Stack hub ve Özel Görüntü İşleme AI Dev Kit kullanarak gerçek dünya eylemlerinden Öngörüler oluşturur.

## <a name="context-and-problem"></a>Bağlam ve sorun

Contoso mağazalarının, müşterilerin güncel ürünlerini mağaza düzenine göre nasıl aldığını hakkında öngörüler elde etmek ister. Her bölüme personeli yerleştiremiyor ve bir analistlerin bir yöneticisinin tüm mağazanın kamera görüntülerini incelemesi için verimsiz bir takım. Bunlara ek olarak, mağazaların hiçbirinde tüm kameralardan analiz için buluta video akışı yapmak için yeterli bant genişliği yoktur.

Contoso, müşterilerinin demograflarını, kısa bir süre içinde, ekran ve ürünleri depolamak için müşterilerin demograflarını, bağlılık ve yeniden eylemlerini belirlemenin kolay bir yolunu bulmak istiyor.

## <a name="solution"></a>Çözüm

Bu Retail Analytics deseninin kenar altına almak için katmanlı bir yaklaşım kullanılır. Özel Görüntü İşleme AI geliştirme setini kullanarak, Azure bilişsel hizmetler 'i çalıştıran özel bir Azure Stack hub 'ına analiz edilmek üzere yalnızca insan yüzlerine sahip görüntüler gönderilir. Anonimleştirilmiş, toplanan veriler Power BI tüm depolarda ve görselleştirmede toplama için Azure 'a gönderilir. Edge ve genel bulutu birleştirmek, contoso 'nun modern AI teknolojisinden yararlanarak şirket ilkeleriyle uyumlu olduğundan ve müşterilerinin gizliliğini de daha da yararlanmasını sağlar.

[![Betfall algılama deseninin çözümü](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Çözümün nasıl çalıştığına ilişkin bir özet aşağıda verilmiştir:

1. Özel Görüntü İşleme AI Dev Kit, IoT Edge çalışma zamanını ve bir ML modelini yükleyerek IoT Hub bir yapılandırma alır.
2. Model bir kişi görürse, bir resim alır ve Azure Stack hub BLOB depolama alanına yükler.
3. Blob hizmeti Azure Stack hub 'da bir Azure Işlevi tetikler.
4. Azure Işlevi, görüntüden demografik ve duygu verileri almak için Yüz Tanıma API'si bir kapsayıcı çağırır.
5. Veriler anonimleştirilmiştir ve bir Azure Event Hubs kümesine gönderilir.
6. Event Hubs kümesi, verileri Stream Analytics 'e iter.
7. Stream Analytics verileri toplar ve Power BI gönderir.

## <a name="components"></a>Bileşenler

Bu çözüm aşağıdaki bileşenleri kullanır:

| Katman | Bileşen | Description |
|----------|-----------|-------------|
| Mağaza içi donanım | [Özel Görüntü İşleme AI geliştirme seti](https://azure.github.io/Vision-AI-DevKit-Pages/) | Yalnızca analiz için kişilerin görüntülerini yakalayan yerel bir ML modeli kullanarak mağaza içi filtreleme sağlar. IoT Hub aracılığıyla güvenli bir şekilde sağlanmış ve güncelleştirilmiş.<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | Azure Event Hubs, Azure Stream Analytics ile düzenli olarak tümleşen anonim verileri almak için ölçeklenebilir bir platform sağlar. |
|  | [Azure Akış Analizi](/azure/stream-analytics/) | Azure Stream Analytics bir iş, anonimleştirilmiş verileri toplar ve görselleştirme için 15 saniyelik Windows ile gruplandırır. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI, çıktıyı Azure Stream Analytics görüntülemek için kullanımı kolay bir pano arabirimi sağlar. |
| Azure Stack hub 'ı | [App Service](/azure-stack/operator/azure-stack-app-service-overview.md) | App Service kaynak sağlayıcısı (RP), Web Apps/API ve Işlevlerine yönelik barındırma ve yönetim özellikleri de dahil olmak üzere Edge bileşenleri için bir temel sağlar. |
| | Azure Kubernetes hizmeti [(aks) altyapısı](https://github.com/Azure/aks-engine) kümesi | Azure Stack hub 'ına dağıtılan AKS-Engine kümesi ile AKS RP, Yüz Tanıma API'si kapsayıcısını çalıştırmak için ölçeklenebilir ve dayanıklı bir altyapı sağlar. |
| | Azure bilişsel Hizmetler [Yüz Tanıma API'si kapsayıcıları](/azure/cognitive-services/face/face-how-to-install-containers)| Yüz Tanıma API'si kapsayıcılarıyla Azure bilişsel hizmetler RP, contoso 'nun özel ağında demografik, duygu ve benzersiz ziyaretçi algılaması sağlar. |
| | Blob Depolama | AI Dev Kit 'ten yakalanan görüntüler Azure Stack hub 'ın BLOB depolama alanına yüklenir. |
| | Azure İşlevleri | Azure Stack Hub üzerinde çalışan bir Azure Işlevi, blob depolamadan giriş alır ve Yüz Tanıma API'si etkileşimleri yönetir. Anonim verileri Azure 'da bulunan bir Event Hubs kümesine yayar.<br><br>|

## <a name="issues-and-considerations"></a>Sorunlar ve dikkat edilmesi gerekenler

Bu çözümü nasıl uygulayacağınıza karar verirken aşağıdaki noktaları göz önünde bulundurun:

### <a name="scalability"></a>Ölçeklenebilirlik

Bu çözümün birden çok kamera ve konumda ölçeklendirilmesini sağlamak için tüm bileşenlerin daha fazla yükü işleyebilmesini sağlamak gerekir. Şunlar gibi eylemler gerçekleştirmeniz gerekebilir:

- Stream Analytics akış birimi sayısını artırın.
- Yüz Tanıma API'si dağıtımını ölçeklendirin.
- Event Hubs kümesi verimini artırın.
- Olağanüstü durumlarda Azure Işlevlerinden bir sanal makineye geçiş yapmanız gerekebilir.

### <a name="availability"></a>Kullanılabilirlik

Bu çözüm katmanlı olduğundan, ağ veya güç arızalarının nasıl ele alınacağını düşünmek önemlidir. İş ihtiyaçlarına bağlı olarak, görüntüleri yerel olarak önbelleğe almak için bir mekanizma uygulamak, sonra bağlantı döndürüldüğünde Azure Stack hub 'a iletmek isteyebilirsiniz. Konum yeterince büyükse, bu konuma Yüz Tanıma API'si kapsayıcı ile Data Box Edge dağıtımı daha iyi bir seçenek olabilir.

### <a name="manageability"></a>Yönetilebilirlik

Bu çözüm, çok sayıda cihaza ve konuma yayılabilir ve bu da bir süre sonra olabilir. [Azure 'un IoT Hizmetleri](/azure/iot-fundamentals/) , yeni konumları ve cihazları otomatik olarak çevrimiçi hale getirmek ve güncel tutmak için kullanılabilir.

### <a name="security"></a>Güvenlik

Bu çözüm, müşteri görüntülerini yakalar ve güvenlik için bir parametre dikkate alır. Tüm depolama hesaplarının uygun erişim ilkeleriyle güvenli olduğundan emin olun ve anahtarları düzenli olarak döndürün. Depolama hesaplarının ve Event Hubs kurumsal ve kamu gizlilik düzenlemelerine uyan bekletme ilkelerine sahip olduğundan emin olun. Ayrıca, Kullanıcı erişim seviyelerini katmandığınızdan emin olun. Katmanlama, kullanıcıların yalnızca rolleri için ihtiyaç duydukları verilere erişmesini sağlar.

## <a name="next-steps"></a>Sonraki adımlar

Bu makalede tanıtılan konular hakkında daha fazla bilgi edinmek için:

- Bkz. yararlanılabilir [,,,](https://aka.ms/tiereddatadeploy),
- Özel Vision kullanma hakkında daha fazla bilgi edinmek için bkz. [özel görüntü işleme AI geliştirme seti](https://azure.github.io/Vision-AI-DevKit-Pages/) . 

Çözüm örneğini test etmeye hazırsanız, [Betfall algılama dağıtım kılavuzu](solution-deployment-guide-retail-footfall-detection.md)ile devam edin. Dağıtım Kılavuzu, bileşenlerinin dağıtılması ve test edilmesi için adım adım yönergeler sağlar.
