---
title: Azure ve Azure Stack hub kullanarak coğrafi olarak dağıtılmış bir uygulamayla doğrudan trafik
description: Azure ve Azure Stack hub kullanarak coğrafi olarak dağıtılmış uygulama çözümü ile trafiği belirli uç noktalara nasıl yönlendireceğinizi öğrenin.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 27d07070becfa902a715b451baae7c81c7e4b46f
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886841"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="85c0f-103">Azure ve Azure Stack hub kullanarak coğrafi olarak dağıtılmış bir uygulamayla doğrudan trafik</span><span class="sxs-lookup"><span data-stu-id="85c0f-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="85c0f-104">Coğrafi olarak dağıtılmış uygulamalar modelini kullanarak çeşitli ölçümleri temel alarak trafiğin belirli uç noktalara nasıl yönlendirileceğini öğrenin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="85c0f-105">Coğrafi tabanlı Yönlendirme ve uç nokta yapılandırmasıyla Traffic Manager profili oluşturma, bilgilerin bölgesel gereksinimlere, şirkete ve uluslararası yönetmelere ve veri gereksinimlerinize göre uç noktalara yönlendirilmesini sağlar.</span><span class="sxs-lookup"><span data-stu-id="85c0f-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="85c0f-106">Bu çözümde, aşağıdakileri yapmak için bir örnek ortam oluşturacaksınız:</span><span class="sxs-lookup"><span data-stu-id="85c0f-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="85c0f-107">Coğrafi olarak dağıtılmış bir uygulama oluşturun.</span><span class="sxs-lookup"><span data-stu-id="85c0f-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="85c0f-108">Uygulamanızı hedeflemek için Traffic Manager kullanın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="85c0f-109">Coğrafi olarak dağıtılmış uygulamalar modelini kullanma</span><span class="sxs-lookup"><span data-stu-id="85c0f-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="85c0f-110">Coğrafi olarak dağıtılmış düzende, uygulamanız bölgeleri kapsar.</span><span class="sxs-lookup"><span data-stu-id="85c0f-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="85c0f-111">Genel bulutu varsayılan olarak kullanabilirsiniz, ancak bazı kullanıcılarınız, verilerinin bölgesinde kalmasını gerektirebilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="85c0f-112">Kullanıcıları gereksinimlerine göre en uygun buluta yönlendirebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="85c0f-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="85c0f-113">Sorunlar ve dikkat edilmesi gerekenler</span><span class="sxs-lookup"><span data-stu-id="85c0f-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="85c0f-114">Ölçeklenebilirlik konusunda dikkat edilmesi gerekenler</span><span class="sxs-lookup"><span data-stu-id="85c0f-114">Scalability considerations</span></span>

<span data-ttu-id="85c0f-115">Bu makalede oluşturacağınız çözüm, ölçeklenebilirlik sağlamak için değildir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="85c0f-116">Ancak, diğer Azure ve şirket içi çözümlerle birlikte kullanılırsa ölçeklenebilirlik gereksinimlerine uyum sağlayabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="85c0f-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="85c0f-117">Traffic Manager aracılığıyla otomatik ölçeklendirmeyle karma çözüm oluşturma hakkında bilgi için bkz. [Azure ile platformlar arası ölçeklendirme çözümleri oluşturma](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="85c0f-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="85c0f-118">Kullanılabilirlik konusunda dikkat edilmesi gerekenler</span><span class="sxs-lookup"><span data-stu-id="85c0f-118">Availability considerations</span></span>

<span data-ttu-id="85c0f-119">Ölçeklenebilirlik konusunda olduğu gibi bu çözüm, kullanılabilirliği doğrudan ele almaz.</span><span class="sxs-lookup"><span data-stu-id="85c0f-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="85c0f-120">Ancak, Azure ve şirket içi çözümler bu çözüm içinde, dahil edilen tüm bileşenler için yüksek kullanılabilirlik sağlamak üzere uygulanabilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="85c0f-121">Bu düzenin kullanılacağı durumlar</span><span class="sxs-lookup"><span data-stu-id="85c0f-121">When to use this pattern</span></span>

- <span data-ttu-id="85c0f-122">Kuruluşunuzun özel bölgesel güvenlik ve dağıtım ilkeleri gerektiren Uluslararası dalları vardır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="85c0f-123">Kuruluşunuzun ofislerinin her biri, her yerel yönetmeliğe ve saat dilimlerine göre raporlama etkinliği gerektiren çalışan, iş ve tesis verilerini çeker.</span><span class="sxs-lookup"><span data-stu-id="85c0f-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="85c0f-124">Yüksek ölçekli gereksinimler, tek bir bölgede birden çok uygulama dağıtımına sahip uygulamaları yatay olarak ölçeklendirerek ve çok fazla yük gereksinimini işlemek için bölgeler arasında karşılanır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="85c0f-125">Topolojiyi planlama</span><span class="sxs-lookup"><span data-stu-id="85c0f-125">Planning the topology</span></span>

