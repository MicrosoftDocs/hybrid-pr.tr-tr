---
title: Azure Stack hub 'da platformlar arası ölçeklendirme kalıbı
description: Azure 'da ve Azure Stack hub 'da ölçeklenebilir bir platformlar arası uygulama oluşturmayı öğrenin.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911965"
---
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="3b4f5-103">Platformlar arası ölçeklendirme kalıbı</span><span class="sxs-lookup"><span data-stu-id="3b4f5-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="3b4f5-104">Yük artışına uyum sağlamak için mevcut bir uygulamaya kaynakları otomatik olarak ekleyin.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="3b4f5-105">Bağlam ve sorun</span><span class="sxs-lookup"><span data-stu-id="3b4f5-105">Context and problem</span></span>

<span data-ttu-id="3b4f5-106">Uygulamanız, isteğe bağlı olarak beklenmeyen artışları karşılamak için kapasiteyi artırabilir.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="3b4f5-107">Bu ölçeklenebilirlik olmaması, kullanıcıların yoğun kullanım süreleri sırasında uygulamaya ulaşmasına neden olur.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="3b4f5-108">Uygulama, sabit sayıda kullanıcıya hizmet verebilir.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="3b4f5-109">Küresel kuruluşlar, güvenli, güvenilir ve kullanılabilir bulut tabanlı uygulamalar gerektirir.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="3b4f5-110">Toplantı, isteğe bağlı olarak artar ve bu talebi desteklemek için doğru altyapıyı kullanmak önemlidir.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="3b4f5-111">İşletmeler, iş verileri güvenliği, depolama ve gerçek zamanlı kullanılabilirliğiyle maliyetleri ve bakımı dengelemeye uğraşır.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="3b4f5-112">Uygulamanızı genel bulutta çalıştırameyebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="3b4f5-113">Bununla birlikte, iş için şirket içi ortamlarında gereken kapasiteyi korumak, uygulama için talepte ani artışları karşılamak amacıyla ekonomik bir şekilde uygulanabilir olmayabilir.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="3b4f5-114">Bu düzende, şirket içi çözümünüzle ortak bulutun elakliğini kullanabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="3b4f5-115">Çözüm</span><span class="sxs-lookup"><span data-stu-id="3b4f5-115">Solution</span></span>

