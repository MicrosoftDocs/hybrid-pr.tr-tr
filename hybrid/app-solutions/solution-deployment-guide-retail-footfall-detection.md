---
title: Azure 'da ve Azure Stack hub 'da AI tabanlı bir kantfall algılama çözümü dağıtma
description: Azure ve Azure Stack hub 'ı kullanarak perakende mağazalarındaki ziyaretçi trafiğini çözümlemek için bir AI tabanlı bir betfall algılama çözümünü dağıtmayı öğrenin.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6913cc522da447092dad0af24e148a3b2576495c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911863"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="1b47a-103">Azure ve Azure Stack hub kullanarak bir AI tabanlı bir kantfall algılama çözümü dağıtma</span><span class="sxs-lookup"><span data-stu-id="1b47a-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="1b47a-104">Bu makalede, Azure, Azure Stack hub ve Özel Görüntü İşleme AI Dev Kit kullanarak gerçek dünya eylemlerinden Öngörüler üreten bir AI tabanlı çözümün nasıl dağıtılacağı açıklanır.</span><span class="sxs-lookup"><span data-stu-id="1b47a-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="1b47a-105">Bu çözümde şunları yapmayı öğreneceksiniz:</span><span class="sxs-lookup"><span data-stu-id="1b47a-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="1b47a-106">Bulut Yerel uygulama paketleri 'ni (CNAB) kenarda dağıtın.</span><span class="sxs-lookup"><span data-stu-id="1b47a-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="1b47a-107">Bulut sınırlarına yayılan bir uygulama dağıtın.</span><span class="sxs-lookup"><span data-stu-id="1b47a-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="1b47a-108">Kenardaki çıkarım için Özel Görüntü İşleme AI geliştirme setini kullanın.</span><span class="sxs-lookup"><span data-stu-id="1b47a-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="1b47a-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="1b47a-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="1b47a-110">Microsoft Azure Stack hub, Azure uzantısıdır.</span><span class="sxs-lookup"><span data-stu-id="1b47a-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="1b47a-111">Azure Stack hub, bulut bilgi işlemin çevikliğini ve yeniliklerini şirket içi ortamınıza sunarak, karma uygulamaları her yerde derleyip dağıtmanıza imkan tanıyan tek karma bulutu etkinleştirir.</span><span class="sxs-lookup"><span data-stu-id="1b47a-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="1b47a-112">[Karma uygulama tasarımı ile ilgili önemli noktalar](overview-app-design-considerations.md) , karma uygulamalar tasarlamak, dağıtmak ve çalıştırmak için yazılım kalitesinin (yerleştirme, ölçeklenebilirlik, kullanılabilirlik, dayanıklılık, yönetilebilirlik ve güvenlik) aynı şekilde gözden geçirmeleri inceler.</span><span class="sxs-lookup"><span data-stu-id="1b47a-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="1b47a-113">Tasarım konuları karma uygulama tasarımını iyileştirirken, üretim ortamlarındaki zorlukları en aza indirmeyle ilgili olarak size yardımcı olur.</span><span class="sxs-lookup"><span data-stu-id="1b47a-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="1b47a-114">Ön koşullar</span><span class="sxs-lookup"><span data-stu-id="1b47a-114">Prerequisites</span></span>

