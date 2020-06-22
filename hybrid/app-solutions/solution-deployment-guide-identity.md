---
title: Azure ve Azure Stack hub uygulamaları için karma bulut kimliğini yapılandırma
description: Azure ve Azure Stack hub uygulamaları için karma bulut kimliğini yapılandırmayı öğrenin.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 1b3c683dd3e4a68413f83fd3cc129d6e6f594e1b
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84912043"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="fe5c8-103">Azure ve Azure Stack hub uygulamaları için karma bulut kimliğini yapılandırma</span><span class="sxs-lookup"><span data-stu-id="fe5c8-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="fe5c8-104">Azure ve Azure Stack hub uygulamalarınız için karma bulut kimliğini yapılandırmayı öğrenin.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="fe5c8-105">Hem genel Azure hem de Azure Stack hub 'ında uygulamalarınıza erişim vermek için iki seçeneğiniz vardır.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="fe5c8-106">Azure Stack hub 'ının internete sürekli bağlantısı olduğunda, Azure Active Directory (Azure AD) kullanabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="fe5c8-107">Azure Stack hub 'ı Internet bağlantısı kesildiğinde, Azure Directory Federasyon Hizmetleri 'ni (AD FS) kullanabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="fe5c8-108">Hizmet sorumlularını, Azure Stack hub 'daki Azure Resource Manager kullanarak dağıtım veya yapılandırma için Azure Stack hub uygulamalarınıza erişim vermek üzere kullanırsınız.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="fe5c8-109">Bu çözümde, aşağıdakileri yapmak için bir örnek ortam oluşturacaksınız:</span><span class="sxs-lookup"><span data-stu-id="fe5c8-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="fe5c8-110">Küresel Azure ve Azure Stack hub 'da karma kimlik oluşturma</span><span class="sxs-lookup"><span data-stu-id="fe5c8-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="fe5c8-111">Azure Stack hub API 'sine erişmek için bir belirteç alın.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="fe5c8-112">Bu çözümdeki adımlarda Azure Stack hub işleci izinlerinizin olması gerekir.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="fe5c8-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="fe5c8-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="fe5c8-114">Microsoft Azure Stack hub, Azure uzantısıdır.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="fe5c8-115">Azure Stack hub, bulut bilgi işlemin çevikliğini ve yeniliklerini şirket içi ortamınıza getirerek, karma uygulamaları her yerde derleyip dağıtmanıza imkan tanıyan tek karma bulutu etkinleştirir.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="fe5c8-116">[Karma uygulama tasarımı ile ilgili önemli noktalar](overview-app-design-considerations.md) , karma uygulamalar tasarlamak, dağıtmak ve çalıştırmak için yazılım kalitesinin (yerleştirme, ölçeklenebilirlik, kullanılabilirlik, dayanıklılık, yönetilebilirlik ve güvenlik) aynı şekilde gözden geçirmeleri inceler.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="fe5c8-117">Tasarım konuları karma uygulama tasarımını iyileştirirken, üretim ortamlarındaki zorlukları en aza indirmeyle ilgili olarak size yardımcı olur.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="fe5c8-118">Portalda Azure AD için bir hizmet sorumlusu oluşturma</span><span class="sxs-lookup"><span data-stu-id="fe5c8-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="fe5c8-119">Kimlik deposu olarak Azure AD 'yi kullanarak Azure Stack hub dağıttıysanız, Azure için yaptığınız gibi hizmet sorumluları oluşturabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="fe5c8-120">[Kaynaklara erişmek için bir uygulama kimliği kullanmak](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) , bu adımları Portal üzerinden nasıl gerçekleştirekullanacağınızı gösterir.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="fe5c8-121">Başlamadan önce [gerekli Azure AD izinlerine](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) sahip olduğunuzdan emin olun.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="fe5c8-122">PowerShell kullanarak AD FS için hizmet sorumlusu oluşturma</span><span class="sxs-lookup"><span data-stu-id="fe5c8-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="fe5c8-123">AD FS ile Azure Stack hub 'ı dağıttıysanız, bir hizmet sorumlusu oluşturmak, erişim için bir rol atamak ve bu kimliği kullanarak PowerShell 'den oturum açmak için PowerShell 'i kullanabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="fe5c8-124">[Kaynaklara erişmek için bir uygulama kimliği kullanmak](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) , PowerShell kullanarak gerekli adımların nasıl gerçekleştirileceğini gösterir.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="fe5c8-125">Azure Stack hub API 'sini kullanma</span><span class="sxs-lookup"><span data-stu-id="fe5c8-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="fe5c8-126">[Azure Stack hub API](/azure-stack/user/azure-stack-rest-api-use.md) çözümü, Azure Stack hub API 'sine erişmek için belirteç alma sürecinde size yol gösterir.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="fe5c8-127">PowerShell kullanarak Azure Stack hub 'a bağlanma</span><span class="sxs-lookup"><span data-stu-id="fe5c8-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="fe5c8-128">[Azure Stack hub 'Da PowerShell ile çalışmaya başlamanız için](/azure-stack/operator/azure-stack-powershell-install.md) hızlı başlangıç, Azure PowerShell yüklemek ve Azure Stack hub yüklemenize bağlanmak için gereken adımlarda size yol gösterir.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="fe5c8-129">Ön koşullar</span><span class="sxs-lookup"><span data-stu-id="fe5c8-129">Prerequisites</span></span>

<span data-ttu-id="fe5c8-130">Azure AD 'ye erişebileceğiniz bir abonelikle bağlantılı bir Azure Stack hub yüklemesi gerekir.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="fe5c8-131">Azure Stack hub yüklemeniz yoksa, bu yönergeleri kullanarak bir [Azure Stack geliştirme seti (ASDK)](/azure-stack/asdk/asdk-install.md)ayarlayabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="fe5c8-132">Kod kullanarak Azure Stack hub 'a bağlanma</span><span class="sxs-lookup"><span data-stu-id="fe5c8-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="fe5c8-133">Kodu kullanarak Azure Stack hub 'ına bağlanmak için, Azure Stack hub yüklemenizin kimlik doğrulama ve grafik uç noktalarını almak üzere Azure Resource Manager uç noktaları API 'sini kullanın.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="fe5c8-134">Ardından REST istekleri kullanarak kimlik doğrulaması yapın.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="fe5c8-135">[GitHub](https://github.com/shriramnat/HybridARMApplication)üzerinde örnek bir istemci uygulaması bulabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="fe5c8-136">Tercih ettiğiniz dilde Azure SDK 'sı Azure API profillerini destekliyorsa, SDK Azure Stack hub ile çalışmayabilir.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="fe5c8-137">Azure API profilleri hakkında daha fazla bilgi edinmek için [API sürüm profillerini yönetme](/azure-stack/user/azure-stack-version-profiles.md) makalesine bakın.</span><span class="sxs-lookup"><span data-stu-id="fe5c8-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles.md) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="fe5c8-138">Sonraki adımlar</span><span class="sxs-lookup"><span data-stu-id="fe5c8-138">Next steps</span></span>

- <span data-ttu-id="fe5c8-139">Azure Stack hub 'ında kimliğin nasıl işlendiği hakkında daha fazla bilgi için bkz. [Azure Stack hub Için kimlik mimarisi](/azure-stack/operator/azure-stack-identity-architecture.md).</span><span class="sxs-lookup"><span data-stu-id="fe5c8-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).</span></span>
- <span data-ttu-id="fe5c8-140">Azure bulut desenleri hakkında daha fazla bilgi edinmek için bkz. [bulut tasarım desenleri](https://docs.microsoft.com/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="fe5c8-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
