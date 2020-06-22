---
title: Azure ve Azure Stack hub 'daki karma geçiş deseninin
description: Güvenlik duvarları tarafından korunan Edge kaynaklarına bağlanmak için Azure ve Azure Stack hub 'daki karma geçiş modelini kullanın.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911971"
---
# <a name="hybrid-relay-pattern"></a>Karma geçiş deseninin

Karma geçiş modelini ve Azure Relay kullanarak güvenlik duvarları tarafından korunan Edge kaynaklarına veya cihazlara nasıl bağlanacağınızı öğrenin.

## <a name="context-and-problem"></a>Bağlam ve sorun

Sınır cihazları çoğunlukla bir kurumsal güvenlik duvarı veya NAT cihazının arkasında bulunur. Güvenli olsalar da, diğer şirket ağlarında ortak bulut veya uç cihazlarla iletişim kuramayabilirler. Genel buluttaki kullanıcılara güvenli bir şekilde belirli bağlantı noktalarını ve işlevleri göstermek gerekebilir.

## <a name="solution"></a>Çözüm

Karma geçiş modelinde, doğrudan iletişim kuramayan iki uç nokta arasında bir WebSockets tüneli kurmak için Azure Relay kullanılır. Şirket içinde olmayan ancak şirket içi bir uç noktaya bağlanması gereken cihazlar, genel buluttaki bir uç noktaya bağlanır. Bu uç nokta, güvenli bir kanal üzerinden önceden tanımlı yolların trafiğini yeniden yönlendirir. Şirket içi ortamın içindeki bir uç nokta trafiği alır ve doğru hedefe yönlendirir.

![Karma geçiş deseninin çözüm mimarisi](media/pattern-hybrid-relay/solution-architecture.png)

Hibrit geçiş deseninin nasıl çalıştığı aşağıda verilmiştir:

1. Bir cihaz, önceden tanımlanmış bir bağlantı noktasında Azure 'daki sanal makineye (VM) bağlanır.
2. Trafik Azure 'daki Azure Relay iletilir.
3. Zaten Azure Relay uzun süreli bir bağlantı oluşturmuş Azure Stack hub 'ındaki VM, trafiği alır ve hedefe iletir.
4. Şirket içi hizmet veya uç nokta, isteği işler.

## <a name="components"></a>Bileşenler

Bu çözüm aşağıdaki bileşenleri kullanır:

| Katman | Bileşen | Description |
|----------|-----------|-------------|
| Azure | Azure VM | Azure VM, şirket içi kaynak için genel olarak erişilebilen bir uç nokta sağlar. |
| | Azure Geçişi | [Azure Relay](/azure/azure-relay/) , Azure vm Ile Azure Stack hub sanal makinesi arasındaki tüneli ve bağlantıyı sürdürmek için altyapı sağlar.|
| Azure Stack hub 'ı | İşlem | Azure Stack hub sanal makinesi, karma geçiş tünelinin sunucu tarafını sağlar. |
| | Depolama | Azure Stack hub 'ına dağıtılan AKS motoru kümesi, Yüz Tanıma API'si kapsayıcısını çalıştırmak için ölçeklenebilir ve dayanıklı bir altyapı sağlar.|

## <a name="issues-and-considerations"></a>Sorunlar ve dikkat edilmesi gerekenler

Bu çözümü nasıl uygulayacağınıza karar verirken aşağıdaki noktaları göz önünde bulundurun:

### <a name="scalability"></a>Ölçeklenebilirlik

Bu model yalnızca istemci ve sunucuda 1:1 bağlantı noktası eşlemelerine izin verir. Örneğin, bağlantı noktası 80, Azure uç noktasındaki bir hizmet için tünededir, başka bir hizmet için kullanılamaz. Bağlantı noktası eşlemeleri uygun şekilde planlanmalıdır. Azure Relay ve VM 'Ler trafiği işlemek için uygun şekilde ölçeklendirmelidir.

### <a name="availability"></a>Kullanılabilirlik

Bu tüneller ve bağlantılar gereksiz değildir. Yüksek kullanılabilirlik sağlamak için, hata denetimi kodu uygulamak isteyebilirsiniz. Başka bir seçenek de, yük dengeleyicinin arkasında Azure Relay bağlı VM havuzu olmalıdır.

### <a name="manageability"></a>Yönetilebilirlik

Bu çözüm, çok sayıda cihaza ve konuma yayılabilir ve bu da bir süre sonra olabilir. Azure 'un IoT Hizmetleri, yeni konumları ve cihazları otomatik olarak çevrimiçi duruma getirebilir ve güncel tutar.

### <a name="security"></a>Güvenlik

Bu model, uçtan bir iç cihazdaki bağlantı noktasına sınırsız erişimine izin verir. İç cihazdaki hizmete veya karma geçiş uç noktasının önüne bir kimlik doğrulama mekanizması eklemeyi düşünün.

## <a name="next-steps"></a>Sonraki adımlar

Bu makalede tanıtılan konular hakkında daha fazla bilgi edinmek için:

- Bu kalıp Azure Relay kullanır. Daha fazla bilgi için [Azure Relay belgelerine](/azure/azure-relay/)bakın.
- En iyi uygulamalar hakkında daha fazla bilgi edinmek ve ek sorulara yanıt almak için bkz. [karma uygulama tasarımı konuları](overview-app-design-considerations.md) .
- Ürünlerin ve çözümlerin tamamı hakkında daha fazla bilgi edinmek için [ürün ve çözümlerin Azure Stack ailesine](/azure-stack) bakın.

Çözüm örneğini test etmeye hazırsanız [karma geçiş çözümü dağıtım kılavuzu](https://aka.ms/hybridrelaydeployment)ile devam edin. Dağıtım Kılavuzu, bileşenlerinin dağıtılması ve test edilmesi için adım adım yönergeler sağlar.