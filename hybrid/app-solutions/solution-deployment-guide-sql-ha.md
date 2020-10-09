---
title: Azure ve Azure Stack hub 'a SQL Server 2016 kullanılabilirlik grubu dağıtma
description: SQL Server 2016 kullanılabilirlik grubunu Azure 'a ve Azure Stack hub 'a dağıtmayı öğrenin.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2c20d621247ec8e1278feb092586232cc08d5480
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852482"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="0955e-103">Azure ve Azure Stack hub 'a SQL Server 2016 kullanılabilirlik grubu dağıtma</span><span class="sxs-lookup"><span data-stu-id="0955e-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="0955e-104">Bu makalede, iki Azure Stack hub ortamında zaman uyumsuz olağanüstü durum kurtarma (DR) sitesiyle temel yüksek kullanılabilirliğe sahip (HA) SQL Server 2016 kurumsal kümesinin otomatik dağıtımı adım adım gösterilir.</span><span class="sxs-lookup"><span data-stu-id="0955e-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="0955e-105">SQL Server 2016 ve yüksek kullanılabilirlik hakkında daha fazla bilgi edinmek için bkz. [Always on kullanılabilirlik grupları: yüksek kullanılabilirlik ve olağanüstü durum kurtarma çözümü](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span><span class="sxs-lookup"><span data-stu-id="0955e-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="0955e-106">Bu çözümde, aşağıdakileri yapmak için bir örnek ortam oluşturacaksınız:</span><span class="sxs-lookup"><span data-stu-id="0955e-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="0955e-107">İki Azure Stack hub arasında bir dağıtımı düzenleyin.</span><span class="sxs-lookup"><span data-stu-id="0955e-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="0955e-108">Azure API profilleriyle ilgili bağımlılık sorunlarını en aza indirmek için Docker 'ı kullanın.</span><span class="sxs-lookup"><span data-stu-id="0955e-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="0955e-109">Olağanüstü durum kurtarma sitesiyle temel yüksek düzeyde kullanılabilir SQL Server 2016 kurumsal kümesi dağıtın.</span><span class="sxs-lookup"><span data-stu-id="0955e-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="0955e-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="0955e-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="0955e-111">Microsoft Azure Stack hub, Azure uzantısıdır.</span><span class="sxs-lookup"><span data-stu-id="0955e-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="0955e-112">Azure Stack hub, bulut bilgi işlemin çevikliğini ve yeniliklerini şirket içi ortamınıza getirerek, karma uygulamaları her yerde derleyip dağıtmanıza imkan tanıyan tek karma bulutu etkinleştirir.</span><span class="sxs-lookup"><span data-stu-id="0955e-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="0955e-113">[Karma uygulama tasarımı ile ilgili önemli noktalar](overview-app-design-considerations.md) , karma uygulamalar tasarlamak, dağıtmak ve çalıştırmak için yazılım kalitesinin (yerleştirme, ölçeklenebilirlik, kullanılabilirlik, dayanıklılık, yönetilebilirlik ve güvenlik) aynı şekilde gözden geçirmeleri inceler.</span><span class="sxs-lookup"><span data-stu-id="0955e-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="0955e-114">Tasarım konuları karma uygulama tasarımını iyileştirirken, üretim ortamlarındaki zorlukları en aza indirmeyle ilgili olarak size yardımcı olur.</span><span class="sxs-lookup"><span data-stu-id="0955e-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="0955e-115">2016 SQL Server için mimari</span><span class="sxs-lookup"><span data-stu-id="0955e-115">Architecture for SQL Server 2016</span></span>