<span data-ttu-id="85c0f-126">Dağıtılmış bir uygulama ayak izi oluşturmadan önce, aşağıdaki şeyleri öğrenmenize yardımcı olur:</span><span class="sxs-lookup"><span data-stu-id="85c0f-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="85c0f-127">**Uygulama Için özel etki alanı:** Müşterilerin uygulamaya erişmek için kullanacağı özel etki alanı adı nedir?</span><span class="sxs-lookup"><span data-stu-id="85c0f-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="85c0f-128">Örnek uygulama için, özel etki alanı adı *www \. scalableasedemo.com* ' dir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="85c0f-129">**Traffic Manager etki alanı:** Bir [Azure Traffic Manager profili](/azure/traffic-manager/traffic-manager-manage-profiles)oluştururken bir etki alanı adı seçilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="85c0f-130">Bu ad, Traffic Manager tarafından yönetilen bir etki alanı girdisini kaydetmek için *trafficmanager.net* sonekiyle birleştirilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="85c0f-131">Örnek uygulama için, seçilen ad *ölçeklenebilir-Ao-demo*' dir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="85c0f-132">Sonuç olarak, Traffic Manager tarafından yönetilen tam etki alanı adı *Scalable-ASE-demo.trafficmanager.net*' dir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="85c0f-133">**Uygulama parmak izini ölçeklendirmeye yönelik strateji:** Uygulama parmak izin tek bir bölgede, birden çok bölgede veya her iki yaklaşımın bir karışımında birden çok App Service ortamına dağıtılıp dağıtılmayacağına karar verin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="85c0f-134">Karar, müşteri trafiğinin nereden kaynaklanacaktır ve bir uygulamanın destekleme arka uç altyapısının ne kadar iyi ölçeklendirilebileceğine ilişkin beklentileri temel almalıdır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="85c0f-135">Örneğin, %100 durum bilgisi içermeyen bir uygulamayla, Azure bölgesi başına birden çok App Service ortamının bir birleşimi kullanılarak, birden fazla Azure bölgesinde dağıtılan App Service ortamları ile çarpılarak bir uygulama daha büyük bir şekilde ölçeklendirilebilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="85c0f-136">Üzerinde seçim yapabileceğiniz 15 + küresel Azure bölgesi sayesinde müşteriler gerçek anlamda dünya genelinde bir hiper ölçekli uygulama ayak izi oluşturabilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="85c0f-137">Burada kullanılan örnek uygulama için, tek bir Azure bölgesinde üç App Service ortamı oluşturulmuştur (Orta Güney ABD).</span><span class="sxs-lookup"><span data-stu-id="85c0f-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="85c0f-138">**App Service ortamları Için adlandırma kuralı:** Her App Service ortamı benzersiz bir ad gerektirir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="85c0f-139">Bir veya iki App Service ortamının ötesinde, her bir App Service ortamının tanımlanmasına yardımcı olmak için bir adlandırma kuralı olması yararlı olacaktır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="85c0f-140">Burada kullanılan örnek uygulama için basit bir adlandırma kuralı kullanılmıştır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="85c0f-141">Üç App Service ortamının adları *fe1ase*, *fe2ase*ve *fe3ase*.</span><span class="sxs-lookup"><span data-stu-id="85c0f-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="85c0f-142">**Uygulamalar Için adlandırma kuralı:** Uygulamanın birden çok örneği dağıtılırsa, dağıtılan uygulamanın her örneği için bir ad gereklidir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="85c0f-143">Power Apps için App Service Ortamı ile aynı uygulama adı birden çok ortamda kullanılabilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="85c0f-144">Her App Service ortamı benzersiz bir etki alanı sonekine sahip olduğundan, geliştiriciler her ortamda tam olarak aynı uygulama adını kullanmayı seçebilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="85c0f-145">Örneğin, bir geliştirici şu şekilde adlandırılan uygulamalara sahip olabilir: *MyApp.foo1.p.azurewebsites.net*, *MyApp.Foo2.p.azurewebsites.net*, *MyApp.Foo3.p.azurewebsites.net*, vb.</span><span class="sxs-lookup"><span data-stu-id="85c0f-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="85c0f-146">Burada kullanılan uygulama için, her bir uygulama örneğinin benzersiz bir adı vardır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="85c0f-147">Kullanılan uygulama örneği adları *webfrontend1*, *webfrontend2*ve *webfrontend3*.</span><span class="sxs-lookup"><span data-stu-id="85c0f-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="85c0f-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="85c0f-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="85c0f-149">Microsoft Azure Stack hub, Azure uzantısıdır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="85c0f-150">Azure Stack hub, bulut bilgi işlemin çevikliğini ve yeniliklerini şirket içi ortamınıza sunarak, karma uygulamaları her yerde derleyip dağıtmanıza imkan tanıyan tek karma bulutu etkinleştirir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="85c0f-151">[Karma uygulama tasarımı ile ilgili önemli noktalar](overview-app-design-considerations.md) , karma uygulamalar tasarlamak, dağıtmak ve çalıştırmak için yazılım kalitesinin (yerleştirme, ölçeklenebilirlik, kullanılabilirlik, dayanıklılık, yönetilebilirlik ve güvenlik) aynı şekilde gözden geçirmeleri inceler.</span><span class="sxs-lookup"><span data-stu-id="85c0f-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="85c0f-152">Tasarım konuları karma uygulama tasarımını iyileştirirken, üretim ortamlarındaki zorlukları en aza indirmeyle ilgili olarak size yardımcı olur.</span><span class="sxs-lookup"><span data-stu-id="85c0f-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="85c0f-153">1. kısım: coğrafi olarak dağıtılmış bir uygulama oluşturma</span><span class="sxs-lookup"><span data-stu-id="85c0f-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="85c0f-154">Bu bölümde, bir Web uygulaması oluşturacaksınız.</span><span class="sxs-lookup"><span data-stu-id="85c0f-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="85c0f-155">Web uygulamaları oluşturun ve yayımlayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="85c0f-156">Azure Repos kod ekleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="85c0f-157">Uygulamanın yapısını birden çok bulut hedeflerine işaret edin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="85c0f-158">CD işlemini yönetin ve yapılandırın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="85c0f-159">Önkoşullar</span><span class="sxs-lookup"><span data-stu-id="85c0f-159">Prerequisites</span></span>

<span data-ttu-id="85c0f-160">Bir Azure aboneliği ve Azure Stack hub yüklemesi gereklidir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="85c0f-161">Coğrafi olarak dağıtılmış uygulama adımları</span><span class="sxs-lookup"><span data-stu-id="85c0f-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="85c0f-162">Özel etki alanı edinme ve DNS 'yi yapılandırma</span><span class="sxs-lookup"><span data-stu-id="85c0f-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="85c0f-163">Etki alanı için DNS bölge dosyasını güncelleştirin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="85c0f-164">Daha sonra Azure AD, özel etki alanı adının sahipliğini doğrulayabilirler.</span><span class="sxs-lookup"><span data-stu-id="85c0f-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="85c0f-165">Azure 'da Azure/Microsoft 365/dış DNS kayıtları için [Azure DNS](/azure/dns/dns-getstarted-portal) kullanın veya DNS girişini [farklı bir DNS kaydedicisinde](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider)ekleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-165">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="85c0f-166">Özel bir etki alanını ortak bir kayıt defteri ile kaydedin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="85c0f-167">Etki alanına ilişkin etki alanı adı kayıt şirketinde oturum açın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="85c0f-168">DNS güncelleştirmelerini yapmak için onaylanan yönetici gerekli olabilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="85c0f-169">Azure AD tarafından belirtilen DNS girişini ekleyerek etki alanı için DNS bölge dosyasını güncelleştirin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="85c0f-170">DNS girişi, posta yönlendirme veya Web barındırma gibi davranışları değiştirmez.</span><span class="sxs-lookup"><span data-stu-id="85c0f-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="85c0f-171">Web uygulamaları oluşturma ve yayımlama</span><span class="sxs-lookup"><span data-stu-id="85c0f-171">Create web apps and publish</span></span>

