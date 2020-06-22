---
title: Azure ve Azure Stack hub 'daki karma geçiş deseninin
description: Güvenlik duvarları tarafından korunan Edge kaynaklarına bağlanmak için Azure ve Azure Stack hub 'daki karma geçiş modelini kullanın.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911971"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="9b72e-103">Karma geçiş deseninin</span><span class="sxs-lookup"><span data-stu-id="9b72e-103">Hybrid relay pattern</span></span>

<span data-ttu-id="9b72e-104">Karma geçiş modelini ve Azure Relay kullanarak güvenlik duvarları tarafından korunan Edge kaynaklarına veya cihazlara nasıl bağlanacağınızı öğrenin.</span><span class="sxs-lookup"><span data-stu-id="9b72e-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="9b72e-105">Bağlam ve sorun</span><span class="sxs-lookup"><span data-stu-id="9b72e-105">Context and problem</span></span>

<span data-ttu-id="9b72e-106">Sınır cihazları çoğunlukla bir kurumsal güvenlik duvarı veya NAT cihazının arkasında bulunur.</span><span class="sxs-lookup"><span data-stu-id="9b72e-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="9b72e-107">Güvenli olsalar da, diğer şirket ağlarında ortak bulut veya uç cihazlarla iletişim kuramayabilirler.</span><span class="sxs-lookup"><span data-stu-id="9b72e-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="9b72e-108">Genel buluttaki kullanıcılara güvenli bir şekilde belirli bağlantı noktalarını ve işlevleri göstermek gerekebilir.</span><span class="sxs-lookup"><span data-stu-id="9b72e-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="9b72e-109">Çözüm</span><span class="sxs-lookup"><span data-stu-id="9b72e-109">Solution</span></span>

