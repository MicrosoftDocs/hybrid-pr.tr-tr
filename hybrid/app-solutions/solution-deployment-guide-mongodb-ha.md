---
title: Yüksek oranda kullanılabilir bir MongoDB çözümünü Azure 'a ve Azure Stack hub 'a dağıtma
description: Yüksek oranda kullanılabilir bir MongoDB çözümünü Azure 'a ve Azure Stack hub 'a dağıtmayı öğrenin
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 624f032def509d8e42d55807d72176e5fce85910
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901516"
---
# <a name="deploy-a-highly-available-mongodb-solution-across-two-azure-stack-hub-environments"></a>Yüksek oranda kullanılabilir bir MongoDB çözümünü iki Azure Stack hub ortamında dağıtın

Bu makalede, iki Azure Stack hub ortamında olağanüstü durum kurtarma (DR) sitesiyle temel yüksek kullanılabilirliğe sahip bir MongoDB kümesinin otomatik dağıtımı adım adım gösterilir. MongoDB ve yüksek kullanılabilirlik hakkında daha fazla bilgi edinmek için bkz. [çoğaltma kümesi üyeleri](https://docs.mongodb.com/manual/core/replica-set-members/).

Bu çözümde şu şekilde bir örnek ortam oluşturacaksınız:

> [!div class="checklist"]
> - İki Azure Stack hub arasında bir dağıtımı düzenleyin.
> - Azure API profilleriyle ilgili bağımlılık sorunlarını en aza indirmek için Docker 'ı kullanın.
> - Yüksek oranda kullanılabilir bir MongoDB kümesini olağanüstü durum kurtarma sitesiyle dağıtın.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub, Azure uzantısıdır. Azure Stack hub, bulut bilgi işlemin çevikliğini ve yeniliklerini şirket içi ortamınıza getirerek, karma uygulamaları her yerde derleyip dağıtmanıza imkan tanıyan tek karma bulutu etkinleştirir.  
> 
> [Karma uygulama tasarımı ile ilgili önemli noktalar](overview-app-design-considerations.md) , karma uygulamalar tasarlamak, dağıtmak ve çalıştırmak için yazılım kalitesinin (yerleştirme, ölçeklenebilirlik, kullanılabilirlik, dayanıklılık, yönetilebilirlik ve güvenlik) aynı şekilde gözden geçirmeleri inceler. Tasarım konuları karma uygulama tasarımını iyileştirirken, üretim ortamlarındaki zorlukları en aza indirmeyle ilgili olarak size yardımcı olur.

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a>Azure Stack hub ile MongoDB mimarisi

![Azure Stack hub 'da yüksek oranda kullanılabilir MongoDB mimarisi](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a>Azure Stack hub ile MongoDB önkoşulları

- İki bağlı Azure Stack hub tümleşik sistemi (Azure Stack hub). Bu dağıtım Azure Stack Geliştirme Seti (ASDK) üzerinde çalışmıyor. Azure Stack hub 'ı hakkında daha fazla bilgi edinmek için bkz. [Azure Stack hub nedir?](https://azure.microsoft.com/products/azure-stack/hub/)
  - Her Azure Stack hub 'ında kiracı aboneliği. 
  - **Her bir Azure Stack Hub için her abonelik KIMLIĞINI ve Azure Resource Manager uç noktasını bir yere unutmayın.**
- Her bir Azure Stack hub 'ındaki kiracı aboneliğine yönelik izinlere sahip bir Azure Active Directory (Azure AD) hizmet sorumlusu. Azure Stack hub 'Ları farklı Azure AD Kiracılarına karşı dağıtılırsa iki hizmet sorumlusu oluşturmanız gerekebilir. Azure Stack Hub için hizmet sorumlusu oluşturma hakkında bilgi edinmek için bkz. [Azure Stack hub kaynaklarına erişmek için uygulama kimliği kullanma](/azure-stack/user/azure-stack-create-service-principals).
  - **Her bir hizmet sorumlusunun uygulama KIMLIĞI, gizli anahtar ve kiracı adını (xxxxx.onmicrosoft.com) bir yere unutmayın.**
- Ubuntu 16,04 her bir Azure Stack hub 'ının Market 'e göre dağıtılmış. Market dağıtımı hakkında daha fazla bilgi için bkz. [Market öğelerini Azure Stack hub 'ına indirme](/azure-stack/operator/azure-stack-download-azure-marketplace-item).
- Yerel makinenizde yüklü [Docker for Windows](https://docs.docker.com/docker-for-windows/) .

## <a name="get-the-docker-image"></a>Docker görüntüsünü al

Her dağıtım için Docker görüntüleri farklı Azure PowerShell sürümleri arasındaki bağımlılık sorunlarını ortadan kaldırır.

1. Docker for Windows Windows kapsayıcıları ' nı kullandığınızdan emin olun.
2. Dağıtım betikleri ile Docker kapsayıcısını almak için yükseltilmiş bir komut isteminde aşağıdaki komutu çalıştırın.

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a>Kümeleri dağıtma

1. Kapsayıcı görüntüsü başarıyla çekildikten sonra görüntüyü başlatın.

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. Kapsayıcı başlatıldıktan sonra, kapsayıcıda yükseltilmiş bir PowerShell terminali vermiş olursunuz. Dağıtım betiğine ulaşmak için dizinleri değiştirin.

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. Dağıtımı çalıştırın. Gereken yerlerde kimlik bilgileri ve kaynak adları sağlayın. HA, HA kümesinin dağıtılacağı Azure Stack hub 'ına başvurur. DR, DR kümesinin dağıtılacağı Azure Stack hub 'ına başvurur.

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

4. `Y`NuGet sağlayıcısı 'nın yüklenmesine izin vermek için yazın ve bu, yüklenecek "2018-03-01-karma" MODÜLLERININ API profilini başlatabilir.

5. HA kaynakları ilk olarak dağıtılır. Dağıtımı izleyin ve bitmesini bekleyin. HA dağıtımının bittiğini belirten iletiyi aldıktan sonra, dağıtılan kaynakları görmek için HA Azure Stack hub 'ın portalını kontrol edebilirsiniz.

6. DR kaynaklarının dağıtımına devam edin ve kümeyle etkileşim kurmak için DR Azure Stack hub 'ında bir sıçrama kutusunu etkinleştirmek istediğinize karar verin.

7. DR Kaynak dağıtımının bitmesini bekleyin.

8. DR kaynak dağıtımı tamamlandıktan sonra kapsayıcıdan çıkın.

  ```powershell
  exit
  ```

## <a name="next-steps"></a>Sonraki adımlar

- DR Azure Stack hub 'ındaki bağlantı kutusu VM 'sini etkinleştirdiyseniz, Mongo CLı 'yı yükleyerek SSH aracılığıyla bağlanabilir ve MongoDB kümesiyle etkileşime geçebilirsiniz. MongoDB ile etkileşim kurma hakkında daha fazla bilgi için bkz. [Mongo kabuğu](https://docs.mongodb.com/manual/mongo/).
- Hibrit bulut uygulamaları hakkında daha fazla bilgi için bkz [. karma bulut çözümleri.](/azure-stack/user/)
- [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)'da kodu bu örnekle değiştirin.
