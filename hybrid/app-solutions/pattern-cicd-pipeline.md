---
title: Azure Stack hub 'daki DevOps deseninin
description: Azure 'da ve Azure Stack hub 'da dağıtımlar arasında tutarlılık sağlamak için DevOps düzeniyle ilgili bilgi edinin.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: e26056a9507a7467473b009725d4f210d9d59ec8
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477244"
---
# <a name="devops-pattern"></a>DevOps kalıbı

Tek bir konumdan kodlayın ve yerel veri merkezinizde, özel bulutlarınızda veya genel bulutta olabilecek geliştirme, test ve üretim ortamlarında birden çok hedefe dağıtın.

## <a name="context-and-problem"></a>Bağlam ve sorun

Uygulama dağıtımı sürekliliği, güvenlik ve güvenilirlik, kuruluşlar ve geliştirme ekipleri için önemli öneme sahiptir.

Uygulamalar genellikle yeniden düzenlenmiş kodunun her bir hedef ortamda çalıştırılmasını gerektirir. Bu, bir uygulamanın tamamen taşınmadığı anlamına gelir. Her bir ortamda gezindiğinden, güncelleştirilmeleri, test edilmiş ve doğrulanması gerekir. Örneğin, bir geliştirme ortamında yazılan kodun daha sonra bir test ortamında çalışması ve son olarak bir üretim ortamına eklerken yeniden yazılması gerekir. Ayrıca, bu kod özellikle konağa bağlanır. Bu, uygulamanızı korumanın maliyetini ve karmaşıklığını artırır. Uygulamanın her bir sürümü her bir ortama bağlıdır. Artan karmaşıklık ve yineleme, güvenlik ve kod kalitesi riskini artırır. Ayrıca, geri yükleme başarısız Konakları kaldırdığınızda veya daha fazla artış sağlamak üzere ek konaklar dağıtırken kod kolayca dağıtılamaz.

## <a name="solution"></a>Çözüm

DevOps stili, birden çok bulutta çalışan bir uygulama oluşturmanıza, test etmenize ve dağıtmanıza olanak sağlar. Bu model sürekli tümleştirme ve sürekli teslim işlemlerini tek bir şekilde depolar. Sürekli tümleştirme sayesinde, bir takım üyesi sürüm denetimine bir değişiklik yaptığı her seferinde kod oluşturulur ve test edilir. Sürekli teslim, bir derlemeden bir üretim ortamına her adımı otomatikleştirir. Birlikte, bu işlemler, farklı ortamlarda dağıtımı destekleyen bir yayın işlemi oluşturur. Bu Düzenle, kodunuzun taslağını oluşturabilir ve ardından aynı kodu bir şirket içi ortama, farklı özel bulutlara ve genel bulutlara dağıtabilirsiniz. Ortamdaki farklılıklar, koddaki değişiklikler yerine yapılandırma dosyasında değişiklik yapılmasını gerektirir.

![DevOps kalıbı](media/pattern-cicd-pipeline/hybrid-ci-cd.png)

Şirket içi, özel bulut ve genel bulut ortamlarında tutarlı bir geliştirme araçları kümesiyle sürekli tümleştirme ve sürekli teslim için bir yöntem uygulayabilirsiniz. DevOps model kullanılarak dağıtılan uygulamalar ve hizmetler, şirket içi ve genel bulut özelliklerinden ve özelliklerinden faydalanarak bu konumların herhangi birinde değiştirilebilir ve çalıştırılabilir.

DevOps yayın işlem hattının kullanılması şunları yapmanıza yardımcı olur:

- Tek bir depoya kod işleme tabanlı yeni bir derleme başlatın.
- Yeni oluşturulan kodunuzu kullanıcı kabul testi için genel buluta otomatik olarak dağıtın.
- Kodunuz testi geçtikten sonra özel buluta otomatik olarak dağıtın.

## <a name="issues-and-considerations"></a>Sorunlar ve dikkat edilmesi gerekenler

DevOps deseninin, hedef ortamdan bağımsız olarak dağıtımlar arasında tutarlılık sağlamaya yöneliktir. Ancak, yetenekler bulutta ve şirket içi ortamlarda farklılık gösterir. Aşağıdaki noktaları göz önünde bulundurun:

- Dağıtımınızdaki işlevler, uç noktalar, hizmetler ve diğer kaynaklar hedef dağıtım konumlarında kullanılabilir mi?
- Yapılandırma yapıtları bulutlar genelinde erişilebilen konumlarda depolanıyor mu?
- Dağıtım parametreleri tüm hedef ortamlarda çalışacak mı?
- Kaynağa özgü özellikler tüm hedef bulutlarda kullanılabilir mi?

Daha fazla bilgi için bkz. [bulut tutarlılığı için Azure Resource Manager şablonları geliştirme](/azure/azure-resource-manager/templates-cloud-consistency).

Ayrıca, bu düzenin nasıl uygulanacağını saptarken aşağıdaki noktaları göz önünde bulundurun:

### <a name="scalability"></a>Ölçeklenebilirlik

