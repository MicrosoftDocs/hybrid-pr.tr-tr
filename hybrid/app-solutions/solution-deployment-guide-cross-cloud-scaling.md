---
title: Azure 'da ve Azure Stack hub 'da çapraz bulutu ölçeklendirme bir uygulama dağıtma
description: Azure 'da ve Azure Stack hub 'da çapraz bulutu ölçeklendiran bir uygulamayı dağıtmayı öğrenin.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 740a8c0ec904fe8eb3f9744626bc9dd6655bdb52
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84912133"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="3337a-103">Azure ve Azure Stack hub kullanarak çapraz bulutu ölçeklendirme bir uygulama dağıtma</span><span class="sxs-lookup"><span data-stu-id="3337a-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3337a-104">Azure Stack merkezi barındırılan bir Web uygulamasından, Traffic Manager aracılığıyla otomatik ölçeklendirme ile Azure 'da barındırılan bir Web uygulamasına geçiş için el ile tetiklenen bir işlem sağlamak üzere bir çoklu bulut çözümü oluşturmayı öğrenin.</span><span class="sxs-lookup"><span data-stu-id="3337a-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="3337a-105">Bu işlem, yük altında esnek ve ölçeklenebilir bulut yardımcı programı sağlar.</span><span class="sxs-lookup"><span data-stu-id="3337a-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="3337a-106">Bu Düzenle, kiracınız uygulamanızı genel bulutta çalıştırmaya hazırlamayabilir.</span><span class="sxs-lookup"><span data-stu-id="3337a-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="3337a-107">Bununla birlikte, iş için şirket içi ortamlarında gereken kapasiteyi korumak, uygulama için talepte ani artışları karşılamak amacıyla ekonomik bir şekilde uygulanabilir olmayabilir.</span><span class="sxs-lookup"><span data-stu-id="3337a-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="3337a-108">Kiracınız, şirket içi çözümüyle ortak bulutun esneklik düzeyini kullanabilir.</span><span class="sxs-lookup"><span data-stu-id="3337a-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="3337a-109">Bu çözümde, aşağıdakileri yapmak için bir örnek ortam oluşturacaksınız:</span><span class="sxs-lookup"><span data-stu-id="3337a-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="3337a-110">Çok düğümlü bir Web uygulaması oluşturun.</span><span class="sxs-lookup"><span data-stu-id="3337a-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="3337a-111">Sürekli dağıtım (CD) işlemini yapılandırın ve yönetin.</span><span class="sxs-lookup"><span data-stu-id="3337a-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="3337a-112">Web uygulamasını Azure Stack hub 'a yayımlayın.</span><span class="sxs-lookup"><span data-stu-id="3337a-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="3337a-113">Bir yayın oluşturun.</span><span class="sxs-lookup"><span data-stu-id="3337a-113">Create a release.</span></span>
> - <span data-ttu-id="3337a-114">Dağıtımlarınızı izlemeyi ve izlemeyi öğrenin.</span><span class="sxs-lookup"><span data-stu-id="3337a-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="3337a-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="3337a-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="3337a-116">Microsoft Azure Stack hub, Azure uzantısıdır.</span><span class="sxs-lookup"><span data-stu-id="3337a-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="3337a-117">Azure Stack hub, bulut bilgi işlemin çevikliğini ve yeniliklerini şirket içi ortamınıza getirerek, karma uygulamaları her yerde derleyip dağıtmanıza imkan tanıyan tek karma bulutu etkinleştirir.</span><span class="sxs-lookup"><span data-stu-id="3337a-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="3337a-118">[Karma uygulama tasarımı ile ilgili önemli noktalar](overview-app-design-considerations.md) , karma uygulamalar tasarlamak, dağıtmak ve çalıştırmak için yazılım kalitesinin (yerleştirme, ölçeklenebilirlik, kullanılabilirlik, dayanıklılık, yönetilebilirlik ve güvenlik) aynı şekilde gözden geçirmeleri inceler.</span><span class="sxs-lookup"><span data-stu-id="3337a-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="3337a-119">Tasarım konuları karma uygulama tasarımını iyileştirirken, üretim ortamlarındaki zorlukları en aza indirmeyle ilgili olarak size yardımcı olur.</span><span class="sxs-lookup"><span data-stu-id="3337a-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="3337a-120">Ön koşullar</span><span class="sxs-lookup"><span data-stu-id="3337a-120">Prerequisites</span></span>