<span data-ttu-id="9b72e-110">Karma geçiş modelinde, doğrudan iletişim kuramayan iki uç nokta arasında bir WebSockets tüneli kurmak için Azure Relay kullanılır.</span><span class="sxs-lookup"><span data-stu-id="9b72e-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="9b72e-111">Şirket içinde olmayan ancak şirket içi bir uç noktaya bağlanması gereken cihazlar, genel buluttaki bir uç noktaya bağlanır.</span><span class="sxs-lookup"><span data-stu-id="9b72e-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="9b72e-112">Bu uç nokta, güvenli bir kanal üzerinden önceden tanımlı yolların trafiğini yeniden yönlendirir.</span><span class="sxs-lookup"><span data-stu-id="9b72e-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="9b72e-113">Şirket içi ortamın içindeki bir uç nokta trafiği alır ve doğru hedefe yönlendirir.</span><span class="sxs-lookup"><span data-stu-id="9b72e-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![Karma geçiş deseninin çözüm mimarisi](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="9b72e-115">Hibrit geçiş deseninin nasıl çalıştığı aşağıda verilmiştir:</span><span class="sxs-lookup"><span data-stu-id="9b72e-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="9b72e-116">Bir cihaz, önceden tanımlanmış bir bağlantı noktasında Azure 'daki sanal makineye (VM) bağlanır.</span><span class="sxs-lookup"><span data-stu-id="9b72e-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="9b72e-117">Trafik Azure 'daki Azure Relay iletilir.</span><span class="sxs-lookup"><span data-stu-id="9b72e-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="9b72e-118">Zaten Azure Relay uzun süreli bir bağlantı oluşturmuş Azure Stack hub 'ındaki VM, trafiği alır ve hedefe iletir.</span><span class="sxs-lookup"><span data-stu-id="9b72e-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="9b72e-119">Şirket içi hizmet veya uç nokta, isteği işler.</span><span class="sxs-lookup"><span data-stu-id="9b72e-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="9b72e-120">Bileşenler</span><span class="sxs-lookup"><span data-stu-id="9b72e-120">Components</span></span>

<span data-ttu-id="9b72e-121">Bu çözüm aşağıdaki bileşenleri kullanır:</span><span class="sxs-lookup"><span data-stu-id="9b72e-121">This solution uses the following components:</span></span>

| <span data-ttu-id="9b72e-122">Katman</span><span class="sxs-lookup"><span data-stu-id="9b72e-122">Layer</span></span> | <span data-ttu-id="9b72e-123">Bileşen</span><span class="sxs-lookup"><span data-stu-id="9b72e-123">Component</span></span> | <span data-ttu-id="9b72e-124">Description</span><span class="sxs-lookup"><span data-stu-id="9b72e-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="9b72e-125">Azure</span><span class="sxs-lookup"><span data-stu-id="9b72e-125">Azure</span></span> | <span data-ttu-id="9b72e-126">Azure VM</span><span class="sxs-lookup"><span data-stu-id="9b72e-126">Azure VM</span></span> | <span data-ttu-id="9b72e-127">Azure VM, şirket içi kaynak için genel olarak erişilebilen bir uç nokta sağlar.</span><span class="sxs-lookup"><span data-stu-id="9b72e-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="9b72e-128">Azure Geçişi</span><span class="sxs-lookup"><span data-stu-id="9b72e-128">Azure Relay</span></span> | <span data-ttu-id="9b72e-129">[Azure Relay](/azure/azure-relay/) , Azure vm Ile Azure Stack hub sanal makinesi arasındaki tüneli ve bağlantıyı sürdürmek için altyapı sağlar.</span><span class="sxs-lookup"><span data-stu-id="9b72e-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="9b72e-130">Azure Stack hub 'ı</span><span class="sxs-lookup"><span data-stu-id="9b72e-130">Azure Stack Hub</span></span> | <span data-ttu-id="9b72e-131">İşlem</span><span class="sxs-lookup"><span data-stu-id="9b72e-131">Compute</span></span> | <span data-ttu-id="9b72e-132">Azure Stack hub sanal makinesi, karma geçiş tünelinin sunucu tarafını sağlar.</span><span class="sxs-lookup"><span data-stu-id="9b72e-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="9b72e-133">Depolama</span><span class="sxs-lookup"><span data-stu-id="9b72e-133">Storage</span></span> | <span data-ttu-id="9b72e-134">Azure Stack hub 'ına dağıtılan AKS motoru kümesi, Yüz Tanıma API'si kapsayıcısını çalıştırmak için ölçeklenebilir ve dayanıklı bir altyapı sağlar.</span><span class="sxs-lookup"><span data-stu-id="9b72e-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="9b72e-135">Sorunlar ve dikkat edilmesi gerekenler</span><span class="sxs-lookup"><span data-stu-id="9b72e-135">Issues and considerations</span></span>

<span data-ttu-id="9b72e-136">Bu çözümü nasıl uygulayacağınıza karar verirken aşağıdaki noktaları göz önünde bulundurun:</span><span class="sxs-lookup"><span data-stu-id="9b72e-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="9b72e-137">Ölçeklenebilirlik</span><span class="sxs-lookup"><span data-stu-id="9b72e-137">Scalability</span></span>

<span data-ttu-id="9b72e-138">Bu model yalnızca istemci ve sunucuda 1:1 bağlantı noktası eşlemelerine izin verir.</span><span class="sxs-lookup"><span data-stu-id="9b72e-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="9b72e-139">Örneğin, bağlantı noktası 80, Azure uç noktasındaki bir hizmet için tünededir, başka bir hizmet için kullanılamaz.</span><span class="sxs-lookup"><span data-stu-id="9b72e-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="9b72e-140">Bağlantı noktası eşlemeleri uygun şekilde planlanmalıdır.</span><span class="sxs-lookup"><span data-stu-id="9b72e-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="9b72e-141">Azure Relay ve VM 'Ler trafiği işlemek için uygun şekilde ölçeklendirmelidir.</span><span class="sxs-lookup"><span data-stu-id="9b72e-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="9b72e-142">Kullanılabilirlik</span><span class="sxs-lookup"><span data-stu-id="9b72e-142">Availability</span></span>

<span data-ttu-id="9b72e-143">Bu tüneller ve bağlantılar gereksiz değildir.</span><span class="sxs-lookup"><span data-stu-id="9b72e-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="9b72e-144">Yüksek kullanılabilirlik sağlamak için, hata denetimi kodu uygulamak isteyebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="9b72e-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="9b72e-145">Başka bir seçenek de, yük dengeleyicinin arkasında Azure Relay bağlı VM havuzu olmalıdır.</span><span class="sxs-lookup"><span data-stu-id="9b72e-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="9b72e-146">Yönetilebilirlik</span><span class="sxs-lookup"><span data-stu-id="9b72e-146">Manageability</span></span>

<span data-ttu-id="9b72e-147">Bu çözüm, çok sayıda cihaza ve konuma yayılabilir ve bu da bir süre sonra olabilir.</span><span class="sxs-lookup"><span data-stu-id="9b72e-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="9b72e-148">Azure 'un IoT Hizmetleri, yeni konumları ve cihazları otomatik olarak çevrimiçi duruma getirebilir ve güncel tutar.</span><span class="sxs-lookup"><span data-stu-id="9b72e-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="9b72e-149">Güvenlik</span><span class="sxs-lookup"><span data-stu-id="9b72e-149">Security</span></span>

<span data-ttu-id="9b72e-150">Bu model, uçtan bir iç cihazdaki bağlantı noktasına sınırsız erişimine izin verir.</span><span class="sxs-lookup"><span data-stu-id="9b72e-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="9b72e-151">İç cihazdaki hizmete veya karma geçiş uç noktasının önüne bir kimlik doğrulama mekanizması eklemeyi düşünün.</span><span class="sxs-lookup"><span data-stu-id="9b72e-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="9b72e-152">Sonraki adımlar</span><span class="sxs-lookup"><span data-stu-id="9b72e-152">Next steps</span></span>

<span data-ttu-id="9b72e-153">Bu makalede tanıtılan konular hakkında daha fazla bilgi edinmek için:</span><span class="sxs-lookup"><span data-stu-id="9b72e-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="9b72e-154">Bu kalıp Azure Relay kullanır.</span><span class="sxs-lookup"><span data-stu-id="9b72e-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="9b72e-155">Daha fazla bilgi için [Azure Relay belgelerine](/azure/azure-relay/)bakın.</span><span class="sxs-lookup"><span data-stu-id="9b72e-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="9b72e-156">En iyi uygulamalar hakkında daha fazla bilgi edinmek ve ek sorulara yanıt almak için bkz. [karma uygulama tasarımı konuları](overview-app-design-considerations.md) .</span><span class="sxs-lookup"><span data-stu-id="9b72e-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="9b72e-157">Ürünlerin ve çözümlerin tamamı hakkında daha fazla bilgi edinmek için [ürün ve çözümlerin Azure Stack ailesine](/azure-stack) bakın.</span><span class="sxs-lookup"><span data-stu-id="9b72e-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="9b72e-158">Çözüm örneğini test etmeye hazırsanız [karma geçiş çözümü dağıtım kılavuzu](https://aka.ms/hybridrelaydeployment)ile devam edin.</span><span class="sxs-lookup"><span data-stu-id="9b72e-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="9b72e-159">Dağıtım Kılavuzu, bileşenlerinin dağıtılması ve test edilmesi için adım adım yönergeler sağlar.</span><span class="sxs-lookup"><span data-stu-id="9b72e-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>