<span data-ttu-id="85c0f-172">Web uygulamasını Azure 'a ve Azure Stack hub 'a dağıtmak için karma sürekli tümleştirme/sürekli teslim (CI/CD) ayarlayın ve değişiklikleri her iki bulutda otomatik olarak gönderin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="85c0f-173">Çalıştırmak için (Windows Server ve SQL) dağıtılmış uygun görüntülerle Azure Stack hub ve App Service dağıtımı gereklidir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="85c0f-174">Daha fazla bilgi için, [Azure Stack hub 'ında App Service dağıtmaya yönelik önkoşullar](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)bölümüne bakın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="85c0f-175">Azure Repos kod ekleme</span><span class="sxs-lookup"><span data-stu-id="85c0f-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="85c0f-176">Azure Repos üzerinde **Proje oluşturma hakları** olan bir hesapla Visual Studio 'da oturum açın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="85c0f-177">CI/CD, hem uygulama kodu hem de altyapı kodu için uygulanabilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="85c0f-178">Hem özel hem de barındırılan bulut geliştirmesi için [Azure Resource Manager şablonları](https://azure.microsoft.com/resources/templates/) kullanın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Visual Studio 'da bir projeye bağlanma](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="85c0f-180">Varsayılan Web uygulamasını oluşturarak ve açarak **depoyu kopyalayın** .</span><span class="sxs-lookup"><span data-stu-id="85c0f-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Visual Studio 'da depoyu Kopyala](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="85c0f-182">Her iki bulutta Web uygulaması dağıtımı oluşturma</span><span class="sxs-lookup"><span data-stu-id="85c0f-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="85c0f-183">**WebApplication. csproj** dosyasını düzenleyin: seçin `Runtimeidentifier` ve ekleyin `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="85c0f-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="85c0f-184">(Bkz. [kendi Içinde dağıtım](/dotnet/core/deploying/deploy-with-vs#simpleSelf) belgeleri.)</span><span class="sxs-lookup"><span data-stu-id="85c0f-184">(See [Self-contained Deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Visual Studio 'da Web uygulaması proje dosyasını düzenleme](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="85c0f-186">Takım Gezgini kullanarak **Azure Repos kodu Iade edin** .</span><span class="sxs-lookup"><span data-stu-id="85c0f-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="85c0f-187">**Uygulama kodunun** Azure Repos işaretli olduğunu doğrulayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="85c0f-188">Derleme tanımı oluşturma</span><span class="sxs-lookup"><span data-stu-id="85c0f-188">Create the build definition</span></span>

1. <span data-ttu-id="85c0f-189">Derleme tanımları oluşturma yeteneğini onaylamak için **Azure pipelines ' de oturum açın** .</span><span class="sxs-lookup"><span data-stu-id="85c0f-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="85c0f-190">`-r win10-x64`Kod ekleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="85c0f-191">Bu ek, .NET Core ile bağımsız bir dağıtımı tetiklemek için gereklidir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Azure Pipelines içindeki derleme tanımına kod ekleme](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="85c0f-193">**Derlemeyi çalıştırın**.</span><span class="sxs-lookup"><span data-stu-id="85c0f-193">**Run the build**.</span></span> <span data-ttu-id="85c0f-194">[Kendi içinde çalışan dağıtım oluşturma](/dotnet/core/deploying/deploy-with-vs#simpleSelf) Işlemi, Azure 'da ve Azure Stack hub 'da çalışabilecek yapıtları yayımlar.</span><span class="sxs-lookup"><span data-stu-id="85c0f-194">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="85c0f-195">Azure barındırılan Aracısı kullanma</span><span class="sxs-lookup"><span data-stu-id="85c0f-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="85c0f-196">Azure Pipelines barındırılan bir aracının kullanılması, Web uygulamaları oluşturmak ve dağıtmak için kullanışlı bir seçenektir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="85c0f-197">Bakım ve yükseltmeler, kesintisiz geliştirme, test ve dağıtım sağlayan Microsoft Azure tarafından otomatik olarak gerçekleştirilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="85c0f-198">CD işlemini yönetme ve yapılandırma</span><span class="sxs-lookup"><span data-stu-id="85c0f-198">Manage and configure the CD process</span></span>

<span data-ttu-id="85c0f-199">Azure DevOps Services, yayınlar için geliştirme, hazırlık, QA ve üretim ortamları gibi birden çok ortama yüksek düzeyde yapılandırılabilir ve yönetilebilir bir işlem hattı sağlar; belirli aşamalarda onay gerektirme de dahil.</span><span class="sxs-lookup"><span data-stu-id="85c0f-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="85c0f-200">Yayın tanımı oluştur</span><span class="sxs-lookup"><span data-stu-id="85c0f-200">Create release definition</span></span>

1. <span data-ttu-id="85c0f-201">Azure DevOps Services **derleme ve yayınlama** bölümündeki **yayınlar** sekmesinin altına yeni bir yayın eklemek için **artı** düğmesini seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Azure DevOps Services bir yayın tanımı oluşturun](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="85c0f-203">Azure App Service Dağıtım şablonunu uygulayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-203">Apply the Azure App Service Deployment template.</span></span>

   ![Azure DevOps Services Azure App Service dağıtım şablonu Uygula](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="85c0f-205">**Yapıt Ekle**' nin altında, Azure Cloud Build uygulaması için yapıt ekleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Azure DevOps Services içinde Azure Cloud derlemesine yapıt ekleme](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="85c0f-207">İşlem hattı sekmesi altında, ortamın **aşamasını, görev** bağlantısını seçin ve Azure bulut ortamı değerlerini ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Azure DevOps Services 'de Azure bulut ortamı değerlerini ayarlama](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="85c0f-209">**Ortam adını** ayarlayın ve Azure bulut uç noktası için **Azure aboneliğini** seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Azure DevOps Services 'de Azure bulut uç noktası için Azure aboneliğini seçin](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="85c0f-211">**App Service adı**altında, gerekli Azure App Service adını ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Azure App Service adını Azure DevOps Services ayarla](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="85c0f-213">Azure bulut barındırılan ortamı için **Aracı kuyruğu** altında "barındırılan VS2017" yazın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Azure DevOps Services 'de Azure bulut barındırılan ortamı için aracı kuyruğunu ayarla](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="85c0f-215">Azure App Service dağıt menüsünde, ortam için geçerli **paketi veya klasörü** seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="85c0f-216">**Klasör konumuna** **Tamam ' ı** seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-216">Select **OK** to **folder location**.</span></span>
  
      ![Azure DevOps Services Azure App Service ortam için paket veya klasör seçin](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Azure DevOps Services Azure App Service ortam için paket veya klasör seçin](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="85c0f-219">Tüm değişiklikleri kaydedin ve **yayın ardışık düzenine**geri dönün.</span><span class="sxs-lookup"><span data-stu-id="85c0f-219">Save all changes and go back to **release pipeline**.</span></span>

    ![Yayın işlem hattındaki değişiklikleri Azure DevOps Services Kaydet](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="85c0f-221">Azure Stack Hub uygulaması için yapıyı seçerek yeni bir yapıt ekleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Azure DevOps Services Azure Stack Hub uygulaması için yeni yapıt ekleme](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="85c0f-223">Azure App Service dağıtımını uygulayarak bir ortam daha ekleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Azure DevOps Services içinde Azure App Service dağıtımına ortam ekleme](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="85c0f-225">Yeni ortam Azure Stack hub 'ına adlandırın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-225">Name the new environment Azure Stack Hub.</span></span>

    ![Azure DevOps Services içinde Azure App Service dağıtımında ad ortamı](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="85c0f-227">**Görev** sekmesinde Azure Stack hub ortamını bulun.</span><span class="sxs-lookup"><span data-stu-id="85c0f-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Azure DevOps Services Azure DevOps Services Azure Stack hub ortamı](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="85c0f-229">Azure Stack hub uç noktası için aboneliği seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Azure DevOps Services Azure Stack hub uç noktası için abonelik seçin](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="85c0f-231">Azure Stack Hub web uygulaması adını App Service adı olarak ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Azure DevOps Services Azure Stack Hub web uygulaması adını ayarlama](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="85c0f-233">Azure Stack hub aracısını seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-233">Select the Azure Stack Hub agent.</span></span>

    ![Azure DevOps Services Azure Stack hub aracısını seçin](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="85c0f-235">Dağıtım Azure App Service bölümünde, ortam için geçerli **paketi veya klasörü** seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="85c0f-236">Klasör konumuna **Tamam ' ı** seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-236">Select **OK** to folder location.</span></span>

    ![Azure DevOps Services Azure App Service dağıtım için klasör seçin](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Azure DevOps Services Azure App Service dağıtım için klasör seçin](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="85c0f-239">Değişken sekmesi altında adlı bir değişken ekleyin `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , değerini **doğru**olarak ayarlayın ve Azure Stack hub ile kapsamını belirleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Azure DevOps Services Azure Uygulama dağıtımına değişken ekleme](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="85c0f-241">Her iki yapıt içinde **sürekli** dağıtım tetikleme simgesini seçin ve **devam** eden dağıtım tetikleyicisini etkinleştirin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Azure DevOps Services içinde sürekli dağıtım tetikleyicisi seçin](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="85c0f-243">Azure Stack hub ortamında **dağıtım öncesi** koşullar simgesini seçin ve tetikleyiciyi **yayından sonra** olarak ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Azure DevOps Services dağıtım öncesi koşullarını seçin](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="85c0f-245">Tüm değişiklikleri kaydedin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="85c0f-246">Görevler için bazı ayarlar, bir şablondan bir yayın tanımı oluşturulurken otomatik olarak [ortam değişkenleri](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) olarak tanımlanabilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-246">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="85c0f-247">Bu ayarlar görev ayarlarında değiştirilemez; Bunun yerine, bu ayarları düzenlemek için üst ortam öğesinin seçilmesi gerekir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="85c0f-248">2. Bölüm: Web uygulaması seçeneklerini güncelleştirme</span><span class="sxs-lookup"><span data-stu-id="85c0f-248">Part 2: Update web app options</span></span>

<span data-ttu-id="85c0f-249">[Azure App Service](/azure/app-service/overview), yüksek oranda ölçeklenebilen, kendi kendine düzeltme eki uygulayan bir web barındırma hizmeti sunar.</span><span class="sxs-lookup"><span data-stu-id="85c0f-249">[Azure App Service](/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="85c0f-251">Mevcut bir özel DNS adını Azure Web Apps eşleştirin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="85c0f-252">Özel bir DNS adını App Service eşlemek için bir **CNAME kaydı** ve **a kaydı** kullanın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="85c0f-253">Mevcut bir özel DNS adını Azure Web Apps ile eşleme</span><span class="sxs-lookup"><span data-stu-id="85c0f-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="85c0f-254">Kök etki alanı dışındaki tüm özel DNS adları için CNAME kullanın (örneğin, northwind.com).</span><span class="sxs-lookup"><span data-stu-id="85c0f-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="85c0f-255">Canlı siteyi ve onun DNS etki alanı adını App Service'e geçirmek için, bkz. [Etkin DNS adını Azure App Service'e geçirme](/azure/app-service/manage-custom-dns-migrate-domain).</span><span class="sxs-lookup"><span data-stu-id="85c0f-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="85c0f-256">Önkoşullar</span><span class="sxs-lookup"><span data-stu-id="85c0f-256">Prerequisites</span></span>

<span data-ttu-id="85c0f-257">Bu çözümü gerçekleştirmek için:</span><span class="sxs-lookup"><span data-stu-id="85c0f-257">To complete this solution:</span></span>

- <span data-ttu-id="85c0f-258">[App Service bir uygulama oluşturun](/azure/app-service/)veya başka bir çözüm için oluşturulmuş bir uygulama kullanın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-258">[Create an App Service app](/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="85c0f-259">Etki alanı adı satın alıp etki alanı sağlayıcısı için DNS kayıt defterine erişim sağlayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="85c0f-260">Etki alanı için DNS bölge dosyasını güncelleştirin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="85c0f-261">Azure AD, özel etki alanı adının sahipliğini doğrulayacaktır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="85c0f-262">Azure 'da Azure/Microsoft 365/dış DNS kayıtları için [Azure DNS](/azure/dns/dns-getstarted-portal) kullanın veya DNS girişini [farklı bir DNS kaydedicisinde](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider)ekleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-262">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

- <span data-ttu-id="85c0f-263">Özel bir etki alanını ortak bir kayıt defteri ile kaydedin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="85c0f-264">Etki alanına ilişkin etki alanı adı kayıt şirketinde oturum açın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="85c0f-265">(DNS güncelleştirmeleri yapmak için onaylanan yönetici gerekli olabilir.)</span><span class="sxs-lookup"><span data-stu-id="85c0f-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="85c0f-266">Azure AD tarafından belirtilen DNS girişini ekleyerek etki alanı için DNS bölge dosyasını güncelleştirin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="85c0f-267">Örneğin, northwindcloud.com ve www northwindcloud.com için DNS girişleri eklemek için \. , northwindcloud.com kök etki alanı IÇIN DNS ayarlarını yapılandırın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="85c0f-268">[Azure Portal](/azure/app-service/manage-custom-dns-buy-domain)kullanarak bir etki alanı adı satın alınabilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-268">A domain name may be purchased using the [Azure portal](/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="85c0f-269">Özel DNS adını web uygulamasına eşlemek için, web uygulamasının [App Service planı](https://azure.microsoft.com/pricing/details/app-service/) ücretli bir katmanda (**Paylaşılan**, **Temel**, **Standart** veya **Premium** olmalıdır).</span><span class="sxs-lookup"><span data-stu-id="85c0f-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="85c0f-270">CNAME ve A kayıtlarını oluşturma ve eşleme</span><span class="sxs-lookup"><span data-stu-id="85c0f-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="85c0f-271">Etki alanı sağlayıcısı ile DNS kayıtlarına erişme</span><span class="sxs-lookup"><span data-stu-id="85c0f-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="85c0f-272">Azure Web Apps için özel bir DNS adı yapılandırmak üzere Azure DNS kullanın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="85c0f-273">Daha fazla bilgi için bkz. [Bir Azure hizmeti için özel etki alanı ayarları sağlamak üzere Azure DNS'yi kullanma](/azure/dns/dns-custom-domain).</span><span class="sxs-lookup"><span data-stu-id="85c0f-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="85c0f-274">Ana sağlayıcının web sitesinde oturum açın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="85c0f-275">DNS kayıtlarını yönetme sayfasını bulun.</span><span class="sxs-lookup"><span data-stu-id="85c0f-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="85c0f-276">Her etki alanı sağlayıcısının kendi DNS kayıtları arabirimi vardır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="85c0f-277">Sitede **Domain Name**, **DNS** veya **Name Server Management** etiketli alanları bulun.</span><span class="sxs-lookup"><span data-stu-id="85c0f-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="85c0f-278">DNS kayıtları sayfası, **etki Alanlarmda**görüntülenebilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="85c0f-279">**Bölge dosyası**, **DNS kayıtları**veya **Gelişmiş yapılandırma**adlı bağlantıyı bulun.</span><span class="sxs-lookup"><span data-stu-id="85c0f-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="85c0f-280">DNS kayıtları sayfasının bir örneğini aşağıdaki ekran görüntüsünde görebilirsiniz:</span><span class="sxs-lookup"><span data-stu-id="85c0f-280">The following screenshot is an example of a DNS records page:</span></span>

![Örnek DNS kayıtları sayfası](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="85c0f-282">Kayıt oluşturmak için etki alanı adı kaydedicisi ' nde **Ekle veya oluştur** ' u seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="85c0f-283">Bazı sağlayıcıların farklı kayıt türlerini eklemek için farklı bağlantıları vardır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="85c0f-284">Sağlayıcının belgelerine başvurun.</span><span class="sxs-lookup"><span data-stu-id="85c0f-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="85c0f-285">Bir alt etki alanını uygulamanın varsayılan ana bilgisayar adına eşlemek için bir CNAME kaydı ekleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="85c0f-286">Www \. northwindcloud.com etki alanı örneği için, adı ile eşleyen BIR CNAME kaydı ekleyin `<app_name>.azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="85c0f-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="85c0f-287">CNAME eklendikten sonra DNS kayıtları sayfası aşağıdaki örneğe benzer şekilde görünür:</span><span class="sxs-lookup"><span data-stu-id="85c0f-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Azure uygulamasına portal gezintisi](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="85c0f-289">Azure'da CNAME kaydı eşlemesini etkinleştirme</span><span class="sxs-lookup"><span data-stu-id="85c0f-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="85c0f-290">Yeni bir sekmede Azure portal oturum açın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="85c0f-291">Uygulama Hizmetleri'ne gidin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-291">Go to App Services.</span></span>

3. <span data-ttu-id="85c0f-292">Web uygulaması ' nı seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-292">Select web app.</span></span>

4. <span data-ttu-id="85c0f-293">Azure Portal'daki uygulama sayfasının sol gezintisinde **Özel etki alanları**'nı seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="85c0f-294">**+** **Konak adı Ekle**' nin yanındaki simgeyi seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="85c0f-295">Tam etki alanı adını (gibi) yazın `www.northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="85c0f-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="85c0f-296">**Doğrula**'yı seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-296">Select **Validate**.</span></span>

8. <span data-ttu-id="85c0f-297">Belirtilmişse, `A` `TXT` etki alanı adı kayıt şirketlerinde DNS kayıtlarına diğer türlerin (veya) ek kayıtlarını ekleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="85c0f-298">Azure, bu kayıtların değerlerini ve türlerini sağlar:</span><span class="sxs-lookup"><span data-stu-id="85c0f-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="85c0f-299">a.</span><span class="sxs-lookup"><span data-stu-id="85c0f-299">a.</span></span>  <span data-ttu-id="85c0f-300">Uygulamanın IP adresini eşlemek için bir **A** kaydı.</span><span class="sxs-lookup"><span data-stu-id="85c0f-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="85c0f-301">b.</span><span class="sxs-lookup"><span data-stu-id="85c0f-301">b.</span></span>  <span data-ttu-id="85c0f-302">Uygulamanın varsayılan konak adını (`<app_name>.azurewebsites.net`) eşlemek için bir **TXT** kaydı.</span><span class="sxs-lookup"><span data-stu-id="85c0f-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="85c0f-303">App Service, özel etki alanı sahipliğini doğrulamak için bu kaydı yalnızca yapılandırma zamanında kullanır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="85c0f-304">Doğrulamadan sonra TXT kaydını silin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="85c0f-305">Bu görevi etki alanı kaydedici sekmesinde doldurun ve **ana bilgisayar adı Ekle** düğmesi etkinleştirilinceye kadar yeniden doğrulayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="85c0f-306">**Ana bilgisayar adı kayıt türünün** **CNAME** (www.example.com veya herhangi bir alt etki alanı) olarak ayarlandığından emin olun.</span><span class="sxs-lookup"><span data-stu-id="85c0f-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="85c0f-307">**Konak adı ekle**'yi seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="85c0f-308">Tam etki alanı adını (gibi) yazın `northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="85c0f-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="85c0f-309">**Doğrula**'yı seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-309">Select **Validate**.</span></span> <span data-ttu-id="85c0f-310">**Ekleme** etkinleştirilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="85c0f-311">**Ana bilgisayar adı kayıt türünün** **bir kayıt** (example.com) olarak ayarlandığından emin olun.</span><span class="sxs-lookup"><span data-stu-id="85c0f-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="85c0f-312">**Konak adı Ekle**.</span><span class="sxs-lookup"><span data-stu-id="85c0f-312">**Add hostname**.</span></span>

    <span data-ttu-id="85c0f-313">Yeni ana bilgisayar adlarının uygulamanın **özel etki alanları** sayfasında yansıtılması biraz zaman alabilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="85c0f-314">Verileri güncelleştirmek için tarayıcıyı yenilemeyi deneyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-314">Try refreshing the browser to update the data.</span></span>
  
    ![Özel etki alanları](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="85c0f-316">Bir hata oluşursa, sayfanın alt kısmında bir doğrulama hatası bildirimi görüntülenir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![Etki alanı doğrulama hatası](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="85c0f-318">Yukarıdaki adımlar, joker bir etki alanını ( \* . northwindcloud.com) eşlemek için yinelenebilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="85c0f-319">Bu, her biri için ayrı bir CNAME kaydı oluşturmak zorunda kalmadan bu App Service 'e ek alt etki alanları eklenmesine izin verir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="85c0f-320">Bu ayarı yapılandırmak için kaydedici yönergelerini izleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="85c0f-321">Bir tarayıcıda test etme</span><span class="sxs-lookup"><span data-stu-id="85c0f-321">Test in a browser</span></span>

<span data-ttu-id="85c0f-322">Daha önce yapılandırılan DNS adlarını (örneğin, `northwindcloud.com` veya `www.northwindcloud.com` ) inceleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="85c0f-323">3. kısım: özel bir SSL sertifikası bağlama</span><span class="sxs-lookup"><span data-stu-id="85c0f-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="85c0f-324">Bu bölümde şunları göndereceğiz:</span><span class="sxs-lookup"><span data-stu-id="85c0f-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="85c0f-325">Özel SSL sertifikasını App Service bağlayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="85c0f-326">Uygulama için HTTPS 'yi zorunlu tutun.</span><span class="sxs-lookup"><span data-stu-id="85c0f-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="85c0f-327">SSL sertifikası bağlamasını betiklerle otomatikleştirin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="85c0f-328">Gerekirse, Azure portal bir müşteri SSL sertifikası alın ve Web uygulamasına bağlayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="85c0f-329">Daha fazla bilgi için [App Service sertifikaları öğreticisine](/azure/app-service/web-sites-purchase-ssl-web-site)bakın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-329">For more information, see the [App Service Certificates tutorial](/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="85c0f-330">Önkoşullar</span><span class="sxs-lookup"><span data-stu-id="85c0f-330">Prerequisites</span></span>

<span data-ttu-id="85c0f-331">Bu çözümü gerçekleştirmek için:</span><span class="sxs-lookup"><span data-stu-id="85c0f-331">To complete this  solution:</span></span>

- [<span data-ttu-id="85c0f-332">App Service uygulaması oluşturun.</span><span class="sxs-lookup"><span data-stu-id="85c0f-332">Create an App Service app.</span></span>](/azure/app-service/)
- [<span data-ttu-id="85c0f-333">Özel bir DNS adını Web uygulamanıza eşleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-333">Map a custom DNS name to your web app.</span></span>](/azure/app-service/app-service-web-tutorial-custom-domain)
- <span data-ttu-id="85c0f-334">Güvenilir bir sertifika yetkilisinden SSL sertifikası alın ve bu anahtarı kullanarak isteği imzalayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="85c0f-335">SSL sertifikanıza yönelik gereksinimler</span><span class="sxs-lookup"><span data-stu-id="85c0f-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="85c0f-336">Bir sertifikayı App Service’te kullanabilmek için sertifikanın aşağıdaki tüm gereksinimleri karşılaması gerekir:</span><span class="sxs-lookup"><span data-stu-id="85c0f-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="85c0f-337">Güvenilen bir sertifika yetkilisi tarafından imzalandı.</span><span class="sxs-lookup"><span data-stu-id="85c0f-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="85c0f-338">Parola korumalı bir PFX dosyası olarak verildi.</span><span class="sxs-lookup"><span data-stu-id="85c0f-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="85c0f-339">En az 2048 bit uzunluğunda özel anahtar içerir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="85c0f-340">Sertifika zincirindeki tüm ara sertifikaları içerir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="85c0f-341">**Eliptik Eğri Şifreleme (ECC) sertifikaları** App Service ile çalışır, ancak bu kılavuza dahil değildir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="85c0f-342">ECC sertifikaları oluşturma konusunda yardım almak için bir sertifika yetkilisine başvurun.</span><span class="sxs-lookup"><span data-stu-id="85c0f-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="85c0f-343">Web uygulamasını hazırlama</span><span class="sxs-lookup"><span data-stu-id="85c0f-343">Prepare the web app</span></span>

<span data-ttu-id="85c0f-344">Özel bir SSL sertifikasını Web uygulamasına bağlamak için [App Service planının](https://azure.microsoft.com/pricing/details/app-service/) **temel**, **Standart**veya **Premium** katmanda olması gerekir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="85c0f-345">Azure'da oturum açma</span><span class="sxs-lookup"><span data-stu-id="85c0f-345">Sign in to Azure</span></span>

1. <span data-ttu-id="85c0f-346">[Azure Portal](https://portal.azure.com/) açın ve Web uygulamasına gidin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="85c0f-347">Sol menüden **uygulama hizmetleri**' ni ve ardından Web uygulaması adını seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Azure portal web uygulaması seçin](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="85c0f-349">Fiyatlandırma katmanını denetleme</span><span class="sxs-lookup"><span data-stu-id="85c0f-349">Check the pricing tier</span></span>

1. <span data-ttu-id="85c0f-350">Web uygulaması sayfasının sol tarafında, **Ayarlar** bölümüne gidin ve **ölçeği büyütme (App Service planı)** seçeneğini belirleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Web uygulamasındaki ölçek menüsü](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="85c0f-352">Web uygulamasının **ücretsiz** veya **paylaşılan** katmanda olmadığından emin olun.</span><span class="sxs-lookup"><span data-stu-id="85c0f-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="85c0f-353">Web uygulamasının geçerli katmanı, koyu mavi bir kutu içinde vurgulanır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![Web uygulamasındaki fiyatlandırma katmanını denetle](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="85c0f-355">Özel SSL, **ücretsiz** veya **paylaşılan** katmanda desteklenmez.</span><span class="sxs-lookup"><span data-stu-id="85c0f-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="85c0f-356">Üst ölçekte, sonraki bölümde bulunan adımları izleyin veya **fiyatlandırma katmanınızı seçin** SAYFASıNDA, [SSL sertifikanızı karşıya yüklemek ve bağlamak](/azure/app-service/app-service-web-tutorial-custom-ssl)için atlayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="85c0f-357">App Service planınızın ölçeğini artırma</span><span class="sxs-lookup"><span data-stu-id="85c0f-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="85c0f-358">**Temel**, **Standart** veya **Premium** katmanlarından birini seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="85c0f-359">**Seç**’i seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-359">Select **Select**.</span></span>

![Web uygulamanız için fiyatlandırma katmanı seçin](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="85c0f-361">Bildirim görüntülenirken, ölçek işlemi tamamlanır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-361">The scale operation is complete when notification is displayed.</span></span>

![Ölçek artırma bildirimi](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="85c0f-363">SSL sertifikanızı bağlama ve ara sertifikaları birleştirme</span><span class="sxs-lookup"><span data-stu-id="85c0f-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="85c0f-364">Zincirdeki birden çok sertifikayı birleştirin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="85c0f-365">Bir metin düzenleyicisinde aldığınız **her sertifikayı açın** .</span><span class="sxs-lookup"><span data-stu-id="85c0f-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="85c0f-366">*Birleştirilebilen sertifika. CRT*adlı birleştirilmiş sertifika için bir dosya oluşturun.</span><span class="sxs-lookup"><span data-stu-id="85c0f-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="85c0f-367">Bir metin düzenleyicisinde her bir sertifikanın içeriğini bu dosyaya kopyalayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="85c0f-368">Sertifikalarınızın sırası, sertifikanızla başlayıp kök sertifika ile sona ererek sertifika zincirindeki sırayla aynı olmalıdır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="85c0f-369">Aşağıdaki örneğe benzer şekilde görünür:</span><span class="sxs-lookup"><span data-stu-id="85c0f-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="85c0f-370">Sertifikayı PFX dosyasına aktarma</span><span class="sxs-lookup"><span data-stu-id="85c0f-370">Export certificate to PFX</span></span>

<span data-ttu-id="85c0f-371">Birleştirilmiş SSL sertifikasını sertifika tarafından oluşturulan özel anahtarla dışarı aktarın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="85c0f-372">OpenSSL aracılığıyla bir özel anahtar dosyası oluşturulur.</span><span class="sxs-lookup"><span data-stu-id="85c0f-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="85c0f-373">Sertifikayı PFX 'e aktarmak için aşağıdaki komutu çalıştırın ve yer tutucuları `<private-key-file>` ve `<merged-certificate-file>` özel anahtar yolu ve birleştirilmiş sertifika dosyası ile değiştirin:</span><span class="sxs-lookup"><span data-stu-id="85c0f-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="85c0f-374">İstendiğinde, daha sonra App Service SSL sertifikanızı karşıya yüklemek için bir dışarı aktarma parolası tanımlayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="85c0f-375">Sertifika isteğini oluşturmak için IIS veya **Certreq.exe** kullanıldığında, sertifikayı yerel bir makineye yükler ve ardından [sertifikayı PFX 'e dışarı aktarın](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span><span class="sxs-lookup"><span data-stu-id="85c0f-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="85c0f-376">SSL sertifikasını karşıya yükleme</span><span class="sxs-lookup"><span data-stu-id="85c0f-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="85c0f-377">Web uygulamasının sol gezinti bölmesinde **SSL ayarları** ' nı seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="85c0f-378">**Sertifikayı karşıya yükle**' yi seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="85c0f-379">**PFX Sertifika dosyasında**pfx dosyası ' nı seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="85c0f-380">**Sertifika parolası**' nda, pfx dosyası dışarı aktarılırken oluşturulan parolayı yazın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="85c0f-381">**Karşıya Yükle**’yi seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-381">Select **Upload**.</span></span>

    ![SSL sertifikasını karşıya yükle](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="85c0f-383">App Service sertifikayı karşıya yüklemeyi bitirdiğinde, bu, **SSL ayarları** sayfasında görüntülenir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![SSL Ayarları](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="85c0f-385">SSL sertifikanızı bağlama</span><span class="sxs-lookup"><span data-stu-id="85c0f-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="85c0f-386">**SSL bağlamaları** bölümünde **bağlama Ekle**' yi seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="85c0f-387">Sertifika karşıya yüklenmişse, ancak **ana bilgisayar** adı açılır listesinde etki alanı adında görünmüyorsa, tarayıcı sayfasını yenilemeyi deneyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="85c0f-388">**SSL bağlaması Ekle** sayfasında, güvenli hale getirmek için etki alanı adını ve kullanılacak sertifikayı seçmek için açılan listeleri kullanın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="85c0f-389">**SSL Türü** menüsünde [**Sunucu Adı Belirtme (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) veya IP tabanlı SSL seçeneklerinden hangisini kullanacağınızı belirleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="85c0f-390">**SNI tabanlı SSL**: birden çok SNı tabanlı SSL bağlamaları eklenebilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="85c0f-391">Bu seçenek, aynı IP adresi üzerinde birden fazla SSL sertifikası ile birden fazla etki alanının güvenliğini sağlamaya olanak tanır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="85c0f-392">Çoğu modern tarayıcı (Internet Explorer, Chrome, Firefox ve Opera dahil) SNI’yi destekler (daha kapsamlı tarayıcı desteği bilgilerini [Sunucu Adı Belirtimi](https://wikipedia.org/wiki/Server_Name_Indication) bölümünde bulabilirsiniz).</span><span class="sxs-lookup"><span data-stu-id="85c0f-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="85c0f-393">**IP tabanlı SSL**: yalnızca bir IP tabanlı SSL bağlaması eklenebilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="85c0f-394">Bu seçenek yalnızca bir SSL sertifikası ile ayrılmış bir genel IP adresinin güvenliğini sağlamaya olanak tanır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="85c0f-395">Birden çok etki alanının güvenliğini sağlamak için, bunların tümünü aynı SSL sertifikası ile güvenli hale getirin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="85c0f-396">IP tabanlı SSL, SSL bağlama için geleneksel bir seçenektir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="85c0f-397">**Bağlama Ekle**' yi seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-397">Select **Add Binding**.</span></span>

    ![SSL bağlaması Ekle](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="85c0f-399">App Service sertifikayı karşıya yüklemeyi bitirdiğinde, **SSL bağlamaları** bölümlerinde görüntülenir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![SSL bağlamaları karşıya yüklemeyi bitirdi](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="85c0f-401">IP SSL için bir kaydı yeniden eşleyin</span><span class="sxs-lookup"><span data-stu-id="85c0f-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="85c0f-402">Web uygulamasında IP tabanlı SSL kullanılmıyorsa, [özel etki alanınız Için test https](/azure/app-service/app-service-web-tutorial-custom-ssl)'ye atlayın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="85c0f-403">Varsayılan olarak, Web uygulaması paylaşılan bir genel IP adresi kullanır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="85c0f-404">Sertifika, IP tabanlı SSL ile bağlandığında, App Service Web uygulaması için yeni ve ayrılmış bir IP adresi oluşturur.</span><span class="sxs-lookup"><span data-stu-id="85c0f-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="85c0f-405">Bir kayıt Web uygulamasına eşlendiğinde, etki alanı kayıt defteri ayrılmış IP adresi ile birlikte güncelleştirilmeleri gerekir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="85c0f-406">**Özel etki alanı** sayfası, yeni ve ayrılmış IP adresi ile güncelleştirilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="85c0f-407">Bu [IP adresini](/azure/app-service/app-service-web-tutorial-custom-domain)kopyalayın ve ardından [bir kaydı](/azure/app-service/app-service-web-tutorial-custom-domain) bu yeni IP adresiyle yeniden eşleyin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-407">Copy this [IP address](/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="85c0f-408">HTTPS’yi test etme</span><span class="sxs-lookup"><span data-stu-id="85c0f-408">Test HTTPS</span></span>

<span data-ttu-id="85c0f-409">Farklı tarayıcılarda, `https://<your.custom.domain>` Web uygulamasının sunulmasını sağlamak için bölümüne gidin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![Web uygulamasına gidin](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="85c0f-411">Sertifika doğrulama hataları oluşursa, kendinden imzalı bir sertifika neden olabilir ya da PFX dosyasına aktarılırken ara sertifikalar bırakılmış olabilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="85c0f-412">HTTPS'yi zorunlu tutma</span><span class="sxs-lookup"><span data-stu-id="85c0f-412">Enforce HTTPS</span></span>

<span data-ttu-id="85c0f-413">Varsayılan olarak, herkes HTTP kullanarak Web uygulamasına erişebilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="85c0f-414">HTTPS bağlantı noktasına yapılan tüm HTTP istekleri yeniden yönlendirilebilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="85c0f-415">Web uygulaması sayfasında **SL ayarları**' nı seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="85c0f-416">Ardından **Yalnızca HTTPS** menüsünde **Açık**’ı seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-416">Then, in **HTTPS Only**, select **On**.</span></span>

![HTTPS'yi zorunlu tutma](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="85c0f-418">İşlem tamamlandığında, uygulamayı işaret eden HTTP URL 'Lerinden birine gidin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="85c0f-419">Örneğin:</span><span class="sxs-lookup"><span data-stu-id="85c0f-419">For example:</span></span>

- <span data-ttu-id="85c0f-420">https://<app_name>. azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="85c0f-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="85c0f-421">TLS 1.1/1.2 zorlama</span><span class="sxs-lookup"><span data-stu-id="85c0f-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="85c0f-422">Uygulama varsayılan olarak, artık sektör standartları ( [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)gibi) tarafından güvenli olarak kabul edilen [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1,0 ' i sağlar.</span><span class="sxs-lookup"><span data-stu-id="85c0f-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="85c0f-423">Daha yüksek TLS sürümlerini zorlamak için şu adımları izleyin:</span><span class="sxs-lookup"><span data-stu-id="85c0f-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="85c0f-424">Web uygulaması sayfasında, sol gezinti bölmesinde **SSL ayarları**' nı seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="85c0f-425">**TLS sürümü**' nde, en düşük TLS sürümünü seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![TLS 1.1 veya 1.2’yi zorlama](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="85c0f-427">Traffic Manager profili oluşturma</span><span class="sxs-lookup"><span data-stu-id="85c0f-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="85c0f-428">**Create a resource**  >  **Networking**  >  **Profil**  >  **Oluştur**Traffic Manager kaynak ağı oluştur ' u seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="85c0f-429">**Traffic Manager profili oluştur** dikey penceresini aşağıdaki gibi doldurun:</span><span class="sxs-lookup"><span data-stu-id="85c0f-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="85c0f-430">**Ad**alanına profil için bir ad girin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="85c0f-431">Bu adın trafik manager.net bölgesinde benzersiz olması gerekir ve Traffic Manager profiline erişmek için kullanılan DNS adı, trafficmanager.net ile sonuçlanır.</span><span class="sxs-lookup"><span data-stu-id="85c0f-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="85c0f-432">**Yönlendirme yöntemi**' nde **coğrafi yönlendirme yöntemini**seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="85c0f-433">**Abonelikte**, bu profilin oluşturulacağı aboneliği seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="85c0f-434">**Kaynak Grubu** alanında bu profilin yerleştirileceği yeni bir kaynak grubu oluşturun.</span><span class="sxs-lookup"><span data-stu-id="85c0f-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="85c0f-435">**Kaynak grubu konumu** alanında kaynak grubunun konumunu seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="85c0f-436">Bu ayar, kaynak grubunun konumunu ifade eder ve genel olarak dağıtılan Traffic Manager profilini etkilemez.</span><span class="sxs-lookup"><span data-stu-id="85c0f-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="85c0f-437">**Oluştur**’u seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-437">Select **Create**.</span></span>

    7. <span data-ttu-id="85c0f-438">Traffic Manager profilinin genel dağıtımı tamamlandığında, ilgili kaynak grubunda kaynaklardan biri olarak listelenir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Traffic Manager profili oluşturma içindeki kaynak grupları](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="85c0f-440">Traffic Manager uç noktalarını ekleme</span><span class="sxs-lookup"><span data-stu-id="85c0f-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="85c0f-441">Portal arama çubuğunda, yukarıdaki bölümde oluşturulan **Traffic Manager profili** adını arayın ve görünen sonuçlarda Traffic Manager profilini seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="85c0f-442">**Traffic Manager profili**' nde, **Ayarlar** bölümünde **uç noktalar**' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="85c0f-443">**Ekle**’yi seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-443">Select **Add**.</span></span>

4. <span data-ttu-id="85c0f-444">Azure Stack hub uç noktası ekleniyor.</span><span class="sxs-lookup"><span data-stu-id="85c0f-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="85c0f-445">**Tür**için **dış uç nokta**' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="85c0f-446">Bu uç **nokta için,** ideal olarak Azure Stack merkezinin adını girin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="85c0f-447">Tam etki alanı adı (**FQDN**) için, Azure Stack Hub web uygulaması için dış URL 'yi kullanın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="85c0f-448">Coğrafi eşleme altında kaynağın bulunduğu bölgeyi/kıta seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="85c0f-449">Örneğin, **Avrupa.**</span><span class="sxs-lookup"><span data-stu-id="85c0f-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="85c0f-450">Görüntülenen ülke/bölge açılır kısmında, bu uç nokta için geçerli olan ülkeyi seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="85c0f-451">Örneğin, **Almanya**.</span><span class="sxs-lookup"><span data-stu-id="85c0f-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="85c0f-452">**Devre dışı olarak ekle** seçeneğini işaretsiz bırakın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="85c0f-453">**Tamam**’ı seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-453">Select **OK**.</span></span>

12. <span data-ttu-id="85c0f-454">Azure Uç Noktası Ekleme:</span><span class="sxs-lookup"><span data-stu-id="85c0f-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="85c0f-455">**Tür**için **Azure uç noktası**' nı seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="85c0f-456">Uç nokta için bir **ad** girin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="85c0f-457">**Hedef kaynak türü**için **App Service**seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="85c0f-458">**Hedef kaynak**için, aynı abonelikte Web Apps listesini göstermek üzere **bir App Service seçin** öğesini seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="85c0f-459">**Kaynak**bölümünde ilk uç nokta olarak kullanılan App Service 'i seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="85c0f-460">Coğrafi eşleme altında kaynağın bulunduğu bölgeyi/kıta seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="85c0f-461">Örneğin, **Kuzey Amerika/Orta Amerika/Karayipler.**</span><span class="sxs-lookup"><span data-stu-id="85c0f-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="85c0f-462">Görüntülenen ülke/bölge açılır listesi altında, yukarıdaki bölgesel gruplamayı seçmek için bu noktayı boş bırakın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="85c0f-463">**Devre dışı olarak ekle** seçeneğini işaretsiz bırakın.</span><span class="sxs-lookup"><span data-stu-id="85c0f-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="85c0f-464">**Tamam**’ı seçin.</span><span class="sxs-lookup"><span data-stu-id="85c0f-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="85c0f-465">Kaynak için varsayılan uç nokta olarak kullanılacak bir coğrafi kapsamı (Dünya) olan en az bir uç nokta oluşturun.</span><span class="sxs-lookup"><span data-stu-id="85c0f-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="85c0f-466">Her iki uç noktanın eklenmesi tamamlandığında, bunlar **Traffic Manager profilinde** görüntülenir ve bunların Izleme durumu **çevrimiçi**olarak gösterilir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![Traffic Manager profili uç noktası durumu](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="85c0f-468">Küresel kurumsal, Azure coğrafi dağıtım özelliklerini kullanır</span><span class="sxs-lookup"><span data-stu-id="85c0f-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="85c0f-469">Azure Traffic Manager ve coğrafi konuma özgü uç noktalar aracılığıyla veri trafiğini yönlendirmek, küresel kuruluşların bölgesel yönetmeliklere erişmesini ve verilerin uyumlu ve güvenli kalmasını sağlar ve bu da yerel ve uzak iş konumlarının başarısı için önemlidir.</span><span class="sxs-lookup"><span data-stu-id="85c0f-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="85c0f-470">Sonraki adımlar</span><span class="sxs-lookup"><span data-stu-id="85c0f-470">Next steps</span></span>

- <span data-ttu-id="85c0f-471">Azure bulut desenleri hakkında daha fazla bilgi edinmek için bkz. [bulut tasarım desenleri](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="85c0f-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
