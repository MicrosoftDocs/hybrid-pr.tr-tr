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
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="5e50a-103">Coğrafi olarak dağıtılmış uygulama kalıbı</span><span class="sxs-lookup"><span data-stu-id="5e50a-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="5e50a-104">Birden çok bölgede uygulama uç noktaları sağlamayı ve Kullanıcı trafiğini konum ve uyumluluk ihtiyaçlarına göre yönlendirme hakkında bilgi edinin.</span><span class="sxs-lookup"><span data-stu-id="5e50a-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="5e50a-105">Bağlam ve sorun</span><span class="sxs-lookup"><span data-stu-id="5e50a-105">Context and problem</span></span>

<span data-ttu-id="5e50a-106">Geniş erişime sahip kuruluşlar, her Kullanıcı, konum ve bir cihaz için gerekli güvenlik, uyumluluk ve performans düzeylerini sağlarken verilere erişimi güvenle ve doğru bir şekilde dağıtabilir ve etkinleştirir.</span><span class="sxs-lookup"><span data-stu-id="5e50a-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="5e50a-107">Çözüm</span><span class="sxs-lookup"><span data-stu-id="5e50a-107">Solution</span></span>

<span data-ttu-id="5e50a-108">Azure Stack hub coğrafi trafik yönlendirme deseninin veya coğrafi olarak Dağıtılmış uygulamaların, trafiğin çeşitli ölçümlere bağlı olarak belirli uç noktalara yönlendirilmelerini sağlar.</span><span class="sxs-lookup"><span data-stu-id="5e50a-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="5e50a-109">Coğrafi tabanlı Yönlendirme ve uç nokta yapılandırması ile Traffic Manager oluşturmak, bölgesel gereksinimlere, şirkete ve uluslararası yönetmelere ve veri ihtiyaçlarına göre trafiği uç noktalara yönlendirir.</span><span class="sxs-lookup"><span data-stu-id="5e50a-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![Coğrafi olarak dağıtılmış desenler](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="5e50a-111">Bileşenler</span><span class="sxs-lookup"><span data-stu-id="5e50a-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="5e50a-112">Bulutun dışında</span><span class="sxs-lookup"><span data-stu-id="5e50a-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="5e50a-113">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="5e50a-113">Traffic Manager</span></span>

<span data-ttu-id="5e50a-114">Diyagramda Traffic Manager, genel bulutun dışında bulunur, ancak hem yerel veri merkezinde hem de genel bulutta trafiği koordine edebilmelidir.</span><span class="sxs-lookup"><span data-stu-id="5e50a-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="5e50a-115">Dengeleyici trafiği coğrafi konumlara yönlendirir.</span><span class="sxs-lookup"><span data-stu-id="5e50a-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="5e50a-116">Etki Alanı Adı Sistemi (DNS)</span><span class="sxs-lookup"><span data-stu-id="5e50a-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="5e50a-117">Etki alanı adı sistemi veya DNS, IP adresine bir Web sitesi veya hizmet adı çevirmekten (veya çözümlemeden) sorumludur.</span><span class="sxs-lookup"><span data-stu-id="5e50a-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="5e50a-118">Genel bulut</span><span class="sxs-lookup"><span data-stu-id="5e50a-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="5e50a-119">Bulut uç noktası</span><span class="sxs-lookup"><span data-stu-id="5e50a-119">Cloud Endpoint</span></span>

<span data-ttu-id="5e50a-120">Genel IP adresleri, Trafik Yöneticisi üzerinden gelen trafiği genel bulut uygulama kaynakları uç noktasına yönlendirmek için kullanılır.</span><span class="sxs-lookup"><span data-stu-id="5e50a-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="5e50a-121">Yerel bulutlar</span><span class="sxs-lookup"><span data-stu-id="5e50a-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="5e50a-122">Yerel uç nokta</span><span class="sxs-lookup"><span data-stu-id="5e50a-122">Local endpoint</span></span>

<span data-ttu-id="5e50a-123">Genel IP adresleri, Trafik Yöneticisi üzerinden gelen trafiği genel bulut uygulama kaynakları uç noktasına yönlendirmek için kullanılır.</span><span class="sxs-lookup"><span data-stu-id="5e50a-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="5e50a-124">Sorunlar ve dikkat edilmesi gerekenler</span><span class="sxs-lookup"><span data-stu-id="5e50a-124">Issues and considerations</span></span>

<span data-ttu-id="5e50a-125">Bu düzenin nasıl uygulanacağına karar verirken aşağıdaki noktaları göz önünde bulundurun:</span><span class="sxs-lookup"><span data-stu-id="5e50a-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="5e50a-126">Ölçeklenebilirlik</span><span class="sxs-lookup"><span data-stu-id="5e50a-126">Scalability</span></span>

<span data-ttu-id="5e50a-127">Bu model, trafiğin artışlarını karşılamak için ölçeklendirilmesi yerine coğrafi trafik yönlendirmeyi işler.</span><span class="sxs-lookup"><span data-stu-id="5e50a-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="5e50a-128">Ancak, bu stili diğer Azure ve şirket içi çözümlerle birleştirebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="5e50a-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="5e50a-129">Örneğin, bu model, platformlar arası ölçeklendirme düzeniyle birlikte kullanılabilir.</span><span class="sxs-lookup"><span data-stu-id="5e50a-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="5e50a-130">Kullanılabilirlik</span><span class="sxs-lookup"><span data-stu-id="5e50a-130">Availability</span></span>

<span data-ttu-id="5e50a-131">Yerel olarak dağıtılan uygulamaların şirket içi donanım yapılandırması ve yazılım dağıtımı aracılığıyla yüksek kullanılabilirlik için yapılandırıldığından emin olun.</span><span class="sxs-lookup"><span data-stu-id="5e50a-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="5e50a-132">Yönetilebilirlik</span><span class="sxs-lookup"><span data-stu-id="5e50a-132">Manageability</span></span>

<span data-ttu-id="5e50a-133">Bu model, ortamlar arasında sorunsuz yönetim ve tanıdık arabirim sağlar.</span><span class="sxs-lookup"><span data-stu-id="5e50a-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="5e50a-134">Bu düzenin kullanılacağı durumlar</span><span class="sxs-lookup"><span data-stu-id="5e50a-134">When to use this pattern</span></span>

- <span data-ttu-id="5e50a-135">Kuruluşumun özel bölgesel güvenlik ve dağıtım ilkeleri gerektiren Uluslararası dalları vardır.</span><span class="sxs-lookup"><span data-stu-id="5e50a-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="5e50a-136">Kuruluş ofislerimin her biri, her yerel yönetmeliğe ve saat dilimine göre raporlama etkinliği gerektiren çalışan, iş ve tesis verilerini çeker.</span><span class="sxs-lookup"><span data-stu-id="5e50a-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="5e50a-137">Yüksek ölçekli gereksinimler, tek bir bölgede ve bölgeler arasında birden fazla uygulama dağıtımı, çok fazla yük gereksinimini işleyecek şekilde, uygulamalar tarafından yatay olarak ölçeklendirilirken karşılandı.</span><span class="sxs-lookup"><span data-stu-id="5e50a-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="5e50a-138">Uygulamalar, tek bölgede kesintiler halinde bile yüksek oranda kullanılabilir ve istemci isteklerine yanıt vermelidir.</span><span class="sxs-lookup"><span data-stu-id="5e50a-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="5e50a-139">Sonraki adımlar</span><span class="sxs-lookup"><span data-stu-id="5e50a-139">Next steps</span></span>

<span data-ttu-id="5e50a-140">Bu makalede tanıtılan konular hakkında daha fazla bilgi edinmek için:</span><span class="sxs-lookup"><span data-stu-id="5e50a-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="5e50a-141">Bu DNS tabanlı trafik yük dengeleyicinin nasıl çalıştığı hakkında daha fazla bilgi edinmek için bkz. [Azure Traffic Manager genel bakış](/azure/traffic-manager/traffic-manager-overview) .</span><span class="sxs-lookup"><span data-stu-id="5e50a-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="5e50a-142">En iyi uygulamalar hakkında daha fazla bilgi edinmek ve ek sorulara yanıt almak için bkz. [karma uygulama tasarımı konuları](overview-app-design-considerations.md) .</span><span class="sxs-lookup"><span data-stu-id="5e50a-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="5e50a-143">Ürünlerin ve çözümlerin tamamı hakkında daha fazla bilgi edinmek için [ürün ve çözümlerin Azure Stack ailesine](/azure-stack) bakın.</span><span class="sxs-lookup"><span data-stu-id="5e50a-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="5e50a-144">Çözüm örneğini test etmeye hazırsanız, [coğrafi olarak dağıtılmış uygulama çözümü dağıtım kılavuzu](solution-deployment-guide-geo-distributed.md)ile devam edin.</span><span class="sxs-lookup"><span data-stu-id="5e50a-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="5e50a-145">Dağıtım Kılavuzu, bileşenlerinin dağıtılması ve test edilmesi için adım adım yönergeler sağlar.</span><span class="sxs-lookup"><span data-stu-id="5e50a-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="5e50a-146">Coğrafi olarak dağıtılmış uygulama modelini kullanarak çeşitli ölçümlere göre trafiği belirli uç noktalara yönlendirirsiniz.</span><span class="sxs-lookup"><span data-stu-id="5e50a-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="5e50a-147">Coğrafi tabanlı Yönlendirme ve uç nokta yapılandırmasıyla Traffic Manager profili oluşturma, bilgilerin bölgesel gereksinimlere, şirkete ve uluslararası yönetmelere ve veri gereksinimlerinize göre uç noktalara yönlendirilmesini sağlar.</span><span class="sxs-lookup"><span data-stu-id="5e50a-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
