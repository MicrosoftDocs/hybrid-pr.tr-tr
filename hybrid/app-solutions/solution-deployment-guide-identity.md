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
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>Azure ve Azure Stack hub uygulamaları için karma bulut kimliğini yapılandırma

Azure ve Azure Stack hub uygulamalarınız için karma bulut kimliğini yapılandırmayı öğrenin.

Hem genel Azure hem de Azure Stack hub 'ında uygulamalarınıza erişim vermek için iki seçeneğiniz vardır.

 * Azure Stack hub 'ının internete sürekli bağlantısı olduğunda, Azure Active Directory (Azure AD) kullanabilirsiniz.
 * Azure Stack hub 'ı Internet bağlantısı kesildiğinde, Azure Directory Federasyon Hizmetleri 'ni (AD FS) kullanabilirsiniz.

Hizmet sorumlularını, Azure Stack hub 'daki Azure Resource Manager kullanarak dağıtım veya yapılandırma için Azure Stack hub uygulamalarınıza erişim vermek üzere kullanırsınız.

Bu çözümde, aşağıdakileri yapmak için bir örnek ortam oluşturacaksınız:

> [!div class="checklist"]
> - Küresel Azure ve Azure Stack hub 'da karma kimlik oluşturma
> - Azure Stack hub API 'sine erişmek için bir belirteç alın.

Bu çözümdeki adımlarda Azure Stack hub işleci izinlerinizin olması gerekir.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub, Azure uzantısıdır. Azure Stack hub, bulut bilgi işlemin çevikliğini ve yeniliklerini şirket içi ortamınıza getirerek, karma uygulamaları her yerde derleyip dağıtmanıza imkan tanıyan tek karma bulutu etkinleştirir.  
> 
> [Karma uygulama tasarımı ile ilgili önemli noktalar](overview-app-design-considerations.md) , karma uygulamalar tasarlamak, dağıtmak ve çalıştırmak için yazılım kalitesinin (yerleştirme, ölçeklenebilirlik, kullanılabilirlik, dayanıklılık, yönetilebilirlik ve güvenlik) aynı şekilde gözden geçirmeleri inceler. Tasarım konuları karma uygulama tasarımını iyileştirirken, üretim ortamlarındaki zorlukları en aza indirmeyle ilgili olarak size yardımcı olur.

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>Portalda Azure AD için bir hizmet sorumlusu oluşturma

Kimlik deposu olarak Azure AD 'yi kullanarak Azure Stack hub dağıttıysanız, Azure için yaptığınız gibi hizmet sorumluları oluşturabilirsiniz. [Kaynaklara erişmek için bir uygulama kimliği kullanmak](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) , bu adımları Portal üzerinden nasıl gerçekleştirekullanacağınızı gösterir. Başlamadan önce [gerekli Azure AD izinlerine](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) sahip olduğunuzdan emin olun.

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>PowerShell kullanarak AD FS için hizmet sorumlusu oluşturma

AD FS ile Azure Stack hub 'ı dağıttıysanız, bir hizmet sorumlusu oluşturmak, erişim için bir rol atamak ve bu kimliği kullanarak PowerShell 'den oturum açmak için PowerShell 'i kullanabilirsiniz. [Kaynaklara erişmek için bir uygulama kimliği kullanmak](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) , PowerShell kullanarak gerekli adımların nasıl gerçekleştirileceğini gösterir.

## <a name="using-the-azure-stack-hub-api"></a>Azure Stack hub API 'sini kullanma

[Azure Stack hub API](/azure-stack/user/azure-stack-rest-api-use.md) çözümü, Azure Stack hub API 'sine erişmek için belirteç alma sürecinde size yol gösterir.

## <a name="connect-to-azure-stack-hub-using-powershell"></a>PowerShell kullanarak Azure Stack hub 'a bağlanma

[Azure Stack hub 'Da PowerShell ile çalışmaya başlamanız için](/azure-stack/operator/azure-stack-powershell-install.md) hızlı başlangıç, Azure PowerShell yüklemek ve Azure Stack hub yüklemenize bağlanmak için gereken adımlarda size yol gösterir.

### <a name="prerequisites"></a>Ön koşullar

Azure AD 'ye erişebileceğiniz bir abonelikle bağlantılı bir Azure Stack hub yüklemesi gerekir. Azure Stack hub yüklemeniz yoksa, bu yönergeleri kullanarak bir [Azure Stack geliştirme seti (ASDK)](/azure-stack/asdk/asdk-install.md)ayarlayabilirsiniz.

#### <a name="connect-to-azure-stack-hub-using-code"></a>Kod kullanarak Azure Stack hub 'a bağlanma

Kodu kullanarak Azure Stack hub 'ına bağlanmak için, Azure Stack hub yüklemenizin kimlik doğrulama ve grafik uç noktalarını almak üzere Azure Resource Manager uç noktaları API 'sini kullanın. Ardından REST istekleri kullanarak kimlik doğrulaması yapın. [GitHub](https://github.com/shriramnat/HybridARMApplication)üzerinde örnek bir istemci uygulaması bulabilirsiniz.

>[!Note]
>Tercih ettiğiniz dilde Azure SDK 'sı Azure API profillerini destekliyorsa, SDK Azure Stack hub ile çalışmayabilir. Azure API profilleri hakkında daha fazla bilgi edinmek için [API sürüm profillerini yönetme](/azure-stack/user/azure-stack-version-profiles.md) makalesine bakın.

## <a name="next-steps"></a>Sonraki adımlar

- Azure Stack hub 'ında kimliğin nasıl işlendiği hakkında daha fazla bilgi için bkz. [Azure Stack hub Için kimlik mimarisi](/azure-stack/operator/azure-stack-identity-architecture.md).
- Azure bulut desenleri hakkında daha fazla bilgi edinmek için bkz. [bulut tasarım desenleri](https://docs.microsoft.com/azure/architecture/patterns).