<span data-ttu-id="1b47a-115">Bu dağıtım kılavuzunu kullanmaya başlamadan önce şunları yaptığınızdan emin olun:</span><span class="sxs-lookup"><span data-stu-id="1b47a-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="1b47a-116">[Betfall algılama deseninin](pattern-retail-footfall-detection.md) konusunu gözden geçirin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="1b47a-117">Azure Stack Geliştirme Seti (ASDK) veya Azure Stack hub ile tümleşik sistem örneğine Kullanıcı erişimini edinin:</span><span class="sxs-lookup"><span data-stu-id="1b47a-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="1b47a-118">[Azure Stack hub kaynak sağlayıcısında Azure App Service](/azure-stack/operator/azure-stack-app-service-overview.md) yüklendi.</span><span class="sxs-lookup"><span data-stu-id="1b47a-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview.md) installed.</span></span> <span data-ttu-id="1b47a-119">Azure Stack hub örneğiniz için operatör erişiminizin olması veya yöneticinizin yüklenebilmesi için yöneticinizle birlikte çalışmanız gerekir.</span><span class="sxs-lookup"><span data-stu-id="1b47a-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="1b47a-120">App Service ve depolama kotası sağlayan bir teklifin aboneliği.</span><span class="sxs-lookup"><span data-stu-id="1b47a-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="1b47a-121">Teklif oluşturmak için operatör erişiminizin olması gerekir.</span><span class="sxs-lookup"><span data-stu-id="1b47a-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="1b47a-122">Bir Azure aboneliğine erişim elde edin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="1b47a-123">Azure aboneliğiniz yoksa başlamadan önce [ücretsiz deneme hesabına](https://azure.microsoft.com/free/) kaydolun.</span><span class="sxs-lookup"><span data-stu-id="1b47a-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="1b47a-124">Dizininizde iki hizmet sorumlusu oluşturun:</span><span class="sxs-lookup"><span data-stu-id="1b47a-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="1b47a-125">Azure abonelik kapsamında erişim ile Azure kaynaklarıyla kullanılmak üzere ayarlanmış bir.</span><span class="sxs-lookup"><span data-stu-id="1b47a-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="1b47a-126">Azure Stack hub kaynaklarıyla kullanılmak üzere, Azure Stack hub aboneliği kapsamında erişimli bir kurulum.</span><span class="sxs-lookup"><span data-stu-id="1b47a-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="1b47a-127">Hizmet sorumluları oluşturma ve erişimi yetkilendirme hakkında daha fazla bilgi edinmek için bkz. [kaynaklara erişmek için uygulama kimliği kullanma](/azure-stack/operator/azure-stack-create-service-principals.md).</span><span class="sxs-lookup"><span data-stu-id="1b47a-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md).</span></span> <span data-ttu-id="1b47a-128">Azure CLı 'yı kullanmayı tercih ediyorsanız bkz. Azure [CLI Ile Azure hizmet sorumlusu oluşturma](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="1b47a-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).</span></span>
- <span data-ttu-id="1b47a-129">Azure bilişsel hizmetler 'i Azure 'da veya Azure Stack hub 'da dağıtın.</span><span class="sxs-lookup"><span data-stu-id="1b47a-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="1b47a-130">İlk olarak bilişsel [Hizmetler hakkında daha fazla bilgi edinin](https://azure.microsoft.com/services/cognitive-services/).</span><span class="sxs-lookup"><span data-stu-id="1b47a-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="1b47a-131">Daha sonra, Azure bilişsel [Hizmetler 'i Azure Stack hub 'A dağıtmayı](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) , bilişsel hizmetler 'ı Azure Stack hub 'a dağıtmak</span><span class="sxs-lookup"><span data-stu-id="1b47a-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="1b47a-132">Önce önizlemeye erişim için kaydolmanız gerekir.</span><span class="sxs-lookup"><span data-stu-id="1b47a-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="1b47a-133">Yapılandırılmamış bir Azure Özel Görüntü İşleme AI geliştirme setini kopyalayın veya indirin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="1b47a-134">Ayrıntılar için bkz. [VISION AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span><span class="sxs-lookup"><span data-stu-id="1b47a-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="1b47a-135">Power BI hesaba kaydolun.</span><span class="sxs-lookup"><span data-stu-id="1b47a-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="1b47a-136">Azure bilişsel Hizmetler Yüz Tanıma API'si abonelik anahtarı ve uç nokta URL 'SI.</span><span class="sxs-lookup"><span data-stu-id="1b47a-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="1b47a-137">Her ikisini de deneme bilişsel [Hizmetler](https://azure.microsoft.com/try/cognitive-services/?api=face-api) ücretsiz deneme sürümü ile edinebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="1b47a-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="1b47a-138">Veya bilişsel [Hizmetler hesabı oluşturma](/azure/cognitive-services/cognitive-services-apis-create-account)' daki yönergeleri izleyin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="1b47a-139">Aşağıdaki geliştirme kaynaklarını yükler:</span><span class="sxs-lookup"><span data-stu-id="1b47a-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="1b47a-140">Azure CLI 2.0</span><span class="sxs-lookup"><span data-stu-id="1b47a-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [<span data-ttu-id="1b47a-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="1b47a-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="1b47a-142">[Porter](https://porter.sh/).</span><span class="sxs-lookup"><span data-stu-id="1b47a-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="1b47a-143">Sizin için sağlanmış olan CNAB paket bildirimlerini kullanarak bulut uygulamalarını dağıtmak için bağlantı noktası kullanırsınız.</span><span class="sxs-lookup"><span data-stu-id="1b47a-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="1b47a-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="1b47a-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="1b47a-145">Visual Studio Code için Azure IoT araçları</span><span class="sxs-lookup"><span data-stu-id="1b47a-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="1b47a-146">Visual Studio Code için Python uzantısı</span><span class="sxs-lookup"><span data-stu-id="1b47a-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="1b47a-147">Python</span><span class="sxs-lookup"><span data-stu-id="1b47a-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="1b47a-148">Hibrit bulut uygulamasını dağıtma</span><span class="sxs-lookup"><span data-stu-id="1b47a-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="1b47a-149">İlk olarak, bir kimlik bilgisi kümesi oluşturmak için Porter CLı 'sını kullanın ve ardından bulut uygulamasını dağıtın.</span><span class="sxs-lookup"><span data-stu-id="1b47a-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="1b47a-150">Çözüm örnek kodunu kopyalayın veya indirin https://github.com/azure-samples/azure-intelligent-edge-patterns .</span><span class="sxs-lookup"><span data-stu-id="1b47a-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="1b47a-151">Porter, uygulamanın dağıtımını otomatikleştiren bir kimlik bilgileri kümesi oluşturur.</span><span class="sxs-lookup"><span data-stu-id="1b47a-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="1b47a-152">Kimlik bilgisi oluşturma komutunu çalıştırmadan önce aşağıdakilerin kullanılabilir olduğundan emin olun:</span><span class="sxs-lookup"><span data-stu-id="1b47a-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="1b47a-153">Hizmet sorumlusu KIMLIĞI, anahtarı ve kiracı DNS de dahil olmak üzere Azure kaynaklarına erişmek için bir hizmet sorumlusu.</span><span class="sxs-lookup"><span data-stu-id="1b47a-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="1b47a-154">Azure aboneliğiniz için abonelik KIMLIĞI.</span><span class="sxs-lookup"><span data-stu-id="1b47a-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="1b47a-155">Hizmet sorumlusu KIMLIĞI, anahtarı ve kiracı DNS de dahil olmak üzere Azure Stack hub kaynaklarına erişmek için bir hizmet sorumlusu.</span><span class="sxs-lookup"><span data-stu-id="1b47a-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="1b47a-156">Azure Stack hub aboneliğiniz için abonelik KIMLIĞI.</span><span class="sxs-lookup"><span data-stu-id="1b47a-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="1b47a-157">Azure bilişsel hizmetlerinizin anahtar ve kaynak uç nokta URL 'SI Yüz Tanıma API'si.</span><span class="sxs-lookup"><span data-stu-id="1b47a-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="1b47a-158">Porter kimlik bilgileri oluşturma işlemini çalıştırın ve istemleri izleyin:</span><span class="sxs-lookup"><span data-stu-id="1b47a-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="1b47a-159">Porter, çalıştırılacak bir parametre kümesi de gerektirir.</span><span class="sxs-lookup"><span data-stu-id="1b47a-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="1b47a-160">Bir parametre metin dosyası oluşturun ve aşağıdaki ad/değer çiftlerini girin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="1b47a-161">Gerekli değerlerden herhangi biri ile ilgili yardıma ihtiyacınız varsa Azure Stack hub yöneticinize danışın.</span><span class="sxs-lookup"><span data-stu-id="1b47a-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="1b47a-162">`resource suffix`Bu değer, dağıtımınızın kaynaklarının Azure genelinde benzersiz adlara sahip olduğundan emin olmak için kullanılır.</span><span class="sxs-lookup"><span data-stu-id="1b47a-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="1b47a-163">8 karakterden uzun olmayan harflerin ve sayıların benzersiz bir dizesi olmalıdır.</span><span class="sxs-lookup"><span data-stu-id="1b47a-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   <span data-ttu-id="1b47a-164">Metin dosyasını kaydedin ve yolunu bir yere göz önünde yapın.</span><span class="sxs-lookup"><span data-stu-id="1b47a-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="1b47a-165">Artık, hiter kullanarak hibrit bulut uygulamasını dağıtmaya hazırsınız.</span><span class="sxs-lookup"><span data-stu-id="1b47a-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="1b47a-166">Install komutunu çalıştırın ve kaynaklar Azure 'da ve Azure Stack hub 'a dağıtılırken izleyin:</span><span class="sxs-lookup"><span data-stu-id="1b47a-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="1b47a-167">Dağıtım tamamlandıktan sonra, aşağıdaki değerleri unutmayın:</span><span class="sxs-lookup"><span data-stu-id="1b47a-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="1b47a-168">Kameranın bağlantı dizesi.</span><span class="sxs-lookup"><span data-stu-id="1b47a-168">The camera's connection string.</span></span>
    - <span data-ttu-id="1b47a-169">Görüntü depolama hesabı bağlantı dizesi.</span><span class="sxs-lookup"><span data-stu-id="1b47a-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="1b47a-170">Kaynak grubu adları.</span><span class="sxs-lookup"><span data-stu-id="1b47a-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="1b47a-171">Özel Görüntü İşleme AI DevKit 'ı hazırlama</span><span class="sxs-lookup"><span data-stu-id="1b47a-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="1b47a-172">Ardından, Özel Görüntü İşleme AI geliştirme setini, [VISION AI DevKit hızlı başlangıç](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/)bölümünde gösterildiği gibi ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="1b47a-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="1b47a-173">Ayrıca, önceki adımda belirtilen bağlantı dizesini kullanarak kameranızı ayarlayıp test edersiniz.</span><span class="sxs-lookup"><span data-stu-id="1b47a-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="1b47a-174">Kamera uygulamasını dağıtma</span><span class="sxs-lookup"><span data-stu-id="1b47a-174">Deploy the camera app</span></span>

<span data-ttu-id="1b47a-175">Bir kimlik bilgisi kümesi oluşturmak için Porter CLı 'sını kullanın, ardından Kamera uygulamasını dağıtın.</span><span class="sxs-lookup"><span data-stu-id="1b47a-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="1b47a-176">Porter, uygulamanın dağıtımını otomatikleştiren bir kimlik bilgileri kümesi oluşturur.</span><span class="sxs-lookup"><span data-stu-id="1b47a-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="1b47a-177">Kimlik bilgisi oluşturma komutunu çalıştırmadan önce aşağıdakilerin kullanılabilir olduğundan emin olun:</span><span class="sxs-lookup"><span data-stu-id="1b47a-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="1b47a-178">Hizmet sorumlusu KIMLIĞI, anahtarı ve kiracı DNS de dahil olmak üzere Azure kaynaklarına erişmek için bir hizmet sorumlusu.</span><span class="sxs-lookup"><span data-stu-id="1b47a-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="1b47a-179">Azure aboneliğiniz için abonelik KIMLIĞI.</span><span class="sxs-lookup"><span data-stu-id="1b47a-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="1b47a-180">Bulut uygulamasını dağıtırken sunulan görüntü depolama hesabı bağlantı dizesi.</span><span class="sxs-lookup"><span data-stu-id="1b47a-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="1b47a-181">Porter kimlik bilgileri oluşturma işlemini çalıştırın ve istemleri izleyin:</span><span class="sxs-lookup"><span data-stu-id="1b47a-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="1b47a-182">Porter, çalıştırılacak bir parametre kümesi de gerektirir.</span><span class="sxs-lookup"><span data-stu-id="1b47a-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="1b47a-183">Bir parametre metin dosyası oluşturun ve aşağıdaki metni girin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="1b47a-184">Gerekli değerlerden bazılarını bilmiyorsanız Azure Stack hub yöneticinize danışın.</span><span class="sxs-lookup"><span data-stu-id="1b47a-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="1b47a-185">`deployment suffix`Bu değer, dağıtımınızın kaynaklarının Azure genelinde benzersiz adlara sahip olduğundan emin olmak için kullanılır.</span><span class="sxs-lookup"><span data-stu-id="1b47a-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="1b47a-186">8 karakterden uzun olmayan harflerin ve sayıların benzersiz bir dizesi olmalıdır.</span><span class="sxs-lookup"><span data-stu-id="1b47a-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="1b47a-187">Metin dosyasını kaydedin ve yolunu bir yere göz önünde yapın.</span><span class="sxs-lookup"><span data-stu-id="1b47a-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="1b47a-188">Artık, bağlantı noktası kullanarak Kamera uygulamasını dağıtmaya hazırsınız demektir.</span><span class="sxs-lookup"><span data-stu-id="1b47a-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="1b47a-189">Install komutunu çalıştırın ve IoT Edge dağıtımı oluşturulduğundan izleyin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="1b47a-190">Kameranın `https://<camera-ip>:3000/` , kamera IP adresi olan konumundaki kamera akışını görüntüleyerek kameranın dağıtımının tamamlandığını doğrulayın `<camara-ip>` .</span><span class="sxs-lookup"><span data-stu-id="1b47a-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="1b47a-191">Bu adım 10 dakikaya kadar sürebilir.</span><span class="sxs-lookup"><span data-stu-id="1b47a-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="1b47a-192">Azure Stream Analytics Yapılandır</span><span class="sxs-lookup"><span data-stu-id="1b47a-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="1b47a-193">Artık veriler kameradan Azure Stream Analytics akar, Power BI ile iletişim kurmak için el ile yetkilendirmeniz gerekir.</span><span class="sxs-lookup"><span data-stu-id="1b47a-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="1b47a-194">Azure portal, **tüm kaynaklar**' ı ve \*Process-mtfall \[ yoursuffix \] \* işini açın.</span><span class="sxs-lookup"><span data-stu-id="1b47a-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="1b47a-195">Stream Analytics iş bölmesinin **İş Topolojisi** bölümünde **Çıkışlar** seçeneğini belirleyin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="1b47a-196">**Trafik çıkışı** çıkış havuzunu seçin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="1b47a-197">**Yetkilendirmeyi Yenile** ve Power BI hesabınızda oturum aç ' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Power BI yetkilendirme isteğini yenileme](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="1b47a-199">Çıkış ayarlarını kaydedin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-199">Save the output settings.</span></span>

6. <span data-ttu-id="1b47a-200">**Genel bakış** bölmesine gidin ve Power BI veri göndermeye başlamak için **Başlat** ' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="1b47a-201">İş çıkışı başlangıç saati için **Şimdi**’yi seçip **Başlat** seçeneğini belirleyin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="1b47a-202">İş durumunu bildirim çubuğunda durumu görüntüleyebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="1b47a-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="1b47a-203">Power BI panosu oluşturma</span><span class="sxs-lookup"><span data-stu-id="1b47a-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="1b47a-204">İş başarılı olduktan sonra, [Power BI](https://powerbi.com/) gidin ve iş veya okul hesabınızla oturum açın.</span><span class="sxs-lookup"><span data-stu-id="1b47a-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="1b47a-205">Stream Analytics iş sorgusu sonuçları alıyorsa, oluşturduğunuz alt *veri* kümesi veri kümesi veri **kümeleri** sekmesinde bulunur.</span><span class="sxs-lookup"><span data-stu-id="1b47a-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="1b47a-206">Power BI çalışma alanınızdan, *Ptfall Analizi* adlı yeni bir pano oluşturmak Için **+ Oluştur** ' u seçin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="1b47a-207">Pencerenin üst kısmındaki **Kutucuk ekle**’yi seçin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="1b47a-208">Ardından **Özel Akış Verileri**'ni ve **İleri**'yi seçin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="1b47a-209">**Veri kümeleriniz**altındaki alt **veri kümesini** seçin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="1b47a-210">**Görselleştirme türü** açılan menüsünden **kart** ' ı seçin ve alanlara **yaş** ekleyin **Fields**.</span><span class="sxs-lookup"><span data-stu-id="1b47a-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="1b47a-211">**İleri**'yi seçip ad belirledikten sonra **Uygula**'yı seçerek kutucuğu oluşturun.</span><span class="sxs-lookup"><span data-stu-id="1b47a-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="1b47a-212">İstediğiniz gibi ek alanlar ve kartlar ekleyebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="1b47a-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="1b47a-213">Çözümünüzü test etme</span><span class="sxs-lookup"><span data-stu-id="1b47a-213">Test Your Solution</span></span>

<span data-ttu-id="1b47a-214">Farklı kişilerin kameranın önüne ilerleyirken Power BI ' de oluşturduğunuz karttaki verilerin değiştiğini gözlemleyin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="1b47a-215">Inele, kaydedildikten sonra görünmesi 20 saniyeye kadar sürebilir.</span><span class="sxs-lookup"><span data-stu-id="1b47a-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="1b47a-216">Çözümünüzü kaldırma</span><span class="sxs-lookup"><span data-stu-id="1b47a-216">Remove Your Solution</span></span>

<span data-ttu-id="1b47a-217">Çözümünüzü kaldırmak isterseniz, dağıtım için oluşturduğunuz aynı parametre dosyalarını kullanarak, Porter kullanarak aşağıdaki komutları çalıştırın:</span><span class="sxs-lookup"><span data-stu-id="1b47a-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="1b47a-218">Sonraki adımlar</span><span class="sxs-lookup"><span data-stu-id="1b47a-218">Next steps</span></span>

- <span data-ttu-id="1b47a-219">[Karma uygulama tasarımı konuları] hakkında daha fazla bilgi edinin. (overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="1b47a-219">Learn more about [Hybrid app design considerations].(overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="1b47a-220">[GitHub 'da Bu örnek için koda](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis)yönelik geliştirmeleri gözden geçirin ve önerin.</span><span class="sxs-lookup"><span data-stu-id="1b47a-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
