---
title: Makine öğrenimi modelini kenar düzeninde eğitme
description: Azure ve Azure Stack hub ile en kenarda makine öğrenimi modeli eğitimi yapmayı öğrenin.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911948"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="abc86-103">Makine öğrenimi modelini kenar düzeninde eğitme</span><span class="sxs-lookup"><span data-stu-id="abc86-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="abc86-104">Yalnızca şirket içinde bulunan verilerden taşınabilir makine öğrenimi (ML) modelleri oluşturun.</span><span class="sxs-lookup"><span data-stu-id="abc86-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="abc86-105">Bağlam ve sorun</span><span class="sxs-lookup"><span data-stu-id="abc86-105">Context and problem</span></span>

<span data-ttu-id="abc86-106">Birçok kuruluş, veri bilimcilerinin anladıkları araçları kullanarak şirket içi veya eski verilerden öngörüleri kaldırmak istiyor.</span><span class="sxs-lookup"><span data-stu-id="abc86-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="abc86-107">[Azure Machine Learning](/azure/machine-learning/) , ml ve derin öğrenme modellerini eğmek, ayarlamak ve dağıtmak için buluta özgü araç sağlar.</span><span class="sxs-lookup"><span data-stu-id="abc86-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="abc86-108">Ancak, bazı veriler buluta çok büyük veya yasal nedenlerle buluta gönderilemez.</span><span class="sxs-lookup"><span data-stu-id="abc86-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="abc86-109">Veri bilimcileri, bu modeli kullanarak modelleri şirket içi verileri ve işlem kullanarak eğitebilmeniz için Azure Machine Learning kullanabilir.</span><span class="sxs-lookup"><span data-stu-id="abc86-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="abc86-110">Çözüm</span><span class="sxs-lookup"><span data-stu-id="abc86-110">Solution</span></span>

<span data-ttu-id="abc86-111">Edge düzeninde eğitim, Azure Stack hub 'ında çalışan bir sanal makine (VM) kullanır.</span><span class="sxs-lookup"><span data-stu-id="abc86-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="abc86-112">VM, Azure ML 'de bir işlem hedefi olarak kaydedilir, böylece verilere yalnızca şirket içinde erişilebilir.</span><span class="sxs-lookup"><span data-stu-id="abc86-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="abc86-113">Bu durumda, veriler Azure Stack hub 'ının BLOB depolama alanında depolanır.</span><span class="sxs-lookup"><span data-stu-id="abc86-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="abc86-114">Model eğitilirken Azure ML 'ye kaydedilir, kapsayıcılanmış ve dağıtım için bir Azure Container Registry eklenir.</span><span class="sxs-lookup"><span data-stu-id="abc86-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="abc86-115">Bu düzenin bu yinelemesi için, Azure Stack hub eğitim sanal makinesine genel İnternet üzerinden erişilebilmelidir.</span><span class="sxs-lookup"><span data-stu-id="abc86-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="abc86-116">[![Uç mimaride ML modelini eğitme](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="abc86-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="abc86-117">Düzenin nasıl çalıştığı aşağıda verilmiştir:</span><span class="sxs-lookup"><span data-stu-id="abc86-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="abc86-118">Azure Stack hub sanal makinesi, Azure ML ile bir işlem hedefi olarak dağıtılır ve kaydedilir.</span><span class="sxs-lookup"><span data-stu-id="abc86-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="abc86-119">Azure ML 'de, Azure Stack hub VM 'yi işlem hedefi olarak kullanan bir deneme oluşturulur.</span><span class="sxs-lookup"><span data-stu-id="abc86-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="abc86-120">Model eğitilirken, kaydedilir ve Kapsayıcılı olur.</span><span class="sxs-lookup"><span data-stu-id="abc86-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="abc86-121">Model artık şirket içinde veya bulutta bulunan konumlara dağıtılabilir.</span><span class="sxs-lookup"><span data-stu-id="abc86-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="abc86-122">Bileşenler</span><span class="sxs-lookup"><span data-stu-id="abc86-122">Components</span></span>

<span data-ttu-id="abc86-123">Bu çözüm aşağıdaki bileşenleri kullanır:</span><span class="sxs-lookup"><span data-stu-id="abc86-123">This solution uses the following components:</span></span>

| <span data-ttu-id="abc86-124">Katman</span><span class="sxs-lookup"><span data-stu-id="abc86-124">Layer</span></span> | <span data-ttu-id="abc86-125">Bileşen</span><span class="sxs-lookup"><span data-stu-id="abc86-125">Component</span></span> | <span data-ttu-id="abc86-126">Description</span><span class="sxs-lookup"><span data-stu-id="abc86-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="abc86-127">Azure</span><span class="sxs-lookup"><span data-stu-id="abc86-127">Azure</span></span> | <span data-ttu-id="abc86-128">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="abc86-128">Azure Machine Learning</span></span> | <span data-ttu-id="abc86-129">ML modelinin eğitimini [Azure Machine Learning](/azure/machine-learning/) .</span><span class="sxs-lookup"><span data-stu-id="abc86-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="abc86-130">Azure Container Registry</span><span class="sxs-lookup"><span data-stu-id="abc86-130">Azure Container Registry</span></span> | <span data-ttu-id="abc86-131">Azure ML, modeli bir kapsayıcıya paketler ve dağıtım için bir [Azure Container Registry](/azure/container-registry/) depolar.</span><span class="sxs-lookup"><span data-stu-id="abc86-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="abc86-132">Azure Stack hub 'ı</span><span class="sxs-lookup"><span data-stu-id="abc86-132">Azure Stack Hub</span></span> | <span data-ttu-id="abc86-133">App Service</span><span class="sxs-lookup"><span data-stu-id="abc86-133">App Service</span></span> | <span data-ttu-id="abc86-134">[App Service olan Azure Stack hub](/azure-stack/operator/azure-stack-app-service-overview) , kenardaki bileşenlerin temelini sağlar.</span><span class="sxs-lookup"><span data-stu-id="abc86-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="abc86-135">İşlem</span><span class="sxs-lookup"><span data-stu-id="abc86-135">Compute</span></span> | <span data-ttu-id="abc86-136">ML modelini eğitmek için Docker ile Ubuntu çalıştıran bir Azure Stack hub VM kullanılır.</span><span class="sxs-lookup"><span data-stu-id="abc86-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="abc86-137">Depolama</span><span class="sxs-lookup"><span data-stu-id="abc86-137">Storage</span></span> | <span data-ttu-id="abc86-138">Özel veriler Azure Stack hub BLOB depolama alanında barındırılabilir.</span><span class="sxs-lookup"><span data-stu-id="abc86-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="abc86-139">Sorunlar ve dikkat edilmesi gerekenler</span><span class="sxs-lookup"><span data-stu-id="abc86-139">Issues and considerations</span></span>