![SQL Server 2016 SQL HA Azure Stack hub 'ı](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="0955e-117">SQL Server 2016 önkoşulları</span><span class="sxs-lookup"><span data-stu-id="0955e-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="0955e-118">İki bağlı Azure Stack hub tümleşik sistemi (Azure Stack hub).</span><span class="sxs-lookup"><span data-stu-id="0955e-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="0955e-119">Bu dağıtım Azure Stack Geliştirme Seti (ASDK) üzerinde çalışmıyor.</span><span class="sxs-lookup"><span data-stu-id="0955e-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="0955e-120">Azure Stack hub 'ı hakkında daha fazla bilgi için bkz. [Azure Stack genel bakış](https://azure.microsoft.com/overview/azure-stack/).</span><span class="sxs-lookup"><span data-stu-id="0955e-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="0955e-121">Her Azure Stack hub 'ında kiracı aboneliği.</span><span class="sxs-lookup"><span data-stu-id="0955e-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="0955e-122">**Her bir Azure Stack Hub için her abonelik KIMLIĞINI ve Azure Resource Manager uç noktasını bir yere unutmayın.**</span><span class="sxs-lookup"><span data-stu-id="0955e-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="0955e-123">Her bir Azure Stack hub 'ındaki kiracı aboneliğine yönelik izinlere sahip bir Azure Active Directory (Azure AD) hizmet sorumlusu.</span><span class="sxs-lookup"><span data-stu-id="0955e-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="0955e-124">Azure Stack hub 'Ları farklı Azure AD Kiracılarına karşı dağıtılırsa iki hizmet sorumlusu oluşturmanız gerekebilir.</span><span class="sxs-lookup"><span data-stu-id="0955e-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="0955e-125">Azure Stack Hub için hizmet sorumlusu oluşturma hakkında bilgi edinmek için bkz. [uygulamalara Azure Stack hub kaynaklarına erişim sağlamak için hizmet sorumluları oluşturma](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="0955e-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="0955e-126">**Her bir hizmet sorumlusunun uygulama KIMLIĞI, gizli anahtar ve kiracı adını (xxxxx.onmicrosoft.com) bir yere unutmayın.**</span><span class="sxs-lookup"><span data-stu-id="0955e-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="0955e-127">SQL Server 2016 Enterprise her bir Azure Stack hub 'ının Market 'e göre dağıtılmış.</span><span class="sxs-lookup"><span data-stu-id="0955e-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="0955e-128">Market dağıtımı hakkında daha fazla bilgi için bkz. [Market öğelerini Azure Stack hub 'ına indirme](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="0955e-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="0955e-129">**Kuruluşunuzun uygun SQL lisanslarına sahip olduğundan emin olun.**</span><span class="sxs-lookup"><span data-stu-id="0955e-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="0955e-130">Yerel makinenizde yüklü [Docker for Windows](https://docs.docker.com/docker-for-windows/) .</span><span class="sxs-lookup"><span data-stu-id="0955e-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="0955e-131">Docker görüntüsünü al</span><span class="sxs-lookup"><span data-stu-id="0955e-131">Get the Docker image</span></span>

<span data-ttu-id="0955e-132">Her dağıtım için Docker görüntüleri farklı Azure PowerShell sürümleri arasındaki bağımlılık sorunlarını ortadan kaldırır.</span><span class="sxs-lookup"><span data-stu-id="0955e-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="0955e-133">Docker for Windows Windows kapsayıcıları ' nı kullandığınızdan emin olun.</span><span class="sxs-lookup"><span data-stu-id="0955e-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="0955e-134">Dağıtım betikleri ile Docker kapsayıcısını almak için, yükseltilmiş bir komut isteminde aşağıdaki betiği çalıştırın.</span><span class="sxs-lookup"><span data-stu-id="0955e-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="0955e-135">Kullanılabilirlik grubunu dağıtma</span><span class="sxs-lookup"><span data-stu-id="0955e-135">Deploy the availability group</span></span>

1. <span data-ttu-id="0955e-136">Kapsayıcı görüntüsü başarıyla çekildikten sonra görüntüyü başlatın.</span><span class="sxs-lookup"><span data-stu-id="0955e-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="0955e-137">Kapsayıcı başlatıldıktan sonra, kapsayıcıda yükseltilmiş bir PowerShell terminali vermiş olursunuz.</span><span class="sxs-lookup"><span data-stu-id="0955e-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="0955e-138">Dağıtım betiğine ulaşmak için dizinleri değiştirin.</span><span class="sxs-lookup"><span data-stu-id="0955e-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="0955e-139">Dağıtımı çalıştırın.</span><span class="sxs-lookup"><span data-stu-id="0955e-139">Run the deployment.</span></span> <span data-ttu-id="0955e-140">Gereken yerlerde kimlik bilgileri ve kaynak adları sağlayın.</span><span class="sxs-lookup"><span data-stu-id="0955e-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="0955e-141">HA, HA kümesinin dağıtılacağı Azure Stack hub 'ına başvurur.</span><span class="sxs-lookup"><span data-stu-id="0955e-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="0955e-142">DR, DR kümesinin dağıtılacağı Azure Stack hub 'ına başvurur.</span><span class="sxs-lookup"><span data-stu-id="0955e-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
      -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
      -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
      -AADTenantName_HA "hatenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_HA "haresourcegroupname" `
      -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
      -AzureStackSubscriptionId_HA "haSubscriptionId" `
      -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
      -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
      -AADTenantName_DR "drtenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_DR "drresourcegroupname" `
      -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
      -AzureStackSubscriptionId_DR "drSubscriptionId"
      ```

4. <span data-ttu-id="0955e-143">`Y`NuGet sağlayıcısı 'nın yüklenmesine izin vermek için yazın ve bu, yüklenecek "2018-03-01-karma" MODÜLLERININ API profilini başlatabilir.</span><span class="sxs-lookup"><span data-stu-id="0955e-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="0955e-144">Kaynak dağıtımının tamamlanmasını bekleyin.</span><span class="sxs-lookup"><span data-stu-id="0955e-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="0955e-145">DR kaynak dağıtımı tamamlandıktan sonra kapsayıcıdan çıkın.</span><span class="sxs-lookup"><span data-stu-id="0955e-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="0955e-146">Her bir Azure Stack merkezi portalındaki kaynakları görüntüleyerek dağıtımı inceleyin.</span><span class="sxs-lookup"><span data-stu-id="0955e-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="0955e-147">HA ortamındaki SQL örneklerinden birine bağlanın ve SQL Server Management Studio (SSMS) aracılığıyla kullanılabilirlik grubunu inceleyin.</span><span class="sxs-lookup"><span data-stu-id="0955e-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="0955e-149">Sonraki adımlar</span><span class="sxs-lookup"><span data-stu-id="0955e-149">Next steps</span></span>

- <span data-ttu-id="0955e-150">Küme üzerinde el ile yük devretmek için SQL Server Management Studio kullanın.</span><span class="sxs-lookup"><span data-stu-id="0955e-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="0955e-151">Bkz. [Always on kullanılabilirlik grubunun Zorlanmış El Ile yük devretmesini gerçekleştirme (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span><span class="sxs-lookup"><span data-stu-id="0955e-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="0955e-152">Hibrit bulut uygulamaları hakkında daha fazla bilgi edinin.</span><span class="sxs-lookup"><span data-stu-id="0955e-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="0955e-153">Bkz [. karma bulut çözümleri.](/azure-stack/user/)</span><span class="sxs-lookup"><span data-stu-id="0955e-153">See [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="0955e-154">Kendi verilerinizi kullanın veya [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)'da bu örnekteki kodu değiştirin.</span><span class="sxs-lookup"><span data-stu-id="0955e-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>