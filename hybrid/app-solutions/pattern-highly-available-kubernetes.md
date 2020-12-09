---
title: Azure ve Azure Stack hub kullanarak yüksek kullanılabilirliğe sahip Kubernetes deseninin
description: Bir Kubernetes küme çözümünün Azure ve Azure Stack hub kullanarak yüksek kullanılabilirlik sağladığını öğrenin.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 454cc0a0531882b7a8ec050a461420ce13eebcfe
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911796"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>Yüksek kullanılabilirlik Kubernetes küme deseninin

Bu makalede, Azure Stack hub 'ında Azure Kubernetes hizmeti (AKS) altyapısını kullanarak yüksek kullanılabilirliğe sahip bir Kubernetes tabanlı altyapının nasıl mimarileceği ve çalıştırılacağı açıklanır. Bu senaryo, yüksek oranda kısıtlı ve düzenlenmiş ortamlarda kritik iş yükleri olan kuruluşlar için yaygındır. Finans, savunma ve kamu gibi etki alanlarındaki kuruluşlar.

## <a name="context-and-problem"></a>Bağlam ve sorun

Birçok kuruluş, Kubernetes gibi son teknoloji hizmetlerinden ve teknolojilerinden yararlanan bulutta yerel çözümler geliştirmektedir. Azure, dünyanın çoğu bölgesinde veri merkezleri sağladığından, bazen iş açısından kritik uygulamaların belirli bir konumda çalışması gerektiği durumlar ve senaryolar da vardır. Şunlar arasında dikkat edilecek noktalar:

- Konum duyarlılığı
- Uygulama ve şirket içi sistemler arasındaki gecikme süresi
- Bant genişliği koruma
- Bağlantı
- Mevzuata veya yasal gereksinimler

Azure, Azure Stack hub ile birlikte bu kaygıların çoğunu ele alınmaktadır. Azure Stack Hub üzerinde çalışan bir Kubernetes uygulamasının başarıyla uygulanması için çok sayıda seçenek, kararlar ve dikkat edilecek noktalar aşağıda açıklanmıştır.

## <a name="solution"></a>Çözüm

Bu düzende, katı bir kısıtlama kümesiyle uğraşmak zorunda olduğumuz varsayılmaktadır. Uygulamanın şirket içinde çalışması ve tüm kişisel verilerin genel bulut hizmetlerine ulaşamamalıdır. İzleme ve diğer PII olmayan veriler Azure 'a gönderilebilir ve burada işlenebilirler. Herkese açık bir Container Registry veya başka bir deyişle, bir güvenlik duvarı veya ara sunucu üzerinden filtrelenmiş olabilir.

Burada gösterilen örnek uygulama ( [Azure Kubernetes Service Workshop](/learn/modules/aks-workshop/)temelinde), mümkün olduğunda Kubernetes yerel çözümlerini kullanmak için tasarlanmıştır. Bu tasarım, platform yerel hizmetleri yerine satıcı kilitinin kullanılmasını önler. Örnek olarak, uygulama PaaS hizmeti veya dış veritabanı hizmeti yerine şirket içinde barındırılan bir MongoDB veritabanı arka ucu kullanır.

[![Uygulama deseninin karma](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

Yukarıdaki diyagramda, Azure Stack hub 'da Kubernetes üzerinde çalışan örnek uygulamanın uygulama mimarisi gösterilmektedir. Uygulama aşağıdakiler dahil olmak üzere çeşitli bileşenlerden oluşur:

 1) Azure Stack hub 'da Kubernetes kümesi tabanlı AKS motoru.
 2) Kubernetes 'te sertifika yönetimi için bir araç paketi sunan [CERT-Manager](https://www.jetstack.io/cert-manager/), izin verenden otomatik olarak sertifika istemek için kullanılır.
 3) Ön uç (derecelendirme-Web), API (derecelendirme-API) ve veritabanı (derecelendirmeler-MongoDB) için uygulama bileşenlerini içeren bir Kubernetes ad alanı.
 4) HTTP/HTTPS trafiğini Kubernetes kümesi içindeki uç noktalara yönlendiren giriş denetleyicisi.

Örnek uygulama, uygulama mimarisini göstermek için kullanılır. Tüm bileşenler örnektir. Mimari yalnızca tek bir uygulama dağıtımı içerir. Yüksek kullanılabilirlik (HA) sağlamak için dağıtımı iki farklı Azure Stack hub örneğinde en az iki kez çalıştıracağız; aynı konumda veya iki (veya daha fazla) farklı sitede çalıştırılabilir:

![Altyapı mimarisi](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Azure Container Registry, Azure Izleyici ve diğerleri gibi hizmetler, Azure 'da veya şirket içinde Azure Stack hub 'ın dışında barındırılır. Bu hibrit tasarımı, çözümü tek bir Azure Stack hub örneğinin kesintiine karşı korur.

## <a name="components"></a>Bileşenler

Genel mimari aşağıdaki bileşenlerden oluşur:

**Azure Stack hub** , veri merkezinizde Azure hizmetleri sağlayarak Şirket içi ortamda iş yüklerini çalıştırabilecekleri bir Azure uzantısıdır. Daha fazla bilgi edinmek için [Azure Stack hub 'a genel bakış](/azure-stack/operator/azure-stack-overview) bölümüne gidin.

**Azure Kubernetes hizmet altyapısı (aks motoru)** , bugün Azure 'da kullanılabilen, yönetilen Kubernetes hizmet teklifi 'nın (aks) Azure Kubernetes hizmet sunumunun arkasındaki altyapıdır. Azure Stack hub 'ı için AKS motoru, Azure Stack hub 'ın IaaS yeteneklerini kullanarak tam özellikli, otomatik olarak yönetilen Kubernetes kümelerini dağıtmamıza, ölçeklendirmenize ve yükseltmemize olanak sağlar. Daha fazla bilgi edinmek için [aks altyapısına genel bakış](https://github.com/Azure/aks-engine) bölümüne gidin.

Azure Stack hub 'ındaki Azure ve AKS altyapısındaki AKS motoru arasındaki farklar hakkında daha fazla bilgi edinmek için [bilinen sorunlara ve sınırlamalara](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations) gidin.

**Azure sanal ağı (VNet)** , Kubernetes küme altyapısını barındıran sanal makineler (VM) için her bir Azure Stack hub 'ında ağ altyapısını sağlamak için kullanılır.

**Azure Load Balancer** , Kubernetes API uç noktası ve NGINX giriş denetleyicisi için kullanılır. Yük dengeleyici, belirli bir hizmeti sunan düğümlere ve VM 'lere dış (örneğin, Internet) trafiği yönlendirir.

**Azure Container Registry (ACR)** , kümeye dağıtılan özel Docker görüntülerini ve Helm grafiklerini depolamak için kullanılır. AKS motoru, Azure AD kimliği kullanarak Container Registry kimlik doğrulaması yapabilir. Kubernetes için ACR gerekmez. Docker Hub gibi başka kapsayıcı kayıt defterleri de kullanabilirsiniz.

**Azure Repos** , kodunuzu yönetmek için kullanabileceğiniz bir sürüm denetim araçları kümesidir. GitHub veya diğer git tabanlı depoları da kullanabilirsiniz. Daha fazla bilgi edinmek için [Azure Repos genel bakış](/azure/devops/repos/get-started/what-is-repos) bölümüne gidin.

**Azure Pipelines** , Azure DevOps Services bir parçasıdır ve otomatikleştirilmiş derlemeler, testler ve dağıtımlar çalıştırır. Jenkins gibi üçüncü taraf CI/CD çözümlerini de kullanabilirsiniz. Daha fazla bilgi edinmek için Azure işlem hattına [genel bakış](/azure/devops/pipelines/get-started/what-is-azure-pipelines) bölümüne gidin.

**Azure izleyici** , çözüm ve uygulama telemetrisi içindeki Azure hizmetleri için platform ölçümleri dahil olmak üzere ölçümleri ve günlükleri toplar ve depolar. Bu verileri kullanarak uygulamayı izleyin, uyarıları ve panoları ayarlayın ve hataların kök neden analizini yapın. Azure Izleyici, denetleyiciler, düğümler ve kapsayıcılardan ve kapsayıcı günlükleri ile ana düğüm günlüklerinden ölçüm toplamak için Kubernetes ile tümleşir. Daha fazla bilgi edinmek için [Azure izleyici 'ye genel bakış](/azure/azure-monitor/overview) bölümüne gidin.

**Azure Traffic Manager** , trafiği farklı Azure bölgelerindeki veya Azure Stack hub dağıtımlarında en iyi şekilde dağıtmanıza olanak sağlayan DNS tabanlı bir trafik yük dengeleyicidir. Traffic Manager ayrıca yüksek kullanılabilirlik ve yanıt hızı sağlar. Uygulama uç noktalarına dışarıdan erişebilmelidir. Diğer şirket içi çözümler de mevcuttur.

**Kubernetes giriş denetleyicisi** , bir Kubernetes KÜMESINDEKI hizmetlere http (S) yolları sunar. NGINX veya uygun giriş denetleyicisi bu amaçla kullanılabilir.

**Helm** , bir Kubernetes dağıtımı için Paket Yöneticisi, dağıtımlar, hizmetler, gizlilikler gibi farklı Kubernetes nesnelerini tek bir "grafik" olarak paketlemeyi sağlayan bir yol sağlar. Sürüm yönetimini yayımlayabilir, dağıtabilir, denetleyebilir ve bir grafik nesnesini güncelleştirebilirsiniz. Azure Container Registry, paketlenmiş Held grafiklerini depolamak için bir depo olarak kullanılabilir.

## <a name="design-considerations"></a>Tasarım konusunda dikkat edilmesi gerekenler

Bu model, bu makalenin sonraki bölümlerinde daha ayrıntılı olarak açıklanan bazı üst düzey konuları izler:

- Uygulama, satıcı kilitlemeyi önlemek için Kubernetes-Native çözümlerini kullanır.
- Uygulama, mikro hizmet mimarisini kullanır.
- Azure Stack hub 'A gelen bir bağlantı gerekmez, ancak giden Internet bağlantısına izin verir.

Bu önerilen uygulamalar, gerçek dünyada iş yükleri ve senaryolar için de geçerlidir.

## <a name="scalability-considerations"></a>Ölçeklenebilirlik konusunda dikkat edilmesi gerekenler

Ölçeklenebilirlik, kullanıcılara tutarlı, güvenilir ve iyi uygulama erişimi sağlamak için önemlidir.

Örnek senaryo, uygulama yığınının birden çok katmanında ölçeklenebilirliği ele alır. Farklı katmanlara yönelik üst düzey bir genel bakış aşağıda verilmiştir:

| Mimari düzeyi | Ekranlarını | Nasıl yapabilirim? |
| --- | --- | ---
| Uygulama | Uygulama | Pods/çoğaltmalar sayısına göre yatay ölçekleme/Container Instances * |
| Küme | Kubernetes kümesi | Düğüm sayısı (1 ile 50 arasında), VM-SKU-Boyutlar ve düğüm havuzları (Azure Stack hub 'daki AKS altyapısı şu anda yalnızca tek bir düğüm havuzunu desteklemektedir); AKS altyapısının Scale komutunu kullanma (el ile) |
| Altyapı | Azure Stack Hub | Azure Stack hub dağıtımı içindeki düğüm, kapasite ve ölçek birimi sayısı |

\* Kubernetes ' yatay Pod otomatik Scaler (HPA) kullanma; kapsayıcı örneklerini (CPU/bellek) boyutlandırarak otomatik ölçüm tabanlı ölçekleme veya dikey ölçekleme.

**Azure Stack Hub (altyapı düzeyi)**

Azure Stack Hub bir veri merkezinde fiziksel donanımda çalıştığı için Azure Stack hub altyapısı bu uygulamanın temelidir. Hub donanımınızı seçerken CPU, bellek yoğunluğu, depolama yapılandırması ve sunucu sayısı için seçimler yapmanız gerekir. Azure Stack hub 'ın ölçeklenebilirliği hakkında daha fazla bilgi edinmek için aşağıdaki kaynaklara göz atın:

- [Azure Stack hub 'a genel bakış için kapasite planlaması](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [Azure Stack hub 'ına ek ölçek birimi düğümleri ekleme](/azure-stack/operator/azure-stack-add-scale-node)

**Kubernetes kümesi (küme düzeyi)**

Kubernetes kümesi, ' den oluşur ve işlem, depolama ve ağ kaynakları dahil olmak üzere Azure (Stack) IaaS bileşenlerinin üzerine kurulmuştur. Kubernetes çözümleri, Azure 'da (ve Azure Stack hub) VM olarak dağıtılan ana ve çalışan düğümlerini içerir.

- [Denetim düzlemi düğümleri](/azure/aks/concepts-clusters-workloads#control-plane) (ana), uygulama iş yüklerinin temel Kubernetes hizmetlerini ve düzenlemesini sağlar.
- [Çalışan düğümleri](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) (çalışan) uygulama iş yüklerinizi çalıştırın.

İlk dağıtım için VM boyutlarını seçerken bazı hususlar vardır:  

- **Maliyet** -çalışan düğümlerinizi planlarken, sanal makine başına genel maliyeti göz önünde bulundurun. Örneğin, uygulama iş yükleriniz sınırlı kaynaklar gerektiriyorsa, daha küçük boyutlu sanal makineler dağıtmayı planlamanız gerekir. Azure gibi Azure Stack hub, normalde bir Tüketim esasına göre faturalandırılır. böylece, Kubernetes rollerinin VM 'lerinin uygun şekilde boyutlandırılması, tüketim maliyetlerini iyileştirmek için önemlidir. 

- **Ölçeklenebilirlik** -kümenin ölçeklenebilirliği, ana ve çalışan düğümlerin sayısını ve sayısını ölçeklendirerek ya da ek düğüm havuzları ekleyerek (bugün Azure Stack hub 'ında kullanılamaz) elde edilir. Kümenin ölçeklendirilmesi, kapsayıcı öngörüleri kullanılarak toplanan performans verilerine göre yapılabilir (Azure Izleyici + Log Analytics). 

    Uygulamanız daha fazla (veya daha az) kaynağa ihtiyaç duyuyorsa, geçerli düğümlerinizi yatay olarak (1 ile 50 düğüm arasında) ölçeklendirebilirsiniz (veya bu düğümleri). 50 ' den fazla düğüme ihtiyacınız varsa, ayrı bir abonelikte ek bir küme oluşturabilirsiniz. Kümeyi yeniden dağıtmaya gerek kalmadan gerçek VM 'Leri dikey olarak başka bir VM boyutuna ölçeklendirebilirsiniz.

    Ölçeklendirme, başlangıçta Kubernetes kümesini dağıtmak için kullanılan AKS motoru yardımcı VM kullanılarak el ile yapılır. Daha fazla bilgi için bkz. [Kubernetes kümelerini ölçeklendirme](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md)

- **Kotalar** -Azure Stack hub 'ınızda aks dağıtımını planlarken yapılandırdığınız [kotaları](/azure-stack/operator/azure-stack-quota-types) değerlendirin. Her [aboneliğin](/azure-stack/operator/service-plan-offer-subscription-overview) uygun planlara ve yapılandırılmış kotalara sahip olduğundan emin olun. Aboneliğin ölçek genişletme sırasında, kümeleriniz için gereken işlem, depolama ve diğer hizmet miktarına uyum sağlaması gerekir.

- **Uygulama iş yükleri** -Azure Kubernetes hizmet belgesi Için Kubernetes temel kavramlarıyla ilgili [kümeler ve iş yükleri kavramlarını](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) inceleyin. Bu makale, uygulamanızın işlem ve bellek ihtiyaçlarına göre doğru VM boyutunu kapsammanıza yardımcı olur.  

**Uygulama (uygulama düzeyi)**

Uygulama katmanında Kubernetes [yatay Pod otomatik Scaler (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)kullanıyoruz. HPA, farklı ölçümlere (CPU kullanımı gibi) bağlı olarak dağıtımımız kopyaların (pod/Container Instances) sayısını artırabilir veya azaltabilir.

Diğer bir seçenek de kapsayıcı örneklerinin dikey olarak ölçeklendirilmesine olanak sağlar. Bu, belirli bir dağıtım için istenen CPU ve bellek miktarı değiştirilerek gerçekleştirilebilir. Daha fazla bilgi için bkz. kubernetes.io üzerinde [kapsayıcılar Için kaynakları yönetme](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) .

## <a name="networking-and-connectivity-considerations"></a>Ağ ve bağlantı konuları

Ağ ve bağlantı, daha önce Azure Stack hub 'ında Kubernetes için bahsedilen üç katmanı da etkiler. Aşağıdaki tabloda, katmanları ve içerdikleri hizmetler gösterilmektedir:

| Katman | Ekranlarını | Ne? |
| --- | --- | ---
| Uygulama | Uygulama | Uygulama nasıl erişilebilir? Internet 'e sunulacak mı? |
| Küme | Kubernetes kümesi | Kubernetes API 'SI, AKS motoru VM, kapsayıcı görüntüleri çekme (çıkış), izleme verileri ve telemetri gönderme (çıkış) |
| Altyapı | Azure Stack Hub | Portal ve Azure Resource Manager uç noktaları gibi Azure Stack merkezi yönetim uç noktaları erişilebilirliği. |

**Uygulama**

Uygulama katmanı için en önemli nokta, uygulamanın Internet 'ten sunulup erişilebilir olup olmadığına göre belirlenir. Bir Kubernetes perspektifinden, Internet erişilebilirliği bir Kubernetes hizmeti veya giriş denetleyicisi kullanarak bir dağıtımı veya Pod 'yi ortaya çıkarmaktadır.

> [!NOTE]
> Azure Stack hub 'daki ön uç genel IP sayısı 5 ile sınırlı olduğundan, Kubernetes hizmetlerini kullanıma sunmak için giriş denetleyicilerinin kullanılmasını öneririz. Bu tasarım Ayrıca, Kubernetes hizmetlerinin sayısını (yük dengeleyici olan) 5 olarak sınırlar ve bu da birçok dağıtım için çok küçük olacaktır. Daha fazla bilgi edinmek için [aks altyapısı belgelerine](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips) gidin.

Bir Load Balancer veya giriş denetleyicisi aracılığıyla genel IP kullanan bir uygulamayı göstermek, uygulamanın artık Internet üzerinden erişilebilir olduğunu ortaya çıkarmaz. Azure Stack hub 'ının yalnızca yerel intranette görünen genel IP adresine sahip olması mümkündür. tüm genel IP 'Ler gerçekten Internet 'e açık değildir.

Önceki blok, uygulamaya giriş trafiğini kabul eder. Başarılı bir Kubernetes dağıtımı için göz önünde bulundurmanız gereken başka bir konu, giden/çıkış trafiğidir. Çıkış trafiği gerektiren birkaç kullanım durumu aşağıda verilmiştir:

- DockerHub veya Azure Container Registry depolanan kapsayıcı görüntülerini çekme
- Held grafiklerini alma
- Application Insights verileri (veya diğer izleme verilerini) yayma

Bazı kurumsal ortamlar, _saydam_ veya _saydam olmayan_ ara sunucu kullanımını gerektirebilir. Bu sunucular, kümeimizin çeşitli bileşenlerinde belirli bir yapılandırma gerektirir. AKS altyapısı belgeleri, ağ proxy 'lerine uyum hakkında çeşitli ayrıntılar içerir. Daha ayrıntılı bilgi için bkz. [aks altyapısı ve proxy sunucuları](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md)

Son olarak, çapraz küme trafiği Azure Stack hub örnekleri arasında akmalıdır. Örnek dağıtım, bireysel Azure Stack hub örneklerinde çalışan tek bir Kubernetes kümelerinden oluşur. İki veritabanı arasındaki çoğaltma trafiği gibi aralarındaki trafik "dış trafik" olur. Dış trafik iki Azure Stack hub örneğine Kubernetes bağlamak için bir siteden siteye VPN veya Azure Stack hub ortak IP adresleri üzerinden yönlendirilmelidir:

![şirketler arası ve içi küme trafiği](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Küme**  

Kubernetes kümesinin Internet üzerinden erişilebilir olması gerekmez. İlgili bölüm, bir kümeyi çalıştırmak için kullanılan Kubernetes API 'sidir, örneğin kullanarak `kubectl` . Kubernetes API uç noktası, kümeyi çalıştıran herkes tarafından erişilebilir olmalıdır veya üzerine uygulama ve hizmetler dağıtır. Bu konu, aşağıdaki [dağıtım (CI/CD) konuları](#deployment-cicd-considerations) bölümündeki DevOps-perspektifinden daha ayrıntılı bir şekilde ele alınmıştır.

Küme düzeyinde, çıkış trafiğiyle ilgili bazı konular da vardır:

- Düğüm güncelleştirmeleri (Ubuntu için)
- İzleme verileri (Azure LogAnalytics 'e gönderilir)
- Giden trafik gerektiren diğer aracılar (her bir dağıtıcı ortamına özel)

AKS altyapısını kullanarak Kubernetes kümenizi dağıtmadan önce, son ağ tasarımını planlayın. Ayrılmış bir sanal ağ oluşturmak yerine, bir kümeyi var olan bir ağa dağıtmak daha verimli olabilir. Örneğin, Azure Stack hub ortamınızda zaten yapılandırılmış olan bir siteden siteye VPN bağlantısından yararlanabilirsiniz.

**Altyapı**  

Altyapı, Azure Stack hub yönetim uç noktalarına erişme anlamına gelir. Uç noktalar kiracı ve yönetici portalları ve Azure Resource Manager yönetici ve kiracı uç noktaları içerir. Bu uç noktalar Azure Stack hub 'ı ve çekirdek hizmetlerini çalıştırmak için gereklidir.

## <a name="data-and-storage-considerations"></a>Veri ve depolama konuları

İki ayrı bir Kubernetes kümesinde, uygulamamız için iki örnek iki Azure Stack hub örneği üzerinden dağıtılacaktır. Bu tasarıma, aralarındaki verileri nasıl çoğaltacağınızı ve eşitleyeceğiniz hakkında düşünmeniz gerekir.

Azure sayesinde, depolamayı bulut içindeki birden çok bölgeye ve bölgeye çoğaltmak için yerleşik bir özelliktir. Şu anda Azure Stack hub ile iki farklı Azure Stack hub örneğine depolama çoğaltmak için yerel bir yol yoktur. Bunlar, bunları bir küme olarak yönetmeyecek şekilde, iki bağımsız bulut oluşturur. Azure Stack hub 'da çalışan uygulamaların dayanıklılık planlaması, bu bağımsızlık 'yi uygulama tasarımınızda ve dağıtımlarınızda göz önünde bulundurmaya zorlar.

Çoğu durumda, AKS üzerinde dağıtılan dayanıklı ve yüksek oranda kullanılabilir bir uygulama için depolama çoğaltması gerekmez. Ancak, uygulama tasarımınızda Azure Stack hub örneği başına bağımsız depolama alanı göz önünde bulundurmanız gerekir. Bu tasarım, çözümü Azure Stack hub 'a dağıtmaya yönelik bir sorun veya yol bloğudur, Microsoft Iş ortaklarının depolama ekleri sağlayan olası çözümleri vardır. Depolama ekleri, birden çok Azure Stack hub ve Azure arasında bir depolama çoğaltma çözümü sağlar. Daha fazla bilgi için bkz. [Iş ortağı çözümleri](#partner-solutions).

Mimarimizde bu katmanlar göz önünde bulundurulmıştı:

**Yapılandırma**

Yapılandırma Azure Stack hub, AKS altyapısının ve Kubernetes kümesinin yapılandırmasını içerir. Yapılandırma mümkün olduğunca otomatikleştirilebilir ve Azure DevOps veya GitHub gibi git tabanlı bir sürüm denetim sisteminde kod olarak altyapı olarak depolanır. Bu ayarlar birden fazla dağıtımda kolayca eşitlenemez. Bu nedenle, yapılandırmayı depolamadan ve DevOps işlem hattını kullanarak depolamayı ve uygulamayı öneririz.

**Uygulama**

Uygulama, git tabanlı bir depoda depolanmalıdır. Yeni bir dağıtım, uygulamada yapılan değişiklikler veya olağanüstü durum kurtarma her seferinde Azure Pipelines kullanılarak kolayca dağıtılabilir.

**Veriler**

Veriler çoğu uygulama tasarımlarında en önemli noktadır. Uygulama verileri, uygulamanın farklı örnekleri arasında eşitlenmiş olmalıdır. Ayrıca, bir kesinti olursa verilerin bir yedekleme ve olağanüstü durum kurtarma stratejisi olması gerekir.

Bu tasarıma ulaşmak, teknoloji seçimlerine göre farklılık gösterir. Bir veritabanını Azure Stack hub 'ında yüksek oranda kullanılabilir bir biçimde uygulamaya yönelik bazı çözüm örnekleri aşağıda verilmiştir:

- [Azure ve Azure Stack hub 'a SQL Server 2016 kullanılabilirlik grubu dağıtma](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [Yüksek oranda kullanılabilir bir MongoDB çözümünü Azure 'a ve Azure Stack hub 'a dağıtma](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

Birden çok konumdaki verilerle çalışırken dikkat edilmesi gereken noktalar, yüksek oranda kullanılabilir ve dayanıklı bir çözüm için daha karmaşık bir konudur. Aşağıdakileri dikkate alın:

- Azure Stack hub 'Lar arasındaki gecikme ve ağ bağlantısı.
- Hizmetler ve izinler için kimliklerin kullanılabilirliği. Her Azure Stack hub örneği bir dış dizinle tümleştirilir. Dağıtım sırasında, Azure Active Directory (Azure AD) ya da Active Directory Federasyon Hizmetleri (AD FS) (ADFS) kullanmayı tercih edersiniz. Bu nedenle, birden çok bağımsız Azure Stack hub örneğiyle etkileşime girebilen tek bir kimlik kullanmak mümkün olabilir.

## <a name="business-continuity-and-disaster-recovery"></a>İş sürekliliği ve olağanüstü durum kurtarma

İş sürekliliği ve olağanüstü durum kurtarma (BCDR), Azure Stack hub ve Azure 'daki önemli bir konudur. Temel fark, Azure Stack hub 'ında, işlecin tüm BCDR sürecini yönetmesi gerekir. Azure 'da, BCDR 'nin parçaları Microsoft tarafından otomatik olarak yönetilir.

BCDR önceki bölüm [verilerinde ve depolama](#data-and-storage-considerations)alanlarında bahsedilen alanların aynısını etkiler:

- Altyapı/yapılandırma
- Uygulama kullanılabilirliği
- Uygulama Verileri

Önceki bölümde belirtildiği gibi, bu bölgeler Azure Stack hub işlecinin sorumluluğundadır ve kuruluşlar arasında farklılık gösterebilir. BCDR 'yi kullanılabilir araçlara ve süreçlerinize göre planlayın.

**Altyapı ve yapılandırma**

Bu bölüm, fiziksel ve mantıksal altyapıyı ve Azure Stack hub 'ın yapılandırmasını içerir. Yönetici ve kiracı alanlarında eylemleri ele alır.

Azure Stack hub işleci (veya Yöneticisi), Azure Stack hub örneklerinin bakımında sorumludur. Ağ, depolama, kimlik ve bu makalenin kapsamı dışında kalan diğer konular gibi bileşenler de dahil olmak üzere. Azure Stack hub işlemlerinin özellikleri hakkında daha fazla bilgi edinmek için aşağıdaki kaynaklara bakın:

- [Infrastructure Backup hizmeti ile Azure Stack hub 'daki verileri kurtarma](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [Yönetici portalından Azure Stack Hub için yedeklemeyi etkinleştirme](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [Geri dönülemez veri kaybından kurtarma](/azure-stack/operator/azure-stack-backup-recover-data)
- [Infrastructure Backup Hizmeti en iyi uygulamaları](/azure-stack/operator/azure-stack-backup-best-practices)

Azure Stack hub, Kubernetes uygulamalarının dağıtılacağı platform ve dokudır. Kubernetes uygulamasının uygulama sahibi, çözüm için gereken uygulama altyapısını dağıtmak için erişim izni verilen Azure Stack hub 'ın bir kullanıcısı olacaktır. Bu durumda uygulama altyapısı, AKS altyapısı kullanılarak dağıtılan Kubernetes kümesi ve çevreleyen hizmetler anlamına gelir. Bu bileşenler, bir Azure Stack hub teklifiyle kısıtlanmış Azure Stack hub 'ına dağıtılır. Kubernetes uygulama sahibi tarafından kabul edilen teklifin tüm çözümü dağıtmak için yeterli kapasiteye sahip olduğundan emin olun (Azure Stack Hub kotaları olarak ifade edilir). Önceki bölümde önerildiği gibi, uygulama dağıtımı, Azure DevOps Azure Pipelines gibi altyapı kodu ve dağıtım işlem hatları kullanılarak otomatikleştirilmelidir.

Azure Stack hub teklifleri ve kotaları hakkında daha fazla bilgi için bkz. [Azure Stack Hub hizmetleri, planlar, teklifler ve aboneliklerde genel bakış](/azure-stack/operator/service-plan-offer-subscription-overview)

AKS altyapısı yapılandırmasını, çıkışları dahil güvenli bir şekilde kaydetmek ve depolamak önemlidir. Bu dosyalar, Kubernetes kümesine erişmek için kullanılan gizli bilgileri içerir, bu nedenle yönetici olmayan olarak gösterilmelidir.

**Uygulama kullanılabilirliği**

Uygulama, dağıtılan bir örneğin yedeklemelerine güvenmemelidir. Standart bir uygulama olarak, kod olarak altyapı düzenlerini izleyen uygulamayı tamamen yeniden dağıtın. Örneğin, Azure DevOps Azure Pipelines kullanarak yeniden dağıtın. BCDR yordamı, uygulamayı aynı veya başka bir Kubernetes kümesine yeniden dağıtmayı içermelidir.

**Uygulama verileri**

Uygulama verileri, veri kaybını en aza indirmek için önemli bir bölümüdür. Önceki bölümde, uygulamanın iki (veya daha fazla) örneği arasında verileri çoğaltmak ve senkronize etmek için teknikler açıklanmıştı. Verileri depolamak için kullanılan veritabanı altyapısına (MySQL, MongoDB, MSSQL veya diğerleri) bağlı olarak, aralarından seçim yapabileceğiniz farklı veritabanı kullanılabilirliği ve yedekleme teknikleri de bulunur.

Bütünlüğü sağlamak için önerilen yollar şunlardan birini kullanmaktır:
- Belirli bir veritabanı için yerel bir yedekleme çözümü.
- Sizin uygulamanız tarafından kullanılan veritabanı türünün yedeklemesini ve kurtarılmasını resmi olarak destekleyen bir yedekleme çözümü.

> [!IMPORTANT]
> Yedekleme verilerinizi, uygulama verilerinizin bulunduğu Azure Stack hub örneğinde depolamayın. Azure Stack hub örneğinin tamamen kesilmesi da yedeklemelerinizin güvenliğini tehlikeye atabilir.

## <a name="availability-considerations"></a>Kullanılabilirlik konusunda dikkat edilmesi gerekenler

AKS altyapısı aracılığıyla dağıtılan Azure Stack hub 'da Kubernetes yönetilen bir hizmet değildir. Bu, Azure hizmet olarak altyapı (IaaS) kullanılarak bir Kubernetes kümesinin otomatik olarak dağıtılması ve yapılandırması. Bu nedenle, temel alınan altyapıyla aynı kullanılabilirliği sağlar.

Azure Stack hub altyapısı hatalara karşı dayanıklıdır ve birden çok [hata ve güncelleştirme etki alanı](/azure-stack/user/azure-stack-vm-considerations#high-availability)arasında bileşenleri dağıtmak Için kullanılabilirlik kümeleri gibi yetenekler sağlar. Ancak, temel alınan teknoloji (Yük Devretme Kümelemesi), bir donanım arızası olması durumunda etkilenen bir fiziksel sunucudaki VM 'Ler için bazı kesinti süreleri doğurur.

Üretim Kubernetes kümenizin yanı sıra iki (veya daha fazla) kümeye iş yükünü dağıtmak iyi bir uygulamadır. Bu kümeler farklı konumlarda veya veri merkezlerinde barındırılmalıdır ve kullanıcıları küme yanıt süresine veya coğrafi konuma göre yönlendirmek için Azure Traffic Manager gibi teknolojileri kullanmalıdır.

![Trafik akışlarını denetlemek için Traffic Manager kullanma](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

Tek bir Kubernetes kümesine sahip müşteriler tipik olarak belirli bir uygulamanın hizmet IP 'sini veya DNS adına bağlanır. Birden çok küme dağıtımında, müşteriler her bir Kubernetes kümesinde Hizmetleri/girişi işaret eden bir Traffic Manager DNS adına bağlanmalıdır.

![Şirket içi kümeye yönlendirmek için Traffic Manager kullanma](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> Bu model, [Azure 'daki (yönetilen) AKS kümeleri için de en iyi uygulamadır](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment).

AKS altyapısı aracılığıyla dağıtılan Kubernetes kümesi, en az üç ana düğüm ve iki çalışan düğümünden oluşmalıdır.

## <a name="identity-and-security-considerations"></a>Kimlik ve güvenlik konuları

Kimlik ve güvenlik önemli konulardır. Özellikle çözüm bağımsız Azure Stack hub örneklerine yayıldığında. Kubernetes ve Azure (Azure Stack hub dahil), her ikisi de rol tabanlı erişim denetimi (RBAC) için farklı mekanizmalarla sahiptir:

- Azure RBAC, yeni Azure kaynakları oluşturma özelliği de dahil olmak üzere Azure 'daki (ve Azure Stack hub) kaynaklara erişimi denetler. İzinler kullanıcılara, gruplara veya hizmet sorumlularına atanabilir. (Hizmet sorumlusu, uygulamalar tarafından kullanılan bir güvenlik kimliğidir.)
- Kubernetes RBAC, Kubernetes API 'SI için izinleri denetler. Örneğin, Pod ve listeleme, bir kullanıcıya RBAC aracılığıyla yetkilendirilabilen (veya reddedilmeyen) eylemlerdir. Kullanıcılara Kubernetes izinleri atamak için roller ve rol bağlamaları oluşturursunuz.

**Azure Stack hub kimliği ve RBAC**

Azure Stack hub iki kimlik sağlayıcısı seçeneği sağlar. Kullandığınız sağlayıcı, ortama ve bağlı veya bağlantısı kesilmiş bir ortamda çalışıyor olmasına bağlıdır:

- Azure AD-yalnızca bağlı bir ortamda kullanılabilir.
- Geleneksel bir Active Directory ormanına ADFS-bağlı veya bağlantısı kesilmiş bir ortamda kullanılabilir.

Kimlik sağlayıcısı, kaynaklara erişim için kimlik doğrulaması ve yetkilendirme dahil olmak üzere kullanıcıları ve grupları yönetir. Abonelikler, kaynak grupları ve VM 'Ler ya da yük dengeleyiciler gibi bireysel kaynaklar gibi Azure Stack hub kaynaklarına erişim verilebilir. Tutarlı bir erişim modeline sahip olmak için, tüm Azure Stack hub 'Lar için aynı grupları (doğrudan veya iç içe) kullanmayı göz önünde bulundurmanız gerekir. Bir yapılandırma örneği aşağıda verilmiştir:

![Azure Stack hub ile iç içe AAD grupları](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

Örnek, belirli bir amaç için adanmış bir grup (AAD veya ADFS kullanarak) içerir. Örneğin, belirli bir Azure Stack hub örneğindeki Kubernetes küme altyapınızı içeren kaynak grubu için katkıda bulunan izinleri sağlamak üzere (burada "Seattle K8s Cluster katılımcısı"). Bu gruplar daha sonra her bir Azure Stack hub 'ının "alt grupları" içeren genel bir grubun içine yuvalanmıştır.

Örnek kullanıcımız artık tüm Kubernetes altyapı kaynakları kümesini içeren her iki kaynak grubu için "katkıda bulunan" izinlere sahip olacaktır. Örnekler aynı kimlik sağlayıcısını paylaştığından, Kullanıcı hem Azure Stack hub örneklerine ait kaynaklara erişebilir.

> [!IMPORTANT]
> Bu izinler yalnızca Azure Stack merkezini ve en üstünde dağıtılan kaynakları etkiler. Bu erişim düzeyine sahip bir Kullanıcı çok sayıda zarar verebilir, ancak Kubernetes IaaS VM 'lerine veya Kubernetes API 'sine ek erişim olmadan Kubernetes API 'sine erişemez.

**Kubernetes kimliği ve RBAC**

Bir Kubernetes kümesi, varsayılan olarak Azure Stack hub 'ı ile aynı kimlik sağlayıcısını kullanmaz. Kubernetes kümesi, ana ve çalışan düğümlerini barındıran VM 'Ler, kümenin dağıtımı sırasında belirtilen SSH anahtarını kullanır. Bu SSH anahtarı SSH kullanarak bu düğümlere bağlanmak için gereklidir.

Kubernetes API 'SI (örneğin, kullanılarak erişilen), `kubectl` varsayılan "Küme Yöneticisi" hizmet hesabı da dahil olmak üzere hizmet hesaplarıyla de korunur. Bu hizmet hesabı için kimlik bilgileri başlangıçta `.kube/config` Kubernetes ana düğümlerindeki dosyada depolanır.

**Gizli dizi yönetimi ve uygulama kimlik bilgileri**

Bağlantı dizeleri veya veritabanı kimlik bilgileri gibi gizli dizileri depolamak için aşağıdakiler dahil olmak üzere birkaç seçenek vardır:

- Azure Key Vault
- Kubernetes Gizli Dizileri
- HashiCorp kasası gibi üçüncü taraf çözümler (Kubernetes üzerinde çalışıyor)

Gizli dizileri veya kimlik bilgilerini yapılandırma dosyalarınızda, uygulama kodunuzda veya betiklerinizde bir düz metin içinde depolamayın. Ve bunları bir sürüm denetim sisteminde saklamayın. Bunun yerine, Dağıtım Otomasyonu, gerektiğinde gizli dizileri almalıdır.

## <a name="patch-and-update"></a>Düzeltme Eki ve güncelleştirme

Azure Kubernetes hizmetindeki **Düzeltme Eki ve güncelleştirme (PNU)** işlemi kısmen otomatikleştirilmiştir. Kubernetes sürüm yükseltmeleri el ile tetiklenir ve güvenlik güncelleştirmeleri otomatik olarak uygulanır. Bu güncelleştirmeler, işletim sistemi güvenlik düzeltmeleri veya çekirdek güncelleştirmeleri içerebilir. AKS, güncelleştirme işlemini tamamlamaya yönelik olarak bu Linux düğümlerini otomatik olarak yeniden başlatır. 

Azure Stack Hub üzerinde AKS altyapısı kullanılarak dağıtılan bir Kubernetes kümesi için PNU işlemi yönetilmez ve küme işlecinin sorumluluğundadır. 

AKS altyapısı, en önemli iki görevle yardımcı olur:

- [Daha yeni bir Kubernetes ve temel işletim sistemi görüntüsü sürümüne yükselt](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [Yalnızca temel işletim sistemi görüntüsünü yükselt](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

Yeni temel işletim sistemi görüntüleri, en son işletim sistemi güvenlik düzeltmelerini ve çekirdek güncelleştirmelerini içerir. 

[Katılımsız yükseltme](https://wiki.debian.org/UnattendedUpgrades) mekanizması, Azure Stack hub marketi 'nde yeni bir temel işletim sistemi görüntüsü sürümü kullanılabilir olmadan önce yayımlanan güvenlik güncelleştirmelerini otomatik olarak kurar. Katılımsız yükseltme varsayılan olarak etkindir ve güvenlik güncelleştirmeleri otomatik olarak yüklenir, ancak Kubernetes küme düğümlerini yeniden yüklemez. Düğümlerin yeniden başlatılması, açık kaynak [ **K** ubernetes **yeniden** önyüklemesi **D** Aemon (kured))](/azure/aks/node-updates-kured)kullanılarak otomatikleştirilebilir. Kured, yeniden başlatma gerektiren Linux düğümleri için, çalışan Pod ve düğüm yeniden başlatma işlemini otomatik olarak işler.

## <a name="deployment-cicd-considerations"></a>Dağıtım (CI/CD) konuları

Azure ve Azure Stack hub, aynı Azure Resource Manager REST API 'Lerini kullanıma sunar. Bu API 'Ler, diğer tüm Azure bulutlarının (Azure, Azure Çin 21Vianet, Azure Kamu) olduğu gibi karşılanır. Bulutlar arasındaki API sürümlerinde farklılıklar olabilir ve Azure Stack hub yalnızca bir hizmet alt kümesini sağlar. Yönetim uç noktası URI 'SI her bulut için ve Azure Stack hub 'ın her örneği için de farklıdır.

Azure Resource Manager REST API 'Leri, Azure ve Azure Stack hub ile etkileşimde bulunmak için tutarlı bir yol sağlar. Aynı araç kümesi, diğer Azure bulutuyla birlikte kullanılacak şekilde burada kullanılabilir. Hizmetleri Azure Stack hub 'a dağıtmak ve düzenlemek için Azure DevOps, Jenkins veya PowerShell gibi araçları kullanabilirsiniz.

**Dikkat edilmesi gerekenler**

Azure Stack hub dağıtımları, Internet erişilebilirliği sorularından biri olduğunda önemli farklardan biridir. Internet erişilebilirliği, CI/CD işleriniz için Microsoft tarafından barındırılan veya şirket içinde barındırılan bir yapı aracısının seçip seçmeyeceğinizi belirler.

Şirket içinde barındırılan bir aracı Azure Stack hub 'ında (IaaS VM olarak) veya Azure Stack hub 'ına erişebilen bir ağ alt ağında çalışabilir. Farklar hakkında daha fazla bilgi edinmek için [Azure Pipelines aracıları](/azure/devops/pipelines/agents/agents) ' na gidin.

Aşağıdaki görüntü, şirket içinde barındırılan veya Microsoft tarafından barındırılan bir yapı aracısına ihtiyacınız olduğuna karar vermenize yardımcı olur:

![Şirket içinde barındırılan derleme aracıları Evet veya Hayır](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Azure Stack hub Management uç noktaları Internet üzerinden erişilebilir mi?
  - Evet: Azure Stack hub 'a bağlanmak için Microsoft tarafından barındırılan aracılarla Azure Pipelines kullanabiliriz.
  - Hayır: Azure Stack hub 'ın yönetim uç noktalarına bağlanabilecek, şirket içinde barındırılan aracılara ihtiyacımız var.
- Kubernetes kümeimiz Internet üzerinden erişilebilir mi?
  - Evet: doğrudan Kubernetes API uç noktası ile etkileşim kurmak için Microsoft tarafından barındırılan aracılarla Azure Pipelines kullanabiliriz.
  - Hayır: Kubernetes kümesi API uç noktasına bağlanabilecek, kendi kendine barındırılan aracılara ihtiyacımız var.

Azure Stack hub Management uç noktaları ve Kubernetes API 'sinin Internet üzerinden erişilebileceği senaryolarda, dağıtım Microsoft tarafından barındırılan bir aracı kullanabilir. Bu dağıtım, aşağıdaki gibi bir uygulama mimarisine neden olur:

[![Genel mimariye genel bakış](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

Azure Resource Manager uç noktaları, Kubernetes API 'SI veya her ikisine doğrudan Internet üzerinden erişilemezse, ardışık düzen adımlarını çalıştırmak için şirket içinde barındırılan bir yapı aracısından yararlanabilirsiniz. Bu tasarımın daha az bağlantısı olması gerekir ve yalnızca uç noktalarına ve Kubernetes API 'sine Azure Resource Manager yönelik şirket içi ağ bağlantısı ile dağıtılabilir:

[![Şirket içi mimariye genel bakış](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **Bağlantısı kesilen senaryolar hakkında ne var?** Azure Stack hub veya Kubernetes ya da her ikisinin de internet 'e yönelik yönetim uç noktalarına sahip olmadığı senaryolarda, dağıtımlarınız için Azure DevOps kullanılması hala mümkündür. Şirket içinde barındırılan bir aracı Havuzu (Şirket içinde veya Azure Stack hub 'ında çalışan bir DevOps Aracısı) ya da şirket içinde tamamen barındırılan bir Azure DevOps Server kullanabilirsiniz. Şirket içinde barındırılan aracının yalnızca giden HTTPS (TCP/443) Internet bağlantısı olması gerekir.

Bu model her bir Azure Stack hub örneğinde bir Kubernetes kümesi (AKS altyapısı ile dağıtılan ve düzenlenmiş) kullanabilir. Bu, bir ön uç, orta katman, arka uç Hizmetleri (örneğin MongoDB) ve NGINX tabanlı giriş denetleyicisi içeren bir uygulama içerir. K8s kümesinde barındırılan bir veritabanı kullanmak yerine "dış veri depoları" ndan yararlanabilirsiniz. Veritabanı seçenekleri MySQL, SQL Server ya da Azure Stack hub veya IaaS dışında barındırılan herhangi bir tür veritabanını içerir. Bu gibi yapılandırmalarda burada kapsam yoktur.

## <a name="partner-solutions"></a>İş ortağı çözümleri

Azure Stack hub 'ın yeteneklerini genişletebilen Microsoft Iş ortağı çözümleri vardır. Bu çözümler, Kubernetes kümelerinde çalışan uygulamaların dağıtımlarında yararlı bulunur.  

## <a name="storage-and-data-solutions"></a>Depolama ve veri çözümleri

[Veri ve depolama konusunda](#data-and-storage-considerations)açıklandığı gibi, Azure Stack hub 'ın Şu anda birden çok örnek arasında depolama çoğaltması için yerel bir çözümü yoktur. Azure 'dan farklı olarak, birden çok bölgede depolama çoğaltma özelliği mevcut değildir. Azure Stack hub 'ında her örnek kendi ayrı bulutuna aittir. Ancak, Microsoft Iş ortaklarından Azure Stack hub 'larda ve Azure arasında depolama çoğaltması sağlayan çözümler mevcuttur. 

**KORLIK**

[Korkliği](https://www.scality.com/) , 2009 'den beri desteklenen dijital işletmeler içeren Web ölçekli depolama alanı sunar. Yazılım tanımlı depolamamız olan korun HALKASı, emtia x86 sunucularını herhangi bir veri türü için sınırsız bir depolama havuzuna (dosya ve nesne), petabda ölçeğe dönüştürür.

**CLOUYEN**

[Clou,](https://www.cloudian.com/) çok büyük veri kümelerini tek ve kolayca yönetilen bir ortama birleştiren sınırsız sayıda ölçeklenebilir depolamaya sahip kurumsal depolamayı basitleştirir.

## <a name="next-steps"></a>Sonraki adımlar

Bu makalede tanıtılan kavramlar hakkında daha fazla bilgi edinmek için:

- Azure Stack hub 'da [platformlar arası ölçekleme](pattern-cross-cloud-scale.md) ve [coğrafi olarak dağıtılmış uygulama desenleri](pattern-geo-distributed.md) .
- [Azure Kubernetes Service (AKS) üzerinde mikro hizmetler mimarisi](/azure/architecture/reference-architectures/microservices/aks).

Çözüm örneğini test etmeye hazır olduğunuzda, [yüksek kullanılabilirliğe sahip Kubernetes küme dağıtım kılavuzuna](solution-deployment-guide-highly-available-kubernetes.md)devam edin. Dağıtım Kılavuzu, bileşenlerinin dağıtılması ve test edilmesi için adım adım yönergeler sağlar.