<span data-ttu-id="abc86-140">Bu çözümü nasıl uygulayacağınıza karar verirken aşağıdaki noktaları göz önünde bulundurun:</span><span class="sxs-lookup"><span data-stu-id="abc86-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="abc86-141">Ölçeklenebilirlik</span><span class="sxs-lookup"><span data-stu-id="abc86-141">Scalability</span></span>

<span data-ttu-id="abc86-142">Bu çözümün ölçeklendirilmesini sağlamak için, eğitim için Azure Stack hub 'ında uygun boyutta bir VM oluşturmanız gerekir.</span><span class="sxs-lookup"><span data-stu-id="abc86-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="abc86-143">Kullanılabilirlik</span><span class="sxs-lookup"><span data-stu-id="abc86-143">Availability</span></span>

<span data-ttu-id="abc86-144">Eğitim betikleri ve Azure Stack hub VM 'nin eğitim için kullanılan şirket içi verilere erişimi olduğundan emin olun.</span><span class="sxs-lookup"><span data-stu-id="abc86-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="abc86-145">Yönetilebilirlik</span><span class="sxs-lookup"><span data-stu-id="abc86-145">Manageability</span></span>

<span data-ttu-id="abc86-146">Model dağıtımı sırasında karışıklıkları önlemek için modeller ve denemeleri uygun şekilde kaydedildiğinden, sürümlenmiş ve etiketlediğinizden emin olun.</span><span class="sxs-lookup"><span data-stu-id="abc86-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="abc86-147">Güvenlik</span><span class="sxs-lookup"><span data-stu-id="abc86-147">Security</span></span>

<span data-ttu-id="abc86-148">Bu model, Azure ML 'nin şirket içi olası hassas verilere erişmesine olanak tanır.</span><span class="sxs-lookup"><span data-stu-id="abc86-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="abc86-149">Azure Stack hub VM 'ye SSH için kullanılan hesabın güçlü bir parolaya sahip olduğundan ve eğitim betiklerinin buluta veri korumamasını veya bu verileri karşıya yüklemediğinden emin olun.</span><span class="sxs-lookup"><span data-stu-id="abc86-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="abc86-150">Sonraki adımlar</span><span class="sxs-lookup"><span data-stu-id="abc86-150">Next steps</span></span>

<span data-ttu-id="abc86-151">Bu makalede tanıtılan konular hakkında daha fazla bilgi edinmek için:</span><span class="sxs-lookup"><span data-stu-id="abc86-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="abc86-152">ML ve ilgili konulara genel bakış için [Azure Machine Learning belgelerine](/azure/machine-learning) bakın.</span><span class="sxs-lookup"><span data-stu-id="abc86-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="abc86-153">Kapsayıcı dağıtımları için görüntü oluşturma, depolama ve yönetme hakkında bilgi edinmek için bkz. [Azure Container Registry](/azure/container-registry/) .</span><span class="sxs-lookup"><span data-stu-id="abc86-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="abc86-154">Kaynak sağlayıcısı ve dağıtımı hakkında daha fazla bilgi edinmek için [Azure Stack hub 'ındaki App Service](/azure-stack/operator/azure-stack-app-service-overview) başvurun.</span><span class="sxs-lookup"><span data-stu-id="abc86-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="abc86-155">En iyi uygulamalar hakkında daha fazla bilgi edinmek ve Cevaplanan ek sorulara ulaşmak için bkz. [karma uygulama tasarımı konuları](overview-app-design-considerations.md) .</span><span class="sxs-lookup"><span data-stu-id="abc86-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="abc86-156">Ürünlerin ve çözümlerin tamamı hakkında daha fazla bilgi edinmek için [ürün ve çözümlerin Azure Stack ailesine](/azure-stack) bakın.</span><span class="sxs-lookup"><span data-stu-id="abc86-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="abc86-157">Çözüm örneğini test etmeye hazırsanız, [Edge dağıtım kılavuzunda EĞITIM ml modeliyle](https://aka.ms/edgetrainingdeploy)devam edin.</span><span class="sxs-lookup"><span data-stu-id="abc86-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="abc86-158">Dağıtım Kılavuzu, bileşenlerinin dağıtılması ve test edilmesi için adım adım yönergeler sağlar.</span><span class="sxs-lookup"><span data-stu-id="abc86-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