- <span data-ttu-id="3337a-121">Azure aboneliği.</span><span class="sxs-lookup"><span data-stu-id="3337a-121">Azure subscription.</span></span> <span data-ttu-id="3337a-122">Gerekirse, başlamadan önce [ücretsiz bir hesap](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) oluşturun.</span><span class="sxs-lookup"><span data-stu-id="3337a-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="3337a-123">Azure Stack hub tümleşik sistemi veya Azure Stack Geliştirme Seti dağıtımı (ASDK).</span><span class="sxs-lookup"><span data-stu-id="3337a-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="3337a-124">Azure Stack hub 'ı yüklemeyle ilgili yönergeler için bkz. [ASDK 'Yi yükleme](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="3337a-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span></span>
  - <span data-ttu-id="3337a-125">Bir ASDK dağıtım sonrası Otomasyon betiği için şuraya gidin:[https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="3337a-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="3337a-126">Bu yüklemenin tamamlanabilmesi için birkaç saat gerekebilir.</span><span class="sxs-lookup"><span data-stu-id="3337a-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="3337a-127">[App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS hizmetlerini Azure Stack hub 'a dağıtın.</span><span class="sxs-lookup"><span data-stu-id="3337a-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="3337a-128">Azure Stack hub ortamı içinde [planlar/teklifler oluşturun](/azure-stack/operator/service-plan-offer-subscription-overview.md) .</span><span class="sxs-lookup"><span data-stu-id="3337a-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="3337a-129">Azure Stack hub ortamı içinde [Kiracı aboneliği oluşturun](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) .</span><span class="sxs-lookup"><span data-stu-id="3337a-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="3337a-130">Kiracı aboneliği içinde bir Web uygulaması oluşturun.</span><span class="sxs-lookup"><span data-stu-id="3337a-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="3337a-131">Yeni Web uygulaması URL 'sini daha sonra kullanmak üzere unutmayın.</span><span class="sxs-lookup"><span data-stu-id="3337a-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="3337a-132">Kiracı aboneliği içinde Azure Pipelines sanal makineyi (VM) dağıtın.</span><span class="sxs-lookup"><span data-stu-id="3337a-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="3337a-133">.NET 3,5 ile Windows Server 2016 VM gereklidir.</span><span class="sxs-lookup"><span data-stu-id="3337a-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="3337a-134">Bu VM, Azure Stack hub 'ındaki kiracı aboneliğinde özel derleme aracısı olarak oluşturulacak.</span><span class="sxs-lookup"><span data-stu-id="3337a-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="3337a-135">[SQL 2017 Için Windows Server 2016 sanal makine görüntüsü](/azure-stack/operator/azure-stack-add-vm-image.md) Azure Stack hub marketi 'nde kullanılabilir.</span><span class="sxs-lookup"><span data-stu-id="3337a-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image.md) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="3337a-136">Bu görüntü kullanılamıyorsa, ortama eklendiğinden emin olmak için bir Azure Stack hub Işleciyle çalışın.</span><span class="sxs-lookup"><span data-stu-id="3337a-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="3337a-137">Sorunlar ve dikkat edilmesi gerekenler</span><span class="sxs-lookup"><span data-stu-id="3337a-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="3337a-138">Ölçeklenebilirlik</span><span class="sxs-lookup"><span data-stu-id="3337a-138">Scalability</span></span>

<span data-ttu-id="3337a-139">Platformlar arası ölçeklendirmenin anahtar bileşeni, tutarlı ve güvenilir hizmet sağlayan ortak ve şirket içi bulut altyapısı arasında anında ve isteğe bağlı ölçeklendirme sunma olanağıdır.</span><span class="sxs-lookup"><span data-stu-id="3337a-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="3337a-140">Kullanılabilirlik</span><span class="sxs-lookup"><span data-stu-id="3337a-140">Availability</span></span>

<span data-ttu-id="3337a-141">Yerel olarak dağıtılan uygulamaların şirket içi donanım yapılandırması ve yazılım dağıtımı aracılığıyla yüksek kullanılabilirlik için yapılandırıldığından emin olun.</span><span class="sxs-lookup"><span data-stu-id="3337a-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="3337a-142">Yönetilebilirlik</span><span class="sxs-lookup"><span data-stu-id="3337a-142">Manageability</span></span>

<span data-ttu-id="3337a-143">Platformlar arası çözüm, ortamlar arasında sorunsuz yönetim ve tanıdık arabirim sağlar.</span><span class="sxs-lookup"><span data-stu-id="3337a-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="3337a-144">Platformlar arası yönetim için PowerShell önerilir.</span><span class="sxs-lookup"><span data-stu-id="3337a-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="3337a-145">Platformlar arası ölçeklendirme</span><span class="sxs-lookup"><span data-stu-id="3337a-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="3337a-146">Özel etki alanı edinme ve DNS 'yi yapılandırma</span><span class="sxs-lookup"><span data-stu-id="3337a-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="3337a-147">Etki alanı için DNS bölge dosyasını güncelleştirin.</span><span class="sxs-lookup"><span data-stu-id="3337a-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="3337a-148">Azure AD, özel etki alanı adının sahipliğini doğrulayacaktır.</span><span class="sxs-lookup"><span data-stu-id="3337a-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="3337a-149">Azure 'da Azure/Office 365/dış DNS kayıtları için [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) kullanın veya DNS girişini [farklı bir DNS kaydedicisinde](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)ekleyin.</span><span class="sxs-lookup"><span data-stu-id="3337a-149">Use [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

1. <span data-ttu-id="3337a-150">Özel bir etki alanını ortak bir kayıt defteri ile kaydedin.</span><span class="sxs-lookup"><span data-stu-id="3337a-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="3337a-151">Etki alanına ilişkin etki alanı adı kayıt şirketinde oturum açın.</span><span class="sxs-lookup"><span data-stu-id="3337a-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="3337a-152">DNS güncelleştirmeleri yapmak için onaylanan yönetici gerekli olabilir.</span><span class="sxs-lookup"><span data-stu-id="3337a-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="3337a-153">Azure AD tarafından belirtilen DNS girişini ekleyerek etki alanı için DNS bölge dosyasını güncelleştirin.</span><span class="sxs-lookup"><span data-stu-id="3337a-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="3337a-154">(DNS girişi, e-posta yönlendirmeyi veya Web barındırma davranışlarını etkilemez.)</span><span class="sxs-lookup"><span data-stu-id="3337a-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="3337a-155">Azure Stack hub 'da varsayılan bir çok düğümlü Web uygulaması oluşturma</span><span class="sxs-lookup"><span data-stu-id="3337a-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="3337a-156">Azure ve Azure Stack hub 'a Web uygulamaları dağıtmak ve her iki bulutda yapılan değişiklikleri autopush için karma sürekli tümleştirme ve sürekli dağıtım (CI/CD) ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="3337a-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="3337a-157">Çalıştırmak için (Windows Server ve SQL) dağıtılmış uygun görüntülerle Azure Stack hub ve App Service dağıtımı gereklidir.</span><span class="sxs-lookup"><span data-stu-id="3337a-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="3337a-158">Daha fazla bilgi için [Azure Stack hub 'ında App Service dağıtmaya yönelik](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)App Service belge önkoşullarını gözden geçirin.</span><span class="sxs-lookup"><span data-stu-id="3337a-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="3337a-159">Azure Repos kod ekleme</span><span class="sxs-lookup"><span data-stu-id="3337a-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="3337a-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="3337a-160">Azure Repos</span></span>

1. <span data-ttu-id="3337a-161">Azure Repos üzerinde proje oluşturma haklarına sahip bir hesapla Azure Repos için oturum açın.</span><span class="sxs-lookup"><span data-stu-id="3337a-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="3337a-162">Karma CI/CD, hem uygulama kodu hem de altyapı kodu için uygulanabilir.</span><span class="sxs-lookup"><span data-stu-id="3337a-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="3337a-163">Hem özel hem de barındırılan bulut geliştirmesi için [Azure Resource Manager şablonları](https://azure.microsoft.com/resources/templates/) kullanın.</span><span class="sxs-lookup"><span data-stu-id="3337a-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Azure Repos bir projeye bağlanma](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="3337a-165">Varsayılan Web uygulamasını oluşturarak ve açarak **depoyu kopyalayın** .</span><span class="sxs-lookup"><span data-stu-id="3337a-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Azure Web App 'te Depo kopyalama](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="3337a-167">Her iki bulutta da uygulama hizmetleri için kendi kendine içerilen Web uygulaması dağıtımı oluşturma</span><span class="sxs-lookup"><span data-stu-id="3337a-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="3337a-168">**WebApplication. csproj** dosyasını düzenleyin.</span><span class="sxs-lookup"><span data-stu-id="3337a-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="3337a-169">Seçin `Runtimeidentifier` ve ekleyin `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="3337a-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="3337a-170">(Bkz. [kendi içinde dağıtım](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) belgeleri.)</span><span class="sxs-lookup"><span data-stu-id="3337a-170">(See [Self-contained deployment](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Web uygulaması proje dosyasını Düzenle](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="3337a-172">Takım Gezgini kullanarak Azure Repos kodu iade edin.</span><span class="sxs-lookup"><span data-stu-id="3337a-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="3337a-173">Uygulama kodunun Azure Repos işaretli olduğunu doğrulayın.</span><span class="sxs-lookup"><span data-stu-id="3337a-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="3337a-174">Derleme tanımı oluşturma</span><span class="sxs-lookup"><span data-stu-id="3337a-174">Create the build definition</span></span>

1. <span data-ttu-id="3337a-175">Derleme tanımları oluşturma özelliğini onaylamak için Azure Pipelines ' de oturum açın.</span><span class="sxs-lookup"><span data-stu-id="3337a-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="3337a-176">Add **-r win10-x64** kodu.</span><span class="sxs-lookup"><span data-stu-id="3337a-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="3337a-177">Bu ek, .NET Core ile bağımsız bir dağıtımı tetiklemek için gereklidir.</span><span class="sxs-lookup"><span data-stu-id="3337a-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Web uygulamasına kod ekleme](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="3337a-179">Derlemeyi çalıştırın.</span><span class="sxs-lookup"><span data-stu-id="3337a-179">Run the build.</span></span> <span data-ttu-id="3337a-180">[Kendi içinde bulunan dağıtım oluşturma](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) Işlemi, Azure 'da ve Azure Stack hub 'da çalışan yapıtları yayımlayacak.</span><span class="sxs-lookup"><span data-stu-id="3337a-180">The [self-contained deployment build](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="3337a-181">Azure barındırılan Aracısı kullanma</span><span class="sxs-lookup"><span data-stu-id="3337a-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="3337a-182">Azure Pipelines içinde barındırılan bir yapı aracısının kullanılması, Web uygulamaları oluşturmak ve dağıtmak için kullanışlı bir seçenektir.</span><span class="sxs-lookup"><span data-stu-id="3337a-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="3337a-183">Bakım ve yükseltmeler Microsoft Azure tarafından otomatik olarak gerçekleştirilir ve sürekli ve kesintisiz bir geliştirme döngüsünü etkinleştirir.</span><span class="sxs-lookup"><span data-stu-id="3337a-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="3337a-184">CD işlemini yönetme ve yapılandırma</span><span class="sxs-lookup"><span data-stu-id="3337a-184">Manage and configure the CD process</span></span>

<span data-ttu-id="3337a-185">Azure Pipelines ve Azure DevOps Services, yayınlar için geliştirme, hazırlık, QA ve üretim ortamları gibi birden çok ortama yüksek düzeyde yapılandırılabilir ve yönetilebilir bir işlem hattı sağlar; belirli aşamalarda onay gerektirme de dahil.</span><span class="sxs-lookup"><span data-stu-id="3337a-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="3337a-186">Yayın tanımı oluştur</span><span class="sxs-lookup"><span data-stu-id="3337a-186">Create release definition</span></span>

1. <span data-ttu-id="3337a-187">Azure DevOps Services **derleme ve yayınlama** bölümündeki **yayınlar** sekmesinin altına yeni bir yayın eklemek için **artı** düğmesini seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Yayın tanımı oluşturma](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="3337a-189">Azure App Service Dağıtım şablonunu uygulayın.</span><span class="sxs-lookup"><span data-stu-id="3337a-189">Apply the Azure App Service Deployment template.</span></span>

   ![Azure App Service dağıtım şablonu Uygula](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="3337a-191">**Yapıt Ekle**' nin altında, Azure Cloud Build uygulaması için yapıt ekleyin.</span><span class="sxs-lookup"><span data-stu-id="3337a-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Azure Cloud Build 'e yapıt ekleme](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="3337a-193">İşlem hattı sekmesi altında, ortamın **aşamasını, görev** bağlantısını seçin ve Azure bulut ortamı değerlerini ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="3337a-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Azure bulut ortamı değerlerini ayarlama](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="3337a-195">**Ortam adını** ayarlayın ve Azure bulut uç noktası için **Azure aboneliğini** seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Azure bulut uç noktası için Azure aboneliğini seçin](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="3337a-197">**App Service adı**altında, gerekli Azure App Service adını ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="3337a-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Azure App Service adını ayarla](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="3337a-199">Azure bulut barındırılan ortamı için **Aracı kuyruğu** altında "barındırılan VS2017" yazın.</span><span class="sxs-lookup"><span data-stu-id="3337a-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Azure bulut barındırılan ortamı için aracı kuyruğunu ayarla](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="3337a-201">Azure App Service dağıt menüsünde, ortam için geçerli **paketi veya klasörü** seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="3337a-202">**Klasör konumuna** **Tamam ' ı** seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-202">Select **OK** to **folder location**.</span></span>
  
      ![Azure App Service ortamı için paket veya klasör seçin](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Azure App Service ortamı için paket veya klasör seçin](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="3337a-205">Tüm değişiklikleri kaydedin ve **yayın ardışık düzenine**geri dönün.</span><span class="sxs-lookup"><span data-stu-id="3337a-205">Save all changes and go back to **release pipeline**.</span></span>

    ![Değişiklikleri yayın ardışık düzeninde Kaydet](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="3337a-207">Azure Stack Hub uygulaması için yapıyı seçerek yeni bir yapıt ekleyin.</span><span class="sxs-lookup"><span data-stu-id="3337a-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Azure Stack Hub uygulaması için yeni yapıt ekleme](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="3337a-209">Azure App Service dağıtımını uygulayarak bir ortam daha ekleyin.</span><span class="sxs-lookup"><span data-stu-id="3337a-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Azure App Service dağıtımına ortam ekleme](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="3337a-211">"Azure Stack" adlı yeni ortamı adlandırın.</span><span class="sxs-lookup"><span data-stu-id="3337a-211">Name the new environment "Azure Stack".</span></span>

    ![Azure App Service dağıtımında ad ortamı](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="3337a-213">**Görev** sekmesinde Azure Stack ortamını bulun.</span><span class="sxs-lookup"><span data-stu-id="3337a-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Azure Stack ortamı](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="3337a-215">Azure Stack uç noktası için aboneliği seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Azure Stack uç noktası için aboneliği seçin](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="3337a-217">Azure Stack Web uygulaması adını App Service adı olarak ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="3337a-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="3337a-218">![Azure Stack Web uygulaması adını ayarla](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="3337a-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="3337a-219">Azure Stack aracısını seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-219">Select the Azure Stack agent.</span></span>

    ![Azure Stack aracısını seçin](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="3337a-221">Dağıtım Azure App Service bölümünde, ortam için geçerli **paketi veya klasörü** seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="3337a-222">Klasör konumuna **Tamam ' ı** seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-222">Select **OK** to folder location.</span></span>

    ![Azure App Service dağıtımı için klasör seçin](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Azure App Service dağıtımı için klasör seçin](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="3337a-225">Değişken sekmesi altında adlı bir değişken ekleyin `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , değerini **doğru**olarak ayarlayın ve Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3337a-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Azure Uygulama dağıtımına değişken ekleme](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="3337a-227">Her iki yapıt içinde **sürekli** dağıtım tetikleme simgesini seçin ve **devam** eden dağıtım tetikleyicisini etkinleştirin.</span><span class="sxs-lookup"><span data-stu-id="3337a-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Sürekli dağıtım tetikleyicisi seçin](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="3337a-229">Azure Stack ortamında **dağıtım öncesi** koşullar simgesini seçin ve tetikleyiciyi **yayından sonra** olarak ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="3337a-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![Dağıtım öncesi koşulları seçin](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="3337a-231">Tüm değişiklikleri kaydedin.</span><span class="sxs-lookup"><span data-stu-id="3337a-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="3337a-232">Görevler için bazı ayarlar, bir şablondan bir yayın tanımı oluşturulurken otomatik olarak [ortam değişkenleri](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) olarak tanımlanabilir.</span><span class="sxs-lookup"><span data-stu-id="3337a-232">Some settings for the tasks may have been automatically defined as [environment variables](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="3337a-233">Bu ayarlar görev ayarlarında değiştirilemez; Bunun yerine, bu ayarları düzenlemek için üst ortam öğesinin seçilmesi gerekir.</span><span class="sxs-lookup"><span data-stu-id="3337a-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="3337a-234">Visual Studio aracılığıyla Azure Stack hub 'a yayımlama</span><span class="sxs-lookup"><span data-stu-id="3337a-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="3337a-235">Azure DevOps Services derlemesi, uç noktalar oluşturarak Azure hizmet uygulamalarını Azure Stack hub 'ına dağıtabilir.</span><span class="sxs-lookup"><span data-stu-id="3337a-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="3337a-236">Azure Pipelines, Azure Stack hub 'ına bağlanan yapı aracısına bağlanır.</span><span class="sxs-lookup"><span data-stu-id="3337a-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="3337a-237">Azure DevOps Services oturum açın ve uygulama ayarları sayfasına gidin.</span><span class="sxs-lookup"><span data-stu-id="3337a-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="3337a-238">**Ayarlar**' da **güvenlik**' i seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="3337a-239">**VSTS grupları**' nda **Endpoint Creators**' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="3337a-240">**Üyeler** sekmesinde **Ekle**' yi seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="3337a-241">Kullanıcı **ve Grup Ekle**' de, bir Kullanıcı adı girin ve bu kullanıcıyı Kullanıcı listesinden seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="3337a-242">**Değişiklikleri kaydet**'i seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="3337a-243">**VSTS grupları** listesinde **Endpoint Administrators**' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="3337a-244">**Üyeler** sekmesinde **Ekle**' yi seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="3337a-245">Kullanıcı **ve Grup Ekle**' de, bir Kullanıcı adı girin ve bu kullanıcıyı Kullanıcı listesinden seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="3337a-246">**Değişiklikleri kaydet**'i seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-246">Select **Save changes**.</span></span>

<span data-ttu-id="3337a-247">Artık uç nokta bilgileri mevcut olduğuna göre, Azure Stack hub bağlantısına Azure Pipelines kullanıma hazırdır.</span><span class="sxs-lookup"><span data-stu-id="3337a-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="3337a-248">Azure Stack hub 'daki yapı Aracısı Azure Pipelines yönergeleri alır ve aracı Azure Stack hub 'ı ile iletişim için uç nokta bilgileri alır.</span><span class="sxs-lookup"><span data-stu-id="3337a-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="3337a-249">Uygulama derlemesini geliştirme</span><span class="sxs-lookup"><span data-stu-id="3337a-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="3337a-250">Çalıştırmak için (Windows Server ve SQL) dağıtılmış uygun görüntülerle Azure Stack hub ve App Service dağıtımı gereklidir.</span><span class="sxs-lookup"><span data-stu-id="3337a-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="3337a-251">Daha fazla bilgi için, [Azure Stack hub 'ında App Service dağıtmaya yönelik önkoşullar](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)bölümüne bakın.</span><span class="sxs-lookup"><span data-stu-id="3337a-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

<span data-ttu-id="3337a-252">Her iki buluta dağıtmak için Azure Repos Web uygulaması kodu gibi [Azure Resource Manager şablonları](https://azure.microsoft.com/resources/templates/) kullanın.</span><span class="sxs-lookup"><span data-stu-id="3337a-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="3337a-253">Azure Repos projesine kod ekleme</span><span class="sxs-lookup"><span data-stu-id="3337a-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="3337a-254">Azure Stack hub 'ında proje oluşturma haklarına sahip bir hesapla Azure Repos için oturum açın.</span><span class="sxs-lookup"><span data-stu-id="3337a-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="3337a-255">Varsayılan Web uygulamasını oluşturarak ve açarak **depoyu kopyalayın** .</span><span class="sxs-lookup"><span data-stu-id="3337a-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="3337a-256">Her iki bulutta da uygulama hizmetleri için kendi kendine içerilen Web uygulaması dağıtımı oluşturma</span><span class="sxs-lookup"><span data-stu-id="3337a-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="3337a-257">**WebApplication. csproj** dosyasını düzenleyin: seçin `Runtimeidentifier` ve ardından ekleyin `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="3337a-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="3337a-258">Daha fazla bilgi için, bkz. [kendi kendine kapsanan dağıtım](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) belgeleri.</span><span class="sxs-lookup"><span data-stu-id="3337a-258">For more information, see [Self-contained deployment](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="3337a-259">Kodu Azure Repos olarak denetlemek için Takım Gezgini kullanın.</span><span class="sxs-lookup"><span data-stu-id="3337a-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="3337a-260">Uygulama kodunun Azure Repos işaretli olduğunu doğrulayın.</span><span class="sxs-lookup"><span data-stu-id="3337a-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="3337a-261">Derleme tanımı oluşturma</span><span class="sxs-lookup"><span data-stu-id="3337a-261">Create the build definition</span></span>

1. <span data-ttu-id="3337a-262">Derleme tanımı oluşturabileceğiniz bir hesapla Azure Pipelines için oturum açın.</span><span class="sxs-lookup"><span data-stu-id="3337a-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="3337a-263">Projenin **Build Web Application** sayfasına gidin.</span><span class="sxs-lookup"><span data-stu-id="3337a-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="3337a-264">**Bağımsız değişkenler**içinde **-r win10-x64** kodu ekleyin.</span><span class="sxs-lookup"><span data-stu-id="3337a-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="3337a-265">Bu ek, .NET Core ile bağımsız bir dağıtımı tetiklemek için gereklidir.</span><span class="sxs-lookup"><span data-stu-id="3337a-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="3337a-266">Derlemeyi çalıştırın.</span><span class="sxs-lookup"><span data-stu-id="3337a-266">Run the build.</span></span> <span data-ttu-id="3337a-267">[Kendi içinde çalışan dağıtım oluşturma](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) Işlemi, Azure 'da ve Azure Stack hub 'da çalışabilecek yapıtları yayımlar.</span><span class="sxs-lookup"><span data-stu-id="3337a-267">The [self-contained deployment build](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="3337a-268">Azure 'da barındırılan derleme Aracısı kullanma</span><span class="sxs-lookup"><span data-stu-id="3337a-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="3337a-269">Azure Pipelines içinde barındırılan bir yapı aracısının kullanılması, Web uygulamaları oluşturmak ve dağıtmak için kullanışlı bir seçenektir.</span><span class="sxs-lookup"><span data-stu-id="3337a-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="3337a-270">Bakım ve yükseltmeler Microsoft Azure tarafından otomatik olarak gerçekleştirilir ve sürekli ve kesintisiz bir geliştirme döngüsünü etkinleştirir.</span><span class="sxs-lookup"><span data-stu-id="3337a-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="3337a-271">Sürekli dağıtım (CD) işlemini yapılandırma</span><span class="sxs-lookup"><span data-stu-id="3337a-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="3337a-272">Azure Pipelines ve Azure DevOps Services, yayınlar için geliştirme, hazırlama, kalite güvencesi (QA) ve üretim gibi birden çok ortama yönelik yüksek düzeyde yapılandırılabilir ve yönetilebilir bir işlem hattı sağlar.</span><span class="sxs-lookup"><span data-stu-id="3337a-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="3337a-273">Bu işlem, uygulama yaşam döngüsünün belirli aşamalarındaki onayları gerektirmeyi içerebilir.</span><span class="sxs-lookup"><span data-stu-id="3337a-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="3337a-274">Yayın tanımı oluştur</span><span class="sxs-lookup"><span data-stu-id="3337a-274">Create release definition</span></span>

<span data-ttu-id="3337a-275">Yayın tanımı oluşturmak, uygulama oluşturma işlemindeki son adımdır.</span><span class="sxs-lookup"><span data-stu-id="3337a-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="3337a-276">Bu yayın tanımı bir yayın oluşturmak ve bir derlemeyi dağıtmak için kullanılır.</span><span class="sxs-lookup"><span data-stu-id="3337a-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="3337a-277">Azure Pipelines oturum açın ve proje için **derleme ve yayın** bölümüne gidin.</span><span class="sxs-lookup"><span data-stu-id="3337a-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="3337a-278">**Yayınlar** sekmesinde **[+]** öğesini seçin ve ardından **yayın tanımı oluştur**' u seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="3337a-279">**Şablon seçin**sayfasında, **Azure App Service dağıtım**' ı seçin ve ardından **Uygula**' yı seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="3337a-280">**Yapıt Ekle**sayfasında, **kaynaktan (derleme tanımı)** Azure Cloud Build uygulamasını seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="3337a-281">İşlem **hattı** sekmesinde, **ortam görevlerini görüntülemek**için **1 aşama**, **1 görev** bağlantısını seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="3337a-282">**Görevler** sekmesinde, **ortam adı** olarak Azure yazın ve **Azure aboneliği** listesinden AZURECYÜKSEK Traders-Web EP ' yi seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="3337a-283">Sonraki ekran yakalamadaki **Azure App Service adını**girin `northwindtraders` .</span><span class="sxs-lookup"><span data-stu-id="3337a-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="3337a-284">Aracı aşaması için, **Aracı sırası** LISTESINDEN **barındırılan VS2017** öğesini seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="3337a-285">**Azure App Service dağıt**' da, ortam Için geçerli **paketi veya klasörü** seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="3337a-286">**Dosya veya klasör seç**' de konuma **Tamam** ' **Location**ı seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="3337a-287">Tüm değişiklikleri kaydedin ve **ardışık düzene**geri dönün.</span><span class="sxs-lookup"><span data-stu-id="3337a-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="3337a-288">İşlem **hattı** sekmesinde **yapıt Ekle**' yi seçin ve **kaynak (derleme tanımı)** listesinden **northwindcloud Traders-vessel** ' i seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="3337a-289">**Şablon seçin**sayfasında başka bir ortam ekleyin.</span><span class="sxs-lookup"><span data-stu-id="3337a-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="3337a-290">**Azure App Service dağıtımını** seçin ve ardından **Uygula**' yı seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="3337a-291">`Azure Stack Hub` **Ortam adı**olarak girin.</span><span class="sxs-lookup"><span data-stu-id="3337a-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="3337a-292">**Görevler** sekmesinde Azure Stack hub 'ı bulun ve seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="3337a-293">**Azure aboneliği** listesinden Azure Stack hub uç noktası Için **Azurestack Traders-vessel EP** ' yi seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="3337a-294">**Uygulama hizmeti adı**olarak Azure Stack Hub web uygulaması adını girin.</span><span class="sxs-lookup"><span data-stu-id="3337a-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="3337a-295">**Aracı seçimi**altında, **Aracı sırası** listesinden **Azurestack-b Douglas FI** ' yi seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="3337a-296">**Dağıtım Azure App Service**için, ortam Için geçerli **paketi veya klasörü** seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="3337a-297">**Dosya veya klasör seçin**sayfasında, klasör **konumu**için **Tamam** ' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="3337a-298">**Değişken** sekmesinde adlı değişkeni bulun `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` .</span><span class="sxs-lookup"><span data-stu-id="3337a-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="3337a-299">Değişken değerini **true**olarak ayarlayın ve kapsamını **Azure Stack hub**olarak ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="3337a-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="3337a-300">İşlem **hattı** sekmesinde, Northwindcloud Traders-Web yapıtı için **sürekli dağıtım tetikleme** simgesini seçin ve **sürekli dağıtım tetikleyicisini** **etkin**olarak ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="3337a-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="3337a-301">**Northwindcloud Traders-vessel** yapıtı için aynı şeyi yapın.</span><span class="sxs-lookup"><span data-stu-id="3337a-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="3337a-302">Azure Stack hub ortamı için **dağıtım öncesi koşullar** simgesini, **yayından sonra**olarak ayarla ' yı seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="3337a-303">Tüm değişiklikleri kaydedin.</span><span class="sxs-lookup"><span data-stu-id="3337a-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="3337a-304">Yayın görevlerine yönelik bazı ayarlar, bir şablondan bir yayın tanımı oluşturulurken otomatik olarak [ortam değişkenleri](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) olarak tanımlanır.</span><span class="sxs-lookup"><span data-stu-id="3337a-304">Some settings for release tasks are automatically defined as [environment variables](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="3337a-305">Bu ayarlar görev ayarlarından değiştirilemez, ancak üst ortam öğelerinde değiştirilebilir.</span><span class="sxs-lookup"><span data-stu-id="3337a-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="3337a-306">Yayın oluştur</span><span class="sxs-lookup"><span data-stu-id="3337a-306">Create a release</span></span>

1. <span data-ttu-id="3337a-307">İşlem **hattı** sekmesinde, **yayın** listesini açın ve **yayın oluştur**' u seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="3337a-308">Yayın için bir açıklama girin, doğru yapıtların seçili olup olmadığını denetleyin ve ardından **Oluştur**' u seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="3337a-309">Birkaç dakika sonra, yeni yayının oluşturulduğunu ve yayın adının bir bağlantı olarak görüntülendiğini gösteren bir başlık görünür.</span><span class="sxs-lookup"><span data-stu-id="3337a-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="3337a-310">Yayın Özeti sayfasını görmek için bağlantıyı seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="3337a-311">Yayın Özeti sayfası, sürüm hakkındaki ayrıntıları gösterir.</span><span class="sxs-lookup"><span data-stu-id="3337a-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="3337a-312">"Release-2" için, **ortamlar** bölümünde, Azure için **DAĞıTıM durumu** "sürüyor" olarak ve Azure Stack hub 'ın durumu "başarılı" olarak gösterilir.</span><span class="sxs-lookup"><span data-stu-id="3337a-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="3337a-313">Azure ortamının dağıtım durumu "başarılı" olarak değiştiğinde, yayının onaya hazırlandığını gösteren bir başlık görüntülenir.</span><span class="sxs-lookup"><span data-stu-id="3337a-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="3337a-314">Bir dağıtım beklendiğinde veya başarısız olduysa, mavi **(i)** bilgi simgesi gösterilir.</span><span class="sxs-lookup"><span data-stu-id="3337a-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="3337a-315">Gecikme veya başarısızlık nedenini içeren bir açılır pencere görmek için simgenin üzerine gelin.</span><span class="sxs-lookup"><span data-stu-id="3337a-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="3337a-316">Yayın listesi gibi diğer görünümler de onay beklendiğini belirten bir simge görüntüler.</span><span class="sxs-lookup"><span data-stu-id="3337a-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="3337a-317">Bu simgenin açılır penceresi, ortam adı ve dağıtımla ilgili daha fazla ayrıntı gösterir.</span><span class="sxs-lookup"><span data-stu-id="3337a-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="3337a-318">Yöneticiler, yayınların genel ilerleme durumunu görmek ve hangi sürümlerin onay beklediğini görmek için çok kolay.</span><span class="sxs-lookup"><span data-stu-id="3337a-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="3337a-319">Dağıtımları izleme ve izleme</span><span class="sxs-lookup"><span data-stu-id="3337a-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="3337a-320">2. **Sürüm** Özeti sayfasında **Günlükler**' i seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="3337a-321">Dağıtım sırasında bu sayfada, aracıdan canlı günlük görüntülenir.</span><span class="sxs-lookup"><span data-stu-id="3337a-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="3337a-322">Sol bölmede her bir ortam için dağıtımdaki her bir işlemin durumu gösterilir.</span><span class="sxs-lookup"><span data-stu-id="3337a-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="3337a-323">Dağıtım öncesi veya dağıtım sonrası onayı için **eylem** sütununda kişi simgesini seçerek dağıtımı ve sağladıkları iletiyi kimlerin onayladığını (veya reddettiğini) görüntüleyin.</span><span class="sxs-lookup"><span data-stu-id="3337a-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="3337a-324">Dağıtım bittikten sonra, tüm günlük dosyası sağ bölmede görüntülenir.</span><span class="sxs-lookup"><span data-stu-id="3337a-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="3337a-325">**Başlatma işi**gibi tek bir adım için günlük dosyasını görmek üzere sol bölmedeki herhangi bir **adımı** seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="3337a-326">Tek tek günlükleri görme özelliği, genel dağıtımın parçalarını izlemeyi ve hata ayıklamayı kolaylaştırır.</span><span class="sxs-lookup"><span data-stu-id="3337a-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="3337a-327">Bir adım için günlük dosyasını **kaydedin** veya **tüm günlükleri zip olarak indirin**.</span><span class="sxs-lookup"><span data-stu-id="3337a-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="3337a-328">Sürümle ilgili genel bilgileri görmek için **Özet** sekmesini açın.</span><span class="sxs-lookup"><span data-stu-id="3337a-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="3337a-329">Bu görünüm, derleme hakkındaki ayrıntıları, dağıtıldığı ortamları, dağıtım durumunu ve sürümle ilgili diğer bilgileri gösterir.</span><span class="sxs-lookup"><span data-stu-id="3337a-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="3337a-330">Belirli bir ortama yönelik mevcut ve bekleyen dağıtımlar hakkındaki bilgileri görmek için bir ortam bağlantısı (**Azure** veya **Azure Stack hub**) seçin.</span><span class="sxs-lookup"><span data-stu-id="3337a-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="3337a-331">Aynı yapılandırmanın her iki ortama de dağıtıldığını denetlemek için bu görünümleri hızlı bir şekilde kullanın.</span><span class="sxs-lookup"><span data-stu-id="3337a-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="3337a-332">**Dağıtılan üretim uygulamasını** bir tarayıcıda açın.</span><span class="sxs-lookup"><span data-stu-id="3337a-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="3337a-333">Örneğin, Azure App Services Web sitesi için URL 'YI açın `https://[your-app-name\].azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="3337a-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="3337a-334">Azure ve Azure Stack hub 'ın tümleştirilmesi, ölçeklenebilir bir çapraz bulut çözümü sağlar</span><span class="sxs-lookup"><span data-stu-id="3337a-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="3337a-335">Esnek ve sağlam bir çoklu bulut hizmeti, veri güvenliği, yedekleme ve artıklık, tutarlı ve hızlı kullanılabilirlik, ölçeklenebilir depolama ve dağıtım ve coğrafi olarak uyumlu yönlendirme sağlar.</span><span class="sxs-lookup"><span data-stu-id="3337a-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="3337a-336">El ile tetiklenen bu işlem, barındırılan Web uygulamaları ve önemli verilerin hemen kullanılabilirliği arasında güvenilir ve verimli yük geçiş sağlar.</span><span class="sxs-lookup"><span data-stu-id="3337a-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="3337a-337">Sonraki adımlar</span><span class="sxs-lookup"><span data-stu-id="3337a-337">Next steps</span></span>

- <span data-ttu-id="3337a-338">Azure bulut desenleri hakkında daha fazla bilgi edinmek için bkz. [bulut tasarım desenleri](https://docs.microsoft.com/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="3337a-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
