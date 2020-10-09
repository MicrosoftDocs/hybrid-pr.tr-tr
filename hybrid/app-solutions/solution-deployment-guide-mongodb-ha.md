---
title: Yüksek oranda kullanılabilir bir MongoDB çözümünü Azure 'a ve Azure Stack hub 'a dağıtma
description: Yüksek oranda kullanılabilir bir MongoDB çözümünü Azure 'a ve Azure Stack hub 'a dağıtmayı öğrenin
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: def9abaa2a7231648f11453f66119399be015a4d
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852516"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a><span data-ttu-id="a8ac3-103">Yüksek oranda kullanılabilir bir MongoDB çözümünü Azure 'a ve Azure Stack hub 'a dağıtma</span><span class="sxs-lookup"><span data-stu-id="a8ac3-103">Deploy a highly available MongoDB solution to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="a8ac3-104">Bu makalede, iki Azure Stack hub ortamında olağanüstü durum kurtarma (DR) sitesiyle temel yüksek kullanılabilirliğe sahip bir MongoDB kümesinin otomatik dağıtımı adım adım gösterilir.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="a8ac3-105">MongoDB ve yüksek kullanılabilirlik hakkında daha fazla bilgi edinmek için bkz. [çoğaltma kümesi üyeleri](https://docs.mongodb.com/manual/core/replica-set-members/).</span><span class="sxs-lookup"><span data-stu-id="a8ac3-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="a8ac3-106">Bu çözümde şu şekilde bir örnek ortam oluşturacaksınız:</span><span class="sxs-lookup"><span data-stu-id="a8ac3-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="a8ac3-107">İki Azure Stack hub arasında bir dağıtımı düzenleyin.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="a8ac3-108">Azure API profilleriyle ilgili bağımlılık sorunlarını en aza indirmek için Docker 'ı kullanın.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="a8ac3-109">Yüksek oranda kullanılabilir bir MongoDB kümesini olağanüstü durum kurtarma sitesiyle dağıtın.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="a8ac3-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="a8ac3-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="a8ac3-111">Microsoft Azure Stack hub, Azure uzantısıdır.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="a8ac3-112">Azure Stack hub, bulut bilgi işlemin çevikliğini ve yeniliklerini şirket içi ortamınıza getirerek, karma uygulamaları her yerde derleyip dağıtmanıza imkan tanıyan tek karma bulutu etkinleştirir.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="a8ac3-113">[Karma uygulama tasarımı ile ilgili önemli noktalar](overview-app-design-considerations.md) , karma uygulamalar tasarlamak, dağıtmak ve çalıştırmak için yazılım kalitesinin (yerleştirme, ölçeklenebilirlik, kullanılabilirlik, dayanıklılık, yönetilebilirlik ve güvenlik) aynı şekilde gözden geçirmeleri inceler.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="a8ac3-114">Tasarım konuları karma uygulama tasarımını iyileştirirken, üretim ortamlarındaki zorlukları en aza indirmeyle ilgili olarak size yardımcı olur.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="a8ac3-115">Azure Stack hub ile MongoDB mimarisi</span><span class="sxs-lookup"><span data-stu-id="a8ac3-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![Azure Stack hub 'da yüksek oranda kullanılabilir MongoDB mimarisi](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="a8ac3-117">Azure Stack hub ile MongoDB önkoşulları</span><span class="sxs-lookup"><span data-stu-id="a8ac3-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="a8ac3-118">İki bağlı Azure Stack hub tümleşik sistemi (Azure Stack hub).</span><span class="sxs-lookup"><span data-stu-id="a8ac3-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="a8ac3-119">Bu dağıtım Azure Stack Geliştirme Seti (ASDK) üzerinde çalışmıyor.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="a8ac3-120">Azure Stack hub 'ı hakkında daha fazla bilgi edinmek için bkz. [Azure Stack hub nedir?](https://azure.microsoft.com/products/azure-stack/hub/)</span><span class="sxs-lookup"><span data-stu-id="a8ac3-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="a8ac3-121">Her Azure Stack hub 'ında kiracı aboneliği.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="a8ac3-122">**Her bir Azure Stack Hub için her abonelik KIMLIĞINI ve Azure Resource Manager uç noktasını bir yere unutmayın.**</span><span class="sxs-lookup"><span data-stu-id="a8ac3-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="a8ac3-123">Her bir Azure Stack hub 'ındaki kiracı aboneliğine yönelik izinlere sahip bir Azure Active Directory (Azure AD) hizmet sorumlusu.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="a8ac3-124">Azure Stack hub 'Ları farklı Azure AD Kiracılarına karşı dağıtılırsa iki hizmet sorumlusu oluşturmanız gerekebilir.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="a8ac3-125">Azure Stack Hub için hizmet sorumlusu oluşturma hakkında bilgi edinmek için bkz. [Azure Stack hub kaynaklarına erişmek için uygulama kimliği kullanma](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="a8ac3-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="a8ac3-126">**Her bir hizmet sorumlusunun uygulama KIMLIĞI, gizli anahtar ve kiracı adını (xxxxx.onmicrosoft.com) bir yere unutmayın.**</span><span class="sxs-lookup"><span data-stu-id="a8ac3-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="a8ac3-127">Ubuntu 16,04 her bir Azure Stack hub 'ının Market 'e göre dağıtılmış.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="a8ac3-128">Market dağıtımı hakkında daha fazla bilgi için bkz. [Market öğelerini Azure Stack hub 'ına indirme](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="a8ac3-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="a8ac3-129">Yerel makinenizde yüklü [Docker for Windows](https://docs.docker.com/docker-for-windows/) .</span><span class="sxs-lookup"><span data-stu-id="a8ac3-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="a8ac3-130">Docker görüntüsünü al</span><span class="sxs-lookup"><span data-stu-id="a8ac3-130">Get the Docker image</span></span>

<span data-ttu-id="a8ac3-131">Her dağıtım için Docker görüntüleri farklı Azure PowerShell sürümleri arasındaki bağımlılık sorunlarını ortadan kaldırır.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="a8ac3-132">Docker for Windows Windows kapsayıcıları ' nı kullandığınızdan emin olun.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="a8ac3-133">Dağıtım betikleri ile Docker kapsayıcısını almak için yükseltilmiş bir komut isteminde aşağıdaki komutu çalıştırın.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="a8ac3-134">Kümeleri dağıtma</span><span class="sxs-lookup"><span data-stu-id="a8ac3-134">Deploy the clusters</span></span>

1. <span data-ttu-id="a8ac3-135">Kapsayıcı görüntüsü başarıyla çekildikten sonra görüntüyü başlatın.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="a8ac3-136">Kapsayıcı başlatıldıktan sonra, kapsayıcıda yükseltilmiş bir PowerShell terminali vermiş olursunuz.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="a8ac3-137">Dağıtım betiğine ulaşmak için dizinleri değiştirin.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="a8ac3-138">Dağıtımı çalıştırın.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-138">Run the deployment.</span></span> <span data-ttu-id="a8ac3-139">Gereken yerlerde kimlik bilgileri ve kaynak adları sağlayın.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="a8ac3-140">HA, HA kümesinin dağıtılacağı Azure Stack hub 'ına başvurur.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="a8ac3-141">DR, DR kümesinin dağıtılacağı Azure Stack hub 'ına başvurur.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
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

4. <span data-ttu-id="a8ac3-142">`Y`NuGet sağlayıcısı 'nın yüklenmesine izin vermek için yazın ve bu, yüklenecek "2018-03-01-karma" MODÜLLERININ API profilini başlatabilir.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="a8ac3-143">HA kaynakları ilk olarak dağıtılır.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-143">The HA resources will deploy first.</span></span> <span data-ttu-id="a8ac3-144">Dağıtımı izleyin ve bitmesini bekleyin.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="a8ac3-145">HA dağıtımının bittiğini belirten iletiyi aldıktan sonra, dağıtılan kaynakları görmek için HA Azure Stack hub 'ın portalını kontrol edebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="a8ac3-146">DR kaynaklarının dağıtımına devam edin ve kümeyle etkileşim kurmak için DR Azure Stack hub 'ında bir sıçrama kutusunu etkinleştirmek istediğinize karar verin.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="a8ac3-147">DR Kaynak dağıtımının bitmesini bekleyin.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="a8ac3-148">DR kaynak dağıtımı tamamlandıktan sonra kapsayıcıdan çıkın.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="a8ac3-149">Sonraki adımlar</span><span class="sxs-lookup"><span data-stu-id="a8ac3-149">Next steps</span></span>

- <span data-ttu-id="a8ac3-150">DR Azure Stack hub 'ındaki bağlantı kutusu VM 'sini etkinleştirdiyseniz, Mongo CLı 'yı yükleyerek SSH aracılığıyla bağlanabilir ve MongoDB kümesiyle etkileşime geçebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="a8ac3-151">MongoDB ile etkileşim kurma hakkında daha fazla bilgi için bkz. [Mongo kabuğu](https://docs.mongodb.com/manual/mongo/).</span><span class="sxs-lookup"><span data-stu-id="a8ac3-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="a8ac3-152">Hibrit bulut uygulamaları hakkında daha fazla bilgi için bkz [. karma bulut çözümleri.](/azure-stack/user/)</span><span class="sxs-lookup"><span data-stu-id="a8ac3-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="a8ac3-153">[GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)'da kodu bu örnekle değiştirin.</span><span class="sxs-lookup"><span data-stu-id="a8ac3-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>