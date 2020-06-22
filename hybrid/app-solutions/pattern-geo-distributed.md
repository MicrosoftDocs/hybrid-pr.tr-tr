---
title: Azure Stack hub 'da coğrafi olarak dağıtılmış uygulama deseninin
description: Azure ve Azure Stack hub kullanarak akıllı kenar için coğrafi olarak dağıtılmış uygulama düzeniyle ilgili bilgi edinin.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911857"
---
# <a name="geo-distributed-app-pattern"></a>Coğrafi olarak dağıtılmış uygulama kalıbı

Birden çok bölgede uygulama uç noktaları sağlamayı ve Kullanıcı trafiğini konum ve uyumluluk ihtiyaçlarına göre yönlendirme hakkında bilgi edinin.

## <a name="context-and-problem"></a>Bağlam ve sorun

Geniş erişime sahip kuruluşlar, her Kullanıcı, konum ve bir cihaz için gerekli güvenlik, uyumluluk ve performans düzeylerini sağlarken verilere erişimi güvenle ve doğru bir şekilde dağıtabilir ve etkinleştirir.

## <a name="solution"></a>Çözüm

Azure Stack hub coğrafi trafik yönlendirme deseninin veya coğrafi olarak Dağıtılmış uygulamaların, trafiğin çeşitli ölçümlere bağlı olarak belirli uç noktalara yönlendirilmelerini sağlar. Coğrafi tabanlı Yönlendirme ve uç nokta yapılandırması ile Traffic Manager oluşturmak, bölgesel gereksinimlere, şirkete ve uluslararası yönetmelere ve veri ihtiyaçlarına göre trafiği uç noktalara yönlendirir.

![Coğrafi olarak dağıtılmış desenler](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>Bileşenler

### <a name="outside-the-cloud"></a>Bulutun dışında

#### <a name="traffic-manager"></a>Traffic Manager

Diyagramda Traffic Manager, genel bulutun dışında bulunur, ancak hem yerel veri merkezinde hem de genel bulutta trafiği koordine edebilmelidir. Dengeleyici trafiği coğrafi konumlara yönlendirir.

#### <a name="domain-name-system-dns"></a>Etki Alanı Adı Sistemi (DNS)

Etki alanı adı sistemi veya DNS, IP adresine bir Web sitesi veya hizmet adı çevirmekten (veya çözümlemeden) sorumludur.

### <a name="public-cloud"></a>Genel bulut

#### <a name="cloud-endpoint"></a>Bulut uç noktası

Genel IP adresleri, Trafik Yöneticisi üzerinden gelen trafiği genel bulut uygulama kaynakları uç noktasına yönlendirmek için kullanılır.  

### <a name="local-clouds"></a>Yerel bulutlar

#### <a name="local-endpoint"></a>Yerel uç nokta

Genel IP adresleri, Trafik Yöneticisi üzerinden gelen trafiği genel bulut uygulama kaynakları uç noktasına yönlendirmek için kullanılır.

## <a name="issues-and-considerations"></a>Sorunlar ve dikkat edilmesi gerekenler

Bu düzenin nasıl uygulanacağına karar verirken aşağıdaki noktaları göz önünde bulundurun:

### <a name="scalability"></a>Ölçeklenebilirlik

Bu model, trafiğin artışlarını karşılamak için ölçeklendirilmesi yerine coğrafi trafik yönlendirmeyi işler. Ancak, bu stili diğer Azure ve şirket içi çözümlerle birleştirebilirsiniz. Örneğin, bu model, platformlar arası ölçeklendirme düzeniyle birlikte kullanılabilir.

### <a name="availability"></a>Kullanılabilirlik

Yerel olarak dağıtılan uygulamaların şirket içi donanım yapılandırması ve yazılım dağıtımı aracılığıyla yüksek kullanılabilirlik için yapılandırıldığından emin olun.

### <a name="manageability"></a>Yönetilebilirlik

Bu model, ortamlar arasında sorunsuz yönetim ve tanıdık arabirim sağlar.

## <a name="when-to-use-this-pattern"></a>Bu düzenin kullanılacağı durumlar

- Kuruluşumun özel bölgesel güvenlik ve dağıtım ilkeleri gerektiren Uluslararası dalları vardır.
- Kuruluş ofislerimin her biri, her yerel yönetmeliğe ve saat dilimine göre raporlama etkinliği gerektiren çalışan, iş ve tesis verilerini çeker.
- Yüksek ölçekli gereksinimler, tek bir bölgede ve bölgeler arasında birden fazla uygulama dağıtımı, çok fazla yük gereksinimini işleyecek şekilde, uygulamalar tarafından yatay olarak ölçeklendirilirken karşılandı.
- Uygulamalar, tek bölgede kesintiler halinde bile yüksek oranda kullanılabilir ve istemci isteklerine yanıt vermelidir.

## <a name="next-steps"></a>Sonraki adımlar

Bu makalede tanıtılan konular hakkında daha fazla bilgi edinmek için:

- Bu DNS tabanlı trafik yük dengeleyicinin nasıl çalıştığı hakkında daha fazla bilgi edinmek için bkz. [Azure Traffic Manager genel bakış](/azure/traffic-manager/traffic-manager-overview) .
- En iyi uygulamalar hakkında daha fazla bilgi edinmek ve ek sorulara yanıt almak için bkz. [karma uygulama tasarımı konuları](overview-app-design-considerations.md) .
- Ürünlerin ve çözümlerin tamamı hakkında daha fazla bilgi edinmek için [ürün ve çözümlerin Azure Stack ailesine](/azure-stack) bakın.

Çözüm örneğini test etmeye hazırsanız, [coğrafi olarak dağıtılmış uygulama çözümü dağıtım kılavuzu](solution-deployment-guide-geo-distributed.md)ile devam edin. Dağıtım Kılavuzu, bileşenlerinin dağıtılması ve test edilmesi için adım adım yönergeler sağlar. Coğrafi olarak dağıtılmış uygulama modelini kullanarak çeşitli ölçümlere göre trafiği belirli uç noktalara yönlendirirsiniz. Coğrafi tabanlı Yönlendirme ve uç nokta yapılandırmasıyla Traffic Manager profili oluşturma, bilgilerin bölgesel gereksinimlere, şirkete ve uluslararası yönetmelere ve veri gereksinimlerinize göre uç noktalara yönlendirilmesini sağlar.