Dağıtım Otomasyonu sistemleri, DevOps desenlerindeki anahtar denetim noktasıdır. Uygulamalar farklılık gösterebilir. Doğru sunucu boyutunun seçimi, beklenen iş yükünün boyutuna bağlıdır. VM 'Ler kapsayıcılardan ölçeğe daha fazla maliyet sağlar. Ancak, ölçeklendirme amacıyla kapsayıcıları kullanabilmek için derleme sürecinizin kapsayıcılar ile çalışması gerekir.

### <a name="availability"></a>Kullanılabilirlik

Devmodel bağlamında kullanılabilirlik, test sonuçları, kod bağımlılıkları veya diğer yapıtlar gibi iş akışınız ile ilişkili tüm durum bilgilerini kurtarabilmekte olduğu anlamına gelir. Kullanılabilirlik gereksinimlerinizi değerlendirirken yaygın kullanılan iki ölçümü göz önünde bulundurun:

- Kurtarma süresi hedefi (RTO), sistem olmadan ne kadar gidebilmeniz gerektiğini belirtir.

- Kurtarma noktası hedefi (RPO), hizmette bir kesinti sistemi etkiliyorsa ne kadar veri kaybedeceğinizi gösterir.

Uygulamada, RTO ve RPO, artıklık ve yedekleme anlamına gelmez. Küresel Azure bulutu 'nda kullanılabilirlik, Azure 'un bir parçası olan, ancak DevOps sistemlerinizin durumunu sürdürmenize gerek kalmadan, donanım kurtarmasının bir sorusu değildir. Azure Stack hub 'ında donanım kurtarma bir göz önünde bulunabilir.

Dağıtım Otomasyonu için kullanılan sistemin tasarlanmasıyla ilgili bir diğer önemli nokta, erişim denetimi ve hizmetleri bulut ortamlarına dağıtmak için gereken hakların uygun yönetimi. Dağıtımları oluşturmak, silmek veya değiştirmek için hangi haklar gerekir? Örneğin, genellikle Azure 'da bir kaynak grubu oluşturmak için bir haklar kümesi gerekir ve kaynak grubunda hizmetleri dağıtmak için başka bir haklar vardır.

### <a name="manageability"></a>Yönetilebilirlik

DevOps düzenine dayalı herhangi bir sistemin tasarımı, portföyde her bir hizmet için Otomasyon, günlüğe kaydetme ve uyarı vermeyi göz önünde bulundurmalıdır. Paylaşılan Hizmetleri, bir uygulama ekibini veya her ikisini birden kullanın, güvenlik ilkelerini ve idare 'yi de izleyin.

Üretim ortamlarını ve geliştirme/test ortamlarını Azure veya Azure Stack hub 'daki ayrı kaynak gruplarında dağıtın. Ardından, her bir ortamın kaynaklarını izleyebilir ve kaynak grubuna göre faturalandırma maliyetlerini oluşturabilirsiniz. Kaynakları, test dağıtımları için yararlı olan bir küme olarak da silebilirsiniz.

## <a name="when-to-use-this-pattern"></a>Bu düzenin kullanılacağı durumlar

Şu durumlarda bu kalıbı kullanın:

- Geliştiricilerinizin ihtiyaçlarını karşılayan bir ortamda kod geliştirebilir ve yeni kod geliştirmek zor olabilecek çözümünüze özel bir ortama dağıtabilirsiniz.
- DevOps deseninin sürekli tümleştirme ve sürekli teslim sürecini izleyebilecekleri sürece, geliştiricilerinizin beğendikleri kod ve araçları kullanabilirsiniz.

Bu düzenin şunlar için kullanılması önerilmez:

- Altyapı, sağlama kaynakları, yapılandırma, kimlik ve güvenlik görevlerini otomatikleştiremiyorum.
- Ekiplerin sürekli tümleştirme/sürekli geliştirme (CI/CD) yaklaşımı uygulamak için karma bulut kaynaklarına erişimleri yoksa.

## <a name="next-steps"></a>Sonraki adımlar

Bu makalede tanıtılan konular hakkında daha fazla bilgi edinmek için:

- Azure Repos ve Azure Pipelines dahil olmak üzere Azure DevOps ve ilgili araçlar hakkında daha fazla bilgi edinmek için bkz. [Azure DevOps belgeleri](/azure/devops) .
- Ürünlerin ve çözümlerin tamamı hakkında daha fazla bilgi edinmek için [ürün ve çözümlerin Azure Stack ailesine](/azure-stack) bakın.

Çözüm örneğini test etmeye hazırsanız, [DevOps karma CI/CD çözümü dağıtım kılavuzu](https://aka.ms/hybriddevopsdeploy)ile devam edin. Dağıtım Kılavuzu, bileşenlerinin dağıtılması ve test edilmesi için adım adım yönergeler sağlar. Karma bir sürekli tümleştirme/sürekli teslim (CI/CD) işlem hattı kullanarak bir uygulamayı Azure 'a ve Azure Stack hub 'a dağıtmayı öğreneceksiniz.