<span data-ttu-id="3b4f5-116">Platformlar arası ölçeklendirme stili, genel bulut kaynaklarıyla yerel bir bulutta bulunan bir uygulamayı genişletir.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="3b4f5-117">Bu, isteğe bağlı olarak bir artış veya azalmayla tetiklenir ve sırasıyla bulutta kaynak ekler veya kaldırır.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="3b4f5-118">Bu kaynaklar yedeklilik, hızlı kullanılabilirlik ve coğrafi uyumlu yönlendirme sağlar.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![Platformlar arası ölçeklendirme kalıbı](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="3b4f5-120">Bu model yalnızca uygulamanızın durum bilgisiz bileşenleri için geçerlidir.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="3b4f5-121">Bileşenler</span><span class="sxs-lookup"><span data-stu-id="3b4f5-121">Components</span></span>

<span data-ttu-id="3b4f5-122">Platformlar arası ölçeklendirme stili aşağıdaki bileşenlerden oluşur.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="3b4f5-123">Bulutun dışında</span><span class="sxs-lookup"><span data-stu-id="3b4f5-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="3b4f5-124">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="3b4f5-124">Traffic Manager</span></span>

<span data-ttu-id="3b4f5-125">Diyagramda bu, genel bulut grubunun dışında bulunur, ancak hem yerel veri merkezinde hem de genel bulutta trafiği koordine edebilmelidir.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="3b4f5-126">Dengeleyici, uç noktaları izleyerek ve gerektiğinde yük devretme yeniden dağıtımı sağlayarak uygulama için yüksek kullanılabilirlik sağlar.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="3b4f5-127">Etki Alanı Adı Sistemi (DNS)</span><span class="sxs-lookup"><span data-stu-id="3b4f5-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="3b4f5-128">Etki alanı adı sistemi veya DNS, IP adresine bir Web sitesi veya hizmet adı çevirmekten (veya çözümlemeden) sorumludur.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="3b4f5-129">Bulut</span><span class="sxs-lookup"><span data-stu-id="3b4f5-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="3b4f5-130">Barındırılan derleme sunucusu</span><span class="sxs-lookup"><span data-stu-id="3b4f5-130">Hosted build server</span></span>

<span data-ttu-id="3b4f5-131">Yapı ardışık yapınızı barındırmak için bir ortam.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="3b4f5-132">Uygulama kaynakları</span><span class="sxs-lookup"><span data-stu-id="3b4f5-132">App resources</span></span>

<span data-ttu-id="3b4f5-133">Sanal Makine Ölçek Kümeleri ve kapsayıcılar gibi uygulama kaynaklarının ölçeği ölçeklendirilmesi ve ölçeğini genişletmek gerekir.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="3b4f5-134">Özel etki alanı adı</span><span class="sxs-lookup"><span data-stu-id="3b4f5-134">Custom domain name</span></span>

<span data-ttu-id="3b4f5-135">Yönlendirme istekleri için özel bir etki alanı adı kullanın glob.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="3b4f5-136">Genel IP adresleri</span><span class="sxs-lookup"><span data-stu-id="3b4f5-136">Public IP addresses</span></span>

<span data-ttu-id="3b4f5-137">Genel IP adresleri, Trafik Yöneticisi üzerinden gelen trafiği genel bulut uygulama kaynakları uç noktasına yönlendirmek için kullanılır.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="3b4f5-138">Yerel bulut</span><span class="sxs-lookup"><span data-stu-id="3b4f5-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="3b4f5-139">Barındırılan derleme sunucusu</span><span class="sxs-lookup"><span data-stu-id="3b4f5-139">Hosted build server</span></span>

<span data-ttu-id="3b4f5-140">Yapı ardışık yapınızı barındırmak için bir ortam.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="3b4f5-141">Uygulama kaynakları</span><span class="sxs-lookup"><span data-stu-id="3b4f5-141">App resources</span></span>

<span data-ttu-id="3b4f5-142">Uygulama kaynakları, sanal makine ölçek kümeleri ve kapsayıcılar gibi ölçeklendirme ve genişleme özelliği gerektirir.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="3b4f5-143">Özel etki alanı adı</span><span class="sxs-lookup"><span data-stu-id="3b4f5-143">Custom domain name</span></span>

<span data-ttu-id="3b4f5-144">Yönlendirme istekleri için özel bir etki alanı adı kullanın glob.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="3b4f5-145">Genel IP adresleri</span><span class="sxs-lookup"><span data-stu-id="3b4f5-145">Public IP addresses</span></span>

<span data-ttu-id="3b4f5-146">Genel IP adresleri, Trafik Yöneticisi üzerinden gelen trafiği genel bulut uygulama kaynakları uç noktasına yönlendirmek için kullanılır.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="3b4f5-147">Sorunlar ve dikkat edilmesi gerekenler</span><span class="sxs-lookup"><span data-stu-id="3b4f5-147">Issues and considerations</span></span>

<span data-ttu-id="3b4f5-148">Bu düzenin nasıl uygulanacağına karar verirken aşağıdaki noktaları göz önünde bulundurun:</span><span class="sxs-lookup"><span data-stu-id="3b4f5-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="3b4f5-149">Ölçeklenebilirlik</span><span class="sxs-lookup"><span data-stu-id="3b4f5-149">Scalability</span></span>

<span data-ttu-id="3b4f5-150">Platformlar arası ölçeklendirmenin anahtar bileşeni, isteğe bağlı ölçeklendirmeyi sunma olanağıdır.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="3b4f5-151">Ölçeklendirme, ortak ve yerel bulut altyapısı arasında gerçekleşmelidir ve isteğe göre tutarlı ve güvenilir bir hizmet sağlar.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="3b4f5-152">Kullanılabilirlik</span><span class="sxs-lookup"><span data-stu-id="3b4f5-152">Availability</span></span>

<span data-ttu-id="3b4f5-153">Yerel olarak dağıtılan uygulamaların şirket içi donanım yapılandırması ve yazılım dağıtımı aracılığıyla yüksek kullanılabilirlik için yapılandırıldığından emin olun.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="3b4f5-154">Yönetilebilirlik</span><span class="sxs-lookup"><span data-stu-id="3b4f5-154">Manageability</span></span>

<span data-ttu-id="3b4f5-155">Platformlar arası desenler, ortamlar arasında sorunsuz yönetim ve tanıdık arabirim sağlar.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="3b4f5-156">Bu düzenin kullanılacağı durumlar</span><span class="sxs-lookup"><span data-stu-id="3b4f5-156">When to use this pattern</span></span>

<span data-ttu-id="3b4f5-157">Bu düzeni kullanarak:</span><span class="sxs-lookup"><span data-stu-id="3b4f5-157">Use this pattern:</span></span>

- <span data-ttu-id="3b4f5-158">Uygulama kapasitenizi, istek üzerine beklenmeyen talepler veya düzenli talepler ile artırmanız gerektiğinde.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="3b4f5-159">Yalnızca tepe noktaları sırasında kullanılacak kaynaklara yatırım yapmak istemezsiniz.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="3b4f5-160">Kullandığınız kadar ödeyin.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-160">Pay for what you use.</span></span>

<span data-ttu-id="3b4f5-161">Bu model şu durumlarda önerilmez:</span><span class="sxs-lookup"><span data-stu-id="3b4f5-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="3b4f5-162">Çözümünüz, kullanıcıların internet üzerinden bağlanmasını gerektirir.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="3b4f5-163">İşletmenizde, kaynak bağlantının yerinde bir çağrıdan gelmesini gerektiren yerel yönetmelikler vardır.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="3b4f5-164">Ağınız, ölçeklendirmenin performansını kısıtlayan düzenli performans sorunları yaşıyor.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="3b4f5-165">Ortamınızın internet bağlantısı kesildi ve genel buluta ulaşılamıyor.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="3b4f5-166">Sonraki adımlar</span><span class="sxs-lookup"><span data-stu-id="3b4f5-166">Next steps</span></span>

<span data-ttu-id="3b4f5-167">Bu makalede tanıtılan konular hakkında daha fazla bilgi edinmek için:</span><span class="sxs-lookup"><span data-stu-id="3b4f5-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="3b4f5-168">Bu DNS tabanlı trafik yük dengeleyicinin nasıl çalıştığı hakkında daha fazla bilgi edinmek için bkz. [Azure Traffic Manager genel bakış](/azure/traffic-manager/traffic-manager-overview) .</span><span class="sxs-lookup"><span data-stu-id="3b4f5-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="3b4f5-169">En iyi uygulamalar hakkında daha fazla bilgi edinmek ve ek sorulara yanıt almak için bkz. [karma uygulama tasarımı konuları](overview-app-design-considerations.md) .</span><span class="sxs-lookup"><span data-stu-id="3b4f5-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="3b4f5-170">Ürünlerin ve çözümlerin tamamı hakkında daha fazla bilgi edinmek için [ürün ve çözümlerin Azure Stack ailesine](/azure-stack) bakın.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="3b4f5-171">Çözüm örneğini test etmeye hazırsanız, [platformlar arası ölçeklendirme çözümü dağıtım kılavuzu](solution-deployment-guide-cross-cloud-scaling.md)ile devam edin.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="3b4f5-172">Dağıtım Kılavuzu, bileşenlerinin dağıtılması ve test edilmesi için adım adım yönergeler sağlar.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="3b4f5-173">Azure Stack merkezi barındırılan bir Web uygulamasından Azure 'da barındırılan bir Web uygulamasına geçiş için el ile tetiklenen bir işlem sağlamak üzere bir çoklu bulut çözümü oluşturmayı öğreneceksiniz.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="3b4f5-174">Ayrıca, yük altında esnek ve ölçeklenebilir bulut yardımcı programını sağlayarak Traffic Manager aracılığıyla otomatik ölçeklendirmeyi nasıl kullanacağınızı öğreneceksiniz.</span><span class="sxs-lookup"><span data-stu-id="3b4f5-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
