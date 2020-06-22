---
title: Çapraz bulutu ölçeklendirirken şirket içi verilerle karma uygulama dağıtma
description: Şirket içi verileri kullanan bir uygulamayı dağıtmayı ve Azure ve Azure Stack hub kullanarak çapraz bulutu ölçeklendireceğinizi öğrenin.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 75289eae902c5363862e345bdedb97cbcee0476e
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911858"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="feb07-103">Çapraz bulutu ölçeklendirirken şirket içi verilerle karma uygulama dağıtma</span><span class="sxs-lookup"><span data-stu-id="feb07-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="feb07-104">Bu çözüm kılavuzunda, hem Azure hem de Azure Stack hub 'ına yayılan ve tek bir şirket içi veri kaynağı kullanan bir karma uygulamanın nasıl dağıtılacağı gösterilir.</span><span class="sxs-lookup"><span data-stu-id="feb07-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="feb07-105">Karma bulut çözümünü kullanarak, özel bir bulutun uyumluluk avantajlarını genel bulutun ölçeklenebilirliği ile birleştirebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="feb07-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="feb07-106">Geliştiricileriniz ayrıca Microsoft Developer ekosisteminden yararlanabilir ve yeteneklerini buluta ve şirket içi ortamlara uygulayabilir.</span><span class="sxs-lookup"><span data-stu-id="feb07-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="feb07-107">Genel bakış ve varsayımlar</span><span class="sxs-lookup"><span data-stu-id="feb07-107">Overview and assumptions</span></span>

<span data-ttu-id="feb07-108">Geliştiricilerin aynı Web uygulamasını ortak bir buluta ve özel buluta dağıtmasına imkan tanıyan bir iş akışı ayarlamak için bu öğreticiyi izleyin.</span><span class="sxs-lookup"><span data-stu-id="feb07-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="feb07-109">Bu uygulama, özel bulutta barındırılan internet 'e ait olmayan yönlendirilebilir ağa erişebilir.</span><span class="sxs-lookup"><span data-stu-id="feb07-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="feb07-110">Bu Web uygulamaları izlenir ve trafikte ani bir artış olduğunda, bir program trafiği genel buluta yönlendirmek için DNS kayıtlarını değiştirir.</span><span class="sxs-lookup"><span data-stu-id="feb07-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="feb07-111">Trafik, ani bir düzeye düştüğünde trafik, özel buluta geri yönlendirilir.</span><span class="sxs-lookup"><span data-stu-id="feb07-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="feb07-112">Bu öğretici aşağıdaki görevleri kapsar:</span><span class="sxs-lookup"><span data-stu-id="feb07-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="feb07-113">Karma bağlantılı SQL Server veritabanı sunucusu dağıtın.</span><span class="sxs-lookup"><span data-stu-id="feb07-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="feb07-114">Küresel Azure 'da bir Web uygulamasını karma ağa bağlayın.</span><span class="sxs-lookup"><span data-stu-id="feb07-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="feb07-115">DNS 'i platformlar arası ölçekleme için yapılandırın.</span><span class="sxs-lookup"><span data-stu-id="feb07-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="feb07-116">Platformlar arası ölçeklendirme için SSL sertifikaları yapılandırın.</span><span class="sxs-lookup"><span data-stu-id="feb07-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="feb07-117">Web uygulamasını yapılandırın ve dağıtın.</span><span class="sxs-lookup"><span data-stu-id="feb07-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="feb07-118">Traffic Manager profili oluşturun ve bulut tabanlı ölçeklendirme için yapılandırın.</span><span class="sxs-lookup"><span data-stu-id="feb07-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="feb07-119">Artan trafik için Application Insights izlemeyi ve uyarı ayarlamayı ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="feb07-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="feb07-120">Genel Azure ve Azure Stack hub arasında otomatik trafik geçişini yapılandırın.</span><span class="sxs-lookup"><span data-stu-id="feb07-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="feb07-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="feb07-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="feb07-122">Microsoft Azure Stack hub, Azure uzantısıdır.</span><span class="sxs-lookup"><span data-stu-id="feb07-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="feb07-123">Azure Stack hub, bulut bilgi işlemin çevikliğini ve yeniliklerini şirket içi ortamınıza sunarak, karma uygulamaları her yerde derleyip dağıtmanıza imkan tanıyan tek karma bulutu etkinleştirir.</span><span class="sxs-lookup"><span data-stu-id="feb07-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="feb07-124">[Karma uygulama tasarımı ile ilgili önemli noktalar](overview-app-design-considerations.md) , karma uygulamalar tasarlamak, dağıtmak ve çalıştırmak için yazılım kalitesinin (yerleştirme, ölçeklenebilirlik, kullanılabilirlik, dayanıklılık, yönetilebilirlik ve güvenlik) aynı şekilde gözden geçirmeleri inceler.</span><span class="sxs-lookup"><span data-stu-id="feb07-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="feb07-125">Tasarım konuları karma uygulama tasarımını iyileştirirken, üretim ortamlarındaki zorlukları en aza indirmeyle ilgili olarak size yardımcı olur.</span><span class="sxs-lookup"><span data-stu-id="feb07-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="feb07-126">Varsayımlar</span><span class="sxs-lookup"><span data-stu-id="feb07-126">Assumptions</span></span>

<span data-ttu-id="feb07-127">Bu öğreticide, genel Azure ve Azure Stack hub ile ilgili temel bilgilere sahip olduğunuz varsayılmaktadır.</span><span class="sxs-lookup"><span data-stu-id="feb07-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="feb07-128">Öğreticiyi başlatmadan önce daha fazla bilgi edinmek istiyorsanız, şu makaleleri gözden geçirin:</span><span class="sxs-lookup"><span data-stu-id="feb07-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="feb07-129">Azure’a giriş</span><span class="sxs-lookup"><span data-stu-id="feb07-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="feb07-130">Azure Stack hub anahtar kavramları</span><span class="sxs-lookup"><span data-stu-id="feb07-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

<span data-ttu-id="feb07-131">Bu öğreticide Ayrıca bir Azure aboneliğiniz olduğunu varsaymaktadır.</span><span class="sxs-lookup"><span data-stu-id="feb07-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="feb07-132">Aboneliğiniz yoksa başlamadan önce [ücretsiz bir hesap oluşturun](https://azure.microsoft.com/free/) .</span><span class="sxs-lookup"><span data-stu-id="feb07-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="feb07-133">Ön koşullar</span><span class="sxs-lookup"><span data-stu-id="feb07-133">Prerequisites</span></span>

<span data-ttu-id="feb07-134">Bu çözüme başlamadan önce, aşağıdaki gereksinimleri karşıladığınızdan emin olun:</span><span class="sxs-lookup"><span data-stu-id="feb07-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="feb07-135">Bir Azure Stack Geliştirme Seti (ASDK) veya Azure Stack hub ile tümleşik sistemdeki bir abonelik.</span><span class="sxs-lookup"><span data-stu-id="feb07-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="feb07-136">ASDK 'yi dağıtmak için, [yükleyiciyi kullanarak asdk dağıtma](/azure-stack/asdk/asdk-install.md)bölümündeki yönergeleri izleyin.</span><span class="sxs-lookup"><span data-stu-id="feb07-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install.md).</span></span>
- <span data-ttu-id="feb07-137">Azure Stack hub yüklemenizde aşağıdakilerin yüklü olması gerekir:</span><span class="sxs-lookup"><span data-stu-id="feb07-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="feb07-138">Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="feb07-138">The Azure App Service.</span></span> <span data-ttu-id="feb07-139">Ortamınızdaki Azure App Service dağıtmak ve yapılandırmak için Azure Stack hub Işletmeniniz ile çalışın.</span><span class="sxs-lookup"><span data-stu-id="feb07-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="feb07-140">Bu öğreticide, App Service için en az bir (1) ayrılmış çalışan rolü olması gerekir.</span><span class="sxs-lookup"><span data-stu-id="feb07-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="feb07-141">Bir Windows Server 2016 görüntüsü.</span><span class="sxs-lookup"><span data-stu-id="feb07-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="feb07-142">Microsoft SQL Server bir görüntüyle Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="feb07-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="feb07-143">Uygun planlar ve teklifler.</span><span class="sxs-lookup"><span data-stu-id="feb07-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="feb07-144">Web uygulamanız için bir etki alanı adı.</span><span class="sxs-lookup"><span data-stu-id="feb07-144">A domain name for your web app.</span></span> <span data-ttu-id="feb07-145">Bir etki alanı adınız yoksa, GoDaddy, BlueHost ve Inmotion gibi bir etki alanı sağlayıcısından bir tane satın alabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="feb07-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="feb07-146">Letiniz gibi güvenilir bir sertifika yetkilisinden etki alanınız için bir SSL sertifikası.</span><span class="sxs-lookup"><span data-stu-id="feb07-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="feb07-147">SQL Server veritabanıyla iletişim kuran ve Application Insights destekleyen bir Web uygulaması.</span><span class="sxs-lookup"><span data-stu-id="feb07-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="feb07-148">[Dotnetcore-SQLDB-öğreticisi](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) örnek uygulamasını GitHub 'dan indirebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="feb07-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="feb07-149">Azure sanal ağı ile Azure Stack hub sanal ağı arasındaki karma ağ.</span><span class="sxs-lookup"><span data-stu-id="feb07-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="feb07-150">Ayrıntılı yönergeler için bkz. [Azure ve Azure Stack hub ile karma bulut bağlantısını yapılandırma](solution-deployment-guide-connectivity.md).</span><span class="sxs-lookup"><span data-stu-id="feb07-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="feb07-151">Azure Stack hub 'ında özel yapı aracısına sahip karma bir sürekli tümleştirme/sürekli dağıtım (CI/CD) işlem hattı.</span><span class="sxs-lookup"><span data-stu-id="feb07-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="feb07-152">Ayrıntılı yönergeler için bkz. [Azure ve Azure Stack hub uygulamalarıyla karma bulut kimliğini yapılandırma](solution-deployment-guide-identity.md).</span><span class="sxs-lookup"><span data-stu-id="feb07-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="feb07-153">Karma bağlantılı SQL Server veritabanı sunucusu dağıtma</span><span class="sxs-lookup"><span data-stu-id="feb07-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="feb07-154">Azure Stack hub Kullanıcı portalında oturum açın.</span><span class="sxs-lookup"><span data-stu-id="feb07-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="feb07-155">**Panoda** **Market**' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Azure Stack hub marketi](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="feb07-157">**Market**' te **işlem**' i seçin ve **daha sonra diğer**' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="feb07-158">**Daha fazla bilgi**Için, **Windows Server görüntüsünde ücretsiz SQL Server lisansı: SQL Server 2017 Developer** ' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Azure Stack hub Kullanıcı portalında bir sanal makine görüntüsü seçin](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="feb07-160">**Ücretsiz SQL Server Lisansı: Windows Server 'da SQL Server 2017 geliştiricisi** **Oluştur**' u seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="feb07-161">**Temel ayarları yapılandırma > temel bilgiler**, sanal makıne (VM) Için bir **ad** , SQL Server SA için BIR **Kullanıcı adı** ve SA için bir **parola** sağlayın.</span><span class="sxs-lookup"><span data-stu-id="feb07-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="feb07-162">**Abonelik** açılan listesinden, dağıtmakta olduğunuz aboneliği seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="feb07-163">**Kaynak grubu**için **var olanı seç** ' i kullanın ve VM 'yi Azure Stack Hub web uygulamanızla aynı kaynak grubuna yerleştirin.</span><span class="sxs-lookup"><span data-stu-id="feb07-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Azure Stack hub Kullanıcı portalında VM için temel ayarları yapılandırma](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="feb07-165">**Boyut**bölümünde VM 'niz için bir boyut seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="feb07-166">Bu öğretici için A2_Standard veya DS2_V2_Standard önerilir.</span><span class="sxs-lookup"><span data-stu-id="feb07-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="feb07-167">**Ayarlar > isteğe bağlı özellikleri yapılandırmak**için aşağıdaki ayarları yapılandırın:</span><span class="sxs-lookup"><span data-stu-id="feb07-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="feb07-168">**Depolama hesabı**: ihtiyacınız varsa yeni bir hesap oluşturun.</span><span class="sxs-lookup"><span data-stu-id="feb07-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="feb07-169">**Sanal ağ**:</span><span class="sxs-lookup"><span data-stu-id="feb07-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="feb07-170">SQL Server VM VPN ağ geçitleri ile aynı sanal ağda dağıtıldığından emin olun.</span><span class="sxs-lookup"><span data-stu-id="feb07-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="feb07-171">**Genel IP adresi**: varsayılan ayarları kullanın.</span><span class="sxs-lookup"><span data-stu-id="feb07-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="feb07-172">**Ağ güvenlik grubu**: (NSG).</span><span class="sxs-lookup"><span data-stu-id="feb07-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="feb07-173">Yeni bir NSG oluşturun.</span><span class="sxs-lookup"><span data-stu-id="feb07-173">Create a new NSG.</span></span>
   - <span data-ttu-id="feb07-174">**Uzantılar ve izleme**: varsayılan ayarları tutun.</span><span class="sxs-lookup"><span data-stu-id="feb07-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="feb07-175">**Tanılama depolama hesabı**: ihtiyacınız varsa yeni bir hesap oluşturun.</span><span class="sxs-lookup"><span data-stu-id="feb07-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="feb07-176">Yapılandırmanızı kaydetmek için **Tamam ' ı** seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-176">Select **OK** to save your configuration.</span></span>

     ![Azure Stack hub Kullanıcı portalında isteğe bağlı VM özelliklerini yapılandırma](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="feb07-178">**SQL Server ayarları**altında aşağıdaki ayarları yapılandırın:</span><span class="sxs-lookup"><span data-stu-id="feb07-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="feb07-179">**SQL bağlantısı**için **Genel (Internet)** seçeneğini belirleyin.</span><span class="sxs-lookup"><span data-stu-id="feb07-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="feb07-180">**Bağlantı noktası**için, varsayılan **1433**' ı koruyun.</span><span class="sxs-lookup"><span data-stu-id="feb07-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="feb07-181">**SQL kimlik doğrulaması**için **Etkinleştir**' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="feb07-182">SQL kimlik doğrulamasını etkinleştirdiğinizde, **temel**bilgiler ' de yapılandırdığınız "SQLAdmin" bilgileri ile otomatik olarak doldurulur.</span><span class="sxs-lookup"><span data-stu-id="feb07-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="feb07-183">Ayarların geri kalanı için varsayılan değerleri tutun.</span><span class="sxs-lookup"><span data-stu-id="feb07-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="feb07-184">**Tamam**’ı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-184">Select **OK**.</span></span>

     ![Azure Stack hub Kullanıcı portalında SQL Server ayarlarını yapılandırma](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="feb07-186">**Özet**SAYFASıNDA, VM yapılandırmasını gözden geçirin ve ardından dağıtımı başlatmak için **Tamam** ' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Azure Stack hub Kullanıcı portalında Yapılandırma Özeti](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="feb07-188">Yeni VM 'yi oluşturmak biraz zaman alır.</span><span class="sxs-lookup"><span data-stu-id="feb07-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="feb07-189">**Sanal makinelerde**VM 'nizin durumunu görüntüleyebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="feb07-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Azure Stack hub Kullanıcı portalında sanal makine durumu](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="feb07-191">Azure 'da ve Azure Stack hub 'da Web uygulamaları oluşturma</span><span class="sxs-lookup"><span data-stu-id="feb07-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="feb07-192">Azure App Service, bir Web uygulamasını çalıştırmayı ve yönetmeyi basitleştirir.</span><span class="sxs-lookup"><span data-stu-id="feb07-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="feb07-193">Azure Stack hub, Azure ile tutarlı olduğundan, App Service her iki ortamda da çalıştırılabilir.</span><span class="sxs-lookup"><span data-stu-id="feb07-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="feb07-194">Uygulamanızı barındırmak için App Service kullanacaksınız.</span><span class="sxs-lookup"><span data-stu-id="feb07-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="feb07-195">Web uygulamaları oluşturma</span><span class="sxs-lookup"><span data-stu-id="feb07-195">Create web apps</span></span>

1. <span data-ttu-id="feb07-196">Azure 'da [App Service planını yönetme](https://docs.microsoft.com/azure/app-service/app-service-plan-manage#create-an-app-service-plan)bölümündeki yönergeleri izleyerek Azure 'da bir Web uygulaması oluşturun.</span><span class="sxs-lookup"><span data-stu-id="feb07-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](https://docs.microsoft.com/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="feb07-197">Web uygulamasını karma ağınızla aynı aboneliğe ve kaynak grubuna yerleştirdiğinizden emin olun.</span><span class="sxs-lookup"><span data-stu-id="feb07-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="feb07-198">Azure Stack hub 'ında önceki adımı (1) yineleyin.</span><span class="sxs-lookup"><span data-stu-id="feb07-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="feb07-199">Azure Stack Hub için yol ekleme</span><span class="sxs-lookup"><span data-stu-id="feb07-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="feb07-200">Azure Stack hub üzerindeki App Service, kullanıcıların uygulamanıza erişmesine izin vermek için genel İnternet 'ten yönlendirilebilir olmalıdır.</span><span class="sxs-lookup"><span data-stu-id="feb07-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="feb07-201">Azure Stack hub 'ınız Internet 'ten erişilebiliyorsa, Azure Stack Hub web uygulaması için herkese açık IP adresini veya URL 'YI bir yere unutmayın.</span><span class="sxs-lookup"><span data-stu-id="feb07-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="feb07-202">Bir ASDK kullanıyorsanız, sanal ortam dışında App Service göstermek için [statik BIR NAT eşlemesi yapılandırabilirsiniz](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) .</span><span class="sxs-lookup"><span data-stu-id="feb07-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="feb07-203">Azure 'da bir Web uygulamasını karma ağa bağlama</span><span class="sxs-lookup"><span data-stu-id="feb07-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="feb07-204">Azure 'daki Web ön ucu ve Azure Stack hub 'ında SQL Server veritabanı arasında bağlantı sağlamak için, Web uygulamasının Azure ile Azure Stack hub arasında karma ağa bağlanması gerekir.</span><span class="sxs-lookup"><span data-stu-id="feb07-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="feb07-205">Bağlantıyı etkinleştirmek için şunları yapmanız gerekir:</span><span class="sxs-lookup"><span data-stu-id="feb07-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="feb07-206">Noktadan siteye bağlantıyı yapılandırın.</span><span class="sxs-lookup"><span data-stu-id="feb07-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="feb07-207">Web uygulamasını yapılandırın.</span><span class="sxs-lookup"><span data-stu-id="feb07-207">Configure the web app.</span></span>
- <span data-ttu-id="feb07-208">Azure Stack hub 'ında yerel ağ geçidini değiştirin.</span><span class="sxs-lookup"><span data-stu-id="feb07-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="feb07-209">Noktadan siteye bağlantı için Azure sanal ağını yapılandırma</span><span class="sxs-lookup"><span data-stu-id="feb07-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="feb07-210">Karma ağın Azure tarafındaki sanal ağ geçidi, Azure App Service ile tümleştirilecek Noktadan siteye bağlantılara izin vermelidir.</span><span class="sxs-lookup"><span data-stu-id="feb07-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="feb07-211">Azure 'da sanal ağ geçidi sayfasına gidin.</span><span class="sxs-lookup"><span data-stu-id="feb07-211">In Azure, go to the virtual network gateway page.</span></span> <span data-ttu-id="feb07-212">**Ayarlar**' ın altında, **Noktadan siteye yapılandırma**' yı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Azure sanal ağ geçidinde Noktadan siteye seçeneği](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="feb07-214">Noktadan siteye yapılandırmak için **Şimdi Yapılandır** ' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Azure sanal ağ geçidinde Noktadan siteye yapılandırmayı Başlat](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="feb07-216">**Noktadan siteye** yapılandırma sayfasında, **adres havuzunda**kullanmak istediğiniz özel IP adresi aralığını girin.</span><span class="sxs-lookup"><span data-stu-id="feb07-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="feb07-217">Belirttiğiniz aralığın, karma ağın genel Azure veya Azure Stack hub bileşenlerinde bulunan alt ağlar tarafından zaten kullanılmakta olan adres aralıklarıyla çakışmadığından emin olun.</span><span class="sxs-lookup"><span data-stu-id="feb07-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="feb07-218">**Tünel türü**altında, **IKEv2 VPN**seçeneğinin işaretini kaldırın.</span><span class="sxs-lookup"><span data-stu-id="feb07-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="feb07-219">Noktadan siteye yapılandırmayı sona **kazanmak Için kaydet** ' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Azure sanal ağ geçidinde Noktadan siteye ayarları](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="feb07-221">Azure App Service uygulamasını karma ağla tümleştirme</span><span class="sxs-lookup"><span data-stu-id="feb07-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="feb07-222">Uygulamayı Azure VNet 'e bağlamak için [Ağ Geçidi gerekli VNET tümleştirmesi](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration)bölümündeki yönergeleri izleyin.</span><span class="sxs-lookup"><span data-stu-id="feb07-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="feb07-223">Web uygulamasını barındıran App Service planına yönelik **Ayarlar** ' a gidin.</span><span class="sxs-lookup"><span data-stu-id="feb07-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="feb07-224">**Ayarlar**' da **ağ**' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-224">In **Settings**, select **Networking**.</span></span>

    ![App Service planı için ağı yapılandırma](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="feb07-226">**Sanal ağ tümleştirmesi**, **yönetmek Için buraya tıklayın ' ı**seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![App Service planı için VNET tümleştirmesini yönetme](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="feb07-228">Yapılandırmak istediğiniz VNET 'i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="feb07-229">**VNET 'e yönlendirilen IP adresleri**altında, Azure vnet, Azure Stack hub VNET ve Noktadan siteye adres ALANLARı için IP adresi aralığını girin.</span><span class="sxs-lookup"><span data-stu-id="feb07-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="feb07-230">Bu ayarları doğrulamak ve kaydetmek için **Kaydet** ' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-230">Select **Save** to validate and save these settings.</span></span>

    ![Sanal ağ tümleştirmesinde yönlendirileceği IP adresi aralıkları](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="feb07-232">App Service Azure sanal ağları ile tümleştirme hakkında daha fazla bilgi edinmek için bkz. [uygulamanızı bir Azure sanal ağı Ile tümleştirme](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet).</span><span class="sxs-lookup"><span data-stu-id="feb07-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="feb07-233">Azure Stack hub sanal ağını yapılandırma</span><span class="sxs-lookup"><span data-stu-id="feb07-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="feb07-234">Azure Stack hub sanal ağındaki yerel ağ geçidinin, trafiği App Service Noktadan siteye adres aralığından yönlendirmek üzere yapılandırılması gerekir.</span><span class="sxs-lookup"><span data-stu-id="feb07-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="feb07-235">Azure Stack hub 'ında **yerel ağ geçidi**' ne gidin.</span><span class="sxs-lookup"><span data-stu-id="feb07-235">In Azure Stack Hub, go to **Local network gateway**.</span></span> <span data-ttu-id="feb07-236">**Ayarlar** altında **Yapılandırma**'yı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-236">Under **Settings**, select **Configuration**.</span></span>

    ![Azure Stack Merkezi yerel ağ geçidinde ağ geçidi yapılandırma seçeneği](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="feb07-238">**Adres alanı**' nda, Azure 'daki sanal ağ geçidi için Noktadan siteye adres aralığını girin.</span><span class="sxs-lookup"><span data-stu-id="feb07-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Azure Stack hub yerel ağ geçidinde Noktadan siteye adres alanı](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="feb07-240">Yapılandırmayı doğrulamak ve kaydetmek için **Kaydet** ' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="feb07-241">DNS 'i platformlar arası ölçeklendirme için yapılandırma</span><span class="sxs-lookup"><span data-stu-id="feb07-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="feb07-242">Kullanıcılar, platformlar arası uygulamalar için düzgün şekilde yapılandırılarak, Web uygulamanızın Genel Azure ve Azure Stack hub örneklerine erişebilir.</span><span class="sxs-lookup"><span data-stu-id="feb07-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="feb07-243">Bu öğreticinin DNS yapılandırması, Yük arttıkça veya azaldıkça Azure Traffic Manager trafiği yönlendirmeye de olanak tanır.</span><span class="sxs-lookup"><span data-stu-id="feb07-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="feb07-244">Bu öğretici, DNS 'yi yönetmek için Azure DNS kullanır, çünkü App Service etki alanları çalışmıyor.</span><span class="sxs-lookup"><span data-stu-id="feb07-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="feb07-245">Alt etki alanları oluşturma</span><span class="sxs-lookup"><span data-stu-id="feb07-245">Create subdomains</span></span>

<span data-ttu-id="feb07-246">Traffic Manager DNS CNAMEs ' i kullandığından, trafiği uç noktalara doğru bir şekilde yönlendirmek için bir alt etki alanı gerekir.</span><span class="sxs-lookup"><span data-stu-id="feb07-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="feb07-247">DNS kayıtları ve etki alanı eşlemesi hakkında daha fazla bilgi için bkz. [Traffic Manager etki alanlarını eşleme](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span><span class="sxs-lookup"><span data-stu-id="feb07-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="feb07-248">Azure uç noktası için kullanıcıların Web uygulamanıza erişmek için kullanabileceği bir alt etki alanı oluşturacaksınız.</span><span class="sxs-lookup"><span data-stu-id="feb07-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="feb07-249">Bu öğretici için **app.Northwind.com**kullanabilir, ancak bu değeri kendi etki alanınızı temel alarak özelleştirmeniz gerekir.</span><span class="sxs-lookup"><span data-stu-id="feb07-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="feb07-250">Ayrıca, Azure Stack hub uç noktası için bir kayıt içeren bir alt etki alanı oluşturmanız gerekir.</span><span class="sxs-lookup"><span data-stu-id="feb07-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="feb07-251">**Azurestack.Northwind.com**kullanabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="feb07-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="feb07-252">Azure 'da özel bir etki alanı yapılandırma</span><span class="sxs-lookup"><span data-stu-id="feb07-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="feb07-253">[CNAME 'i Azure App Service ile eşleyerek](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record) **app.Northwind.com** ana bilgisayar adını Azure Web uygulamasına ekleyin.</span><span class="sxs-lookup"><span data-stu-id="feb07-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="feb07-254">Azure Stack hub 'da özel etki alanlarını yapılandırma</span><span class="sxs-lookup"><span data-stu-id="feb07-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="feb07-255">[Bir kaydı Azure App Service ile eşleyerek](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record)Azure Stack Hub web uygulamasına **azurestack.Northwind.com** ana bilgisayar adını ekleyin.</span><span class="sxs-lookup"><span data-stu-id="feb07-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="feb07-256">App Service uygulaması için internet yönlendirilebilir IP adresini kullanın.</span><span class="sxs-lookup"><span data-stu-id="feb07-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="feb07-257">[CNAME 'i Azure App Service ile eşleyerek](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)Azure Stack Hub web uygulamasına **app.Northwind.com** ana bilgisayar adını ekleyin.</span><span class="sxs-lookup"><span data-stu-id="feb07-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="feb07-258">Önceki adımda yapılandırdığınız ana bilgisayar adını (1) CNAME için hedef olarak kullanın.</span><span class="sxs-lookup"><span data-stu-id="feb07-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="feb07-259">Platformlar arası ölçeklendirme için SSL sertifikalarını yapılandırma</span><span class="sxs-lookup"><span data-stu-id="feb07-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="feb07-260">Web uygulamanız tarafından toplanan gizli verilerin, SQL veritabanında depolanan ve ' de yapılan aktarım sırasında güvenli olduğundan emin olmanız önemlidir.</span><span class="sxs-lookup"><span data-stu-id="feb07-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="feb07-261">Azure ve Azure Stack Hub web uygulamalarınızı tüm gelen trafik için SSL sertifikaları kullanacak şekilde yapılandıracaksınız.</span><span class="sxs-lookup"><span data-stu-id="feb07-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="feb07-262">Azure ve Azure Stack hub 'a SSL ekleme</span><span class="sxs-lookup"><span data-stu-id="feb07-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="feb07-263">Azure 'a SSL eklemek için:</span><span class="sxs-lookup"><span data-stu-id="feb07-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="feb07-264">Aldığınız SSL sertifikasının oluşturduğunuz alt etki alanı için geçerli olduğundan emin olun.</span><span class="sxs-lookup"><span data-stu-id="feb07-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="feb07-265">(Joker karakter sertifikaları kullanmak normaldir.)</span><span class="sxs-lookup"><span data-stu-id="feb07-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="feb07-266">Azure 'da, **Web uygulamanızı hazırlama** ve [var olan özel bir SSL sertifikasını Azure 'a](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) **bağlama bölümündeki** yönergeleri izleyin Web Apps makalesine gidin.</span><span class="sxs-lookup"><span data-stu-id="feb07-266">In Azure, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="feb07-267">**SSL türü**olarak **SNı tabanlı SSL** ' yi seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="feb07-268">Tüm trafiği HTTPS bağlantı noktasına yönlendir.</span><span class="sxs-lookup"><span data-stu-id="feb07-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="feb07-269">[Var olan bir özel SSL sertifikası 'nı Web Apps Azure 'A bağlama](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) makalesindeki **https 'yi zorla** bölümünde bulunan yönergeleri izleyin.</span><span class="sxs-lookup"><span data-stu-id="feb07-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="feb07-270">Azure Stack hub 'ına SSL eklemek için:</span><span class="sxs-lookup"><span data-stu-id="feb07-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="feb07-271">Azure için kullandığınız 1-3 adımlarını yineleyin.</span><span class="sxs-lookup"><span data-stu-id="feb07-271">Repeat steps 1-3 that you used for Azure.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="feb07-272">Web uygulamasını yapılandırma ve dağıtma</span><span class="sxs-lookup"><span data-stu-id="feb07-272">Configure and deploy the web app</span></span>

<span data-ttu-id="feb07-273">Uygulama kodunu, telemetri doğru Application Insights örneğine bildirmek ve Web uygulamalarını doğru bağlantı dizeleri ile yapılandırmak için yapılandıracaksınız.</span><span class="sxs-lookup"><span data-stu-id="feb07-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="feb07-274">Application Insights hakkında daha fazla bilgi edinmek için bkz. [Application Insights nedir?](https://docs.microsoft.com/azure/application-insights/app-insights-overview)</span><span class="sxs-lookup"><span data-stu-id="feb07-274">To learn more about Application Insights, see [What is Application Insights?](https://docs.microsoft.com/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="feb07-275">Application Insights Ekle</span><span class="sxs-lookup"><span data-stu-id="feb07-275">Add Application Insights</span></span>

1. <span data-ttu-id="feb07-276">Web uygulamanızı Microsoft Visual Studio açın.</span><span class="sxs-lookup"><span data-stu-id="feb07-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="feb07-277">Web trafiği arttıkça veya azaldıkça, Application Insights tarafından uyarı oluşturmak için kullanılan Telemetriyi iletmek üzere projenize [Application Insights ekleyin](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) .</span><span class="sxs-lookup"><span data-stu-id="feb07-277">[Add Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="feb07-278">Dinamik bağlantı dizelerini yapılandırma</span><span class="sxs-lookup"><span data-stu-id="feb07-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="feb07-279">Web uygulamasının her örneği SQL veritabanına bağlanmak için farklı bir yöntem kullanacaktır.</span><span class="sxs-lookup"><span data-stu-id="feb07-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="feb07-280">Azure 'daki uygulama SQL Server VM özel IP adresini kullanır ve Azure Stack hub 'ındaki uygulama SQL Server VM genel IP adresini kullanır.</span><span class="sxs-lookup"><span data-stu-id="feb07-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="feb07-281">Azure Stack hub ile tümleşik bir sistemde, genel IP adresi internet yönlendirilebilir olmamalıdır.</span><span class="sxs-lookup"><span data-stu-id="feb07-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="feb07-282">Bir ASDK 'de genel IP adresi, ASDK dışında yönlendirilebilir değildir.</span><span class="sxs-lookup"><span data-stu-id="feb07-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="feb07-283">App Service ortam değişkenlerini, uygulamanın her örneğine farklı bir bağlantı dizesi geçirmek için kullanabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="feb07-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="feb07-284">Uygulamayı Visual Studio 'da açın.</span><span class="sxs-lookup"><span data-stu-id="feb07-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="feb07-285">Startup.cs açın ve aşağıdaki kod bloğunu bulun:</span><span class="sxs-lookup"><span data-stu-id="feb07-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="feb07-286">Önceki kod bloğunu, dosyasında *appsettings.js* tanımlı bir bağlantı dizesi kullanan aşağıdaki kodla değiştirin:</span><span class="sxs-lookup"><span data-stu-id="feb07-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="feb07-287">App Service uygulama ayarlarını yapılandırma</span><span class="sxs-lookup"><span data-stu-id="feb07-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="feb07-288">Azure ve Azure Stack Hub için bağlantı dizeleri oluşturun.</span><span class="sxs-lookup"><span data-stu-id="feb07-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="feb07-289">Dizeler, kullanılan IP adresleri dışında, aynı olmalıdır.</span><span class="sxs-lookup"><span data-stu-id="feb07-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="feb07-290">Azure ve Azure Stack hub 'da, ad içinde ön ek olarak kullanarak, Web uygulamasında uygun bağlantı dizesini [bir uygulama ayarı olarak](https://docs.microsoft.com/azure/app-service/web-sites-configure) ekleyin `SQLCONNSTR\_` .</span><span class="sxs-lookup"><span data-stu-id="feb07-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](https://docs.microsoft.com/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="feb07-291">Web uygulaması ayarlarını **kaydedin** ve uygulamayı yeniden başlatın.</span><span class="sxs-lookup"><span data-stu-id="feb07-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="feb07-292">Küresel Azure 'da otomatik ölçeklendirmeyi etkinleştirme</span><span class="sxs-lookup"><span data-stu-id="feb07-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="feb07-293">Web uygulamanızı bir App Service ortamında oluşturduğunuzda, tek bir örnekle başlar.</span><span class="sxs-lookup"><span data-stu-id="feb07-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="feb07-294">Uygulamanız için daha fazla işlem kaynağı sağlamak üzere örnekleri eklemek için otomatik olarak ölçeği değiştirebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="feb07-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="feb07-295">Benzer şekilde, uygulamanızda otomatik olarak ölçeklendirme yapabilir ve uygulamanızın ihtiyaç duyacağı örnek sayısını azaltabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="feb07-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="feb07-296">Ölçek Genişletme ve ölçekleme yapılandırmak için bir App Service planına sahip olmanız gerekir.</span><span class="sxs-lookup"><span data-stu-id="feb07-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="feb07-297">Bir planınız yoksa, sonraki adımları başlatmadan önce bir tane oluşturun.</span><span class="sxs-lookup"><span data-stu-id="feb07-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="feb07-298">Otomatik ölçeklendirmeyi etkinleştir</span><span class="sxs-lookup"><span data-stu-id="feb07-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="feb07-299">Azure 'da, genişletmek istediğiniz siteler için App Service planı bulun ve ardından **genişleme (App Service planı)** öğesini seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-299">In Azure, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Ölçeği genişletme Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="feb07-301">**Otomatik ölçeklendirmeyi etkinleştir**' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-301">Select **Enable autoscale**.</span></span>

    ![Azure App Service otomatik ölçeklendirmeyi etkinleştir](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="feb07-303">**Otomatik ölçeklendirme ayarı adı**için bir ad girin.</span><span class="sxs-lookup"><span data-stu-id="feb07-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="feb07-304">**Varsayılan** otomatik ölçeklendirme kuralı için, **ölçüm temelinde ölçek**' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="feb07-305">**Örnek sınırlarını** **En düşük: 1**, **en fazla: 10**ve **Varsayılan: 1**olarak ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="feb07-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Azure App Service otomatik ölçeklendirmeyi yapılandırma](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="feb07-307">**+ Kural Ekle**' yi seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="feb07-308">**Ölçüm kaynağı**' nda **geçerli kaynak**' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="feb07-309">Kural için aşağıdaki ölçütleri ve eylemleri kullanın.</span><span class="sxs-lookup"><span data-stu-id="feb07-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="feb07-310">Ölçütler</span><span class="sxs-lookup"><span data-stu-id="feb07-310">Criteria</span></span>

1. <span data-ttu-id="feb07-311">Zaman toplama altında **Ortalama** **' ı** seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="feb07-312">**Ölçüm adı**altında **CPU yüzdesi**' ni seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="feb07-313">**İşleç**altında, **büyüktür**' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="feb07-314">**Eşiği** **50**olarak ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="feb07-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="feb07-315">**Süreyi** **10**olarak ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="feb07-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="feb07-316">Eylem</span><span class="sxs-lookup"><span data-stu-id="feb07-316">Action</span></span>

1. <span data-ttu-id="feb07-317">**İşlem**altında, **sayıyı göre artır**' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="feb07-318">**Örnek sayısını** **2**olarak ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="feb07-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="feb07-319">**Cool tuşuna** **5**olarak ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="feb07-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="feb07-320">**Ekle**'yi seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-320">Select **Add**.</span></span>

5. <span data-ttu-id="feb07-321">**+ Kural Ekle**' yi seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="feb07-322">**Ölçüm kaynağı**' nda **geçerli kaynak** ' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="feb07-323">Geçerli kaynak App Service planınızın adını/GUID 'sini, **kaynak türü** ve **kaynak** açılan listelerini de kullanılamaz hale alacak.</span><span class="sxs-lookup"><span data-stu-id="feb07-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="feb07-324">Otomatik ölçeklendirmeyi etkinleştir</span><span class="sxs-lookup"><span data-stu-id="feb07-324">Enable automatic scale in</span></span>

<span data-ttu-id="feb07-325">Trafik azaldıkça Azure Web uygulaması, maliyetleri azaltmak için etkin örnek sayısını otomatik olarak azaltabilir.</span><span class="sxs-lookup"><span data-stu-id="feb07-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="feb07-326">Bu eylem, genişleme sıkından daha az agresif ve uygulama kullanıcıları üzerindeki etkiyi en aza indirir.</span><span class="sxs-lookup"><span data-stu-id="feb07-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="feb07-327">**Varsayılan** genişleme durumuna gidin, ardından **+ Kural Ekle**' yi seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="feb07-328">Kural için aşağıdaki ölçütleri ve eylemleri kullanın.</span><span class="sxs-lookup"><span data-stu-id="feb07-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="feb07-329">Ölçütler</span><span class="sxs-lookup"><span data-stu-id="feb07-329">Criteria</span></span>

1. <span data-ttu-id="feb07-330">Zaman toplama altında **Ortalama** **' ı** seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="feb07-331">**Ölçüm adı**altında **CPU yüzdesi**' ni seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="feb07-332">**İşleç** **altında küçüktür ' i seçin.**</span><span class="sxs-lookup"><span data-stu-id="feb07-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="feb07-333">**Eşiği** **30**olarak ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="feb07-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="feb07-334">**Süreyi** **10**olarak ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="feb07-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="feb07-335">Eylem</span><span class="sxs-lookup"><span data-stu-id="feb07-335">Action</span></span>

1. <span data-ttu-id="feb07-336">**İşlem**altında **sayıyı göre azalt**' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="feb07-337">**Örnek sayısını** **1**olarak ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="feb07-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="feb07-338">**Cool tuşuna** **5**olarak ayarlayın.</span><span class="sxs-lookup"><span data-stu-id="feb07-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="feb07-339">**Ekle**'yi seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="feb07-340">Traffic Manager profili oluşturma ve platformlar arası ölçeklendirmeyi yapılandırma</span><span class="sxs-lookup"><span data-stu-id="feb07-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="feb07-341">Azure 'da bir Traffic Manager profili oluşturun ve sonra da platformlar arası ölçeklendirmeyi etkinleştirmek için uç noktaları yapılandırın.</span><span class="sxs-lookup"><span data-stu-id="feb07-341">Create a Traffic Manager profile in Azure and then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="feb07-342">Traffic Manager profili oluşturma</span><span class="sxs-lookup"><span data-stu-id="feb07-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="feb07-343">**Kaynak oluştur**’u seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="feb07-344">**Ağ**'ı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-344">Select **Networking**.</span></span>
3. <span data-ttu-id="feb07-345">**Traffic Manager profil** ' i seçin ve aşağıdaki ayarları yapılandırın:</span><span class="sxs-lookup"><span data-stu-id="feb07-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="feb07-346">**Ad**alanına profiliniz için bir ad girin.</span><span class="sxs-lookup"><span data-stu-id="feb07-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="feb07-347">Bu ad trafficmanager.net bölgesinde benzersiz **olmalıdır** ve yenı bir DNS adı oluşturmak için kullanılır (örneğin, northwindstore.trafficmanager.net).</span><span class="sxs-lookup"><span data-stu-id="feb07-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="feb07-348">**Yönlendirme yöntemi**için **ağırlıklı**' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="feb07-349">**Abonelik**için, bu profili oluşturmak istediğiniz aboneliği seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="feb07-350">**Kaynak grubu**' nda, bu profil için yeni bir kaynak grubu oluşturun.</span><span class="sxs-lookup"><span data-stu-id="feb07-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="feb07-351">**Kaynak grubu konumu** alanında kaynak grubunun konumunu seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="feb07-352">Bu ayar, kaynak grubunun konumunu ifade eder ve genel olarak dağıtılan Traffic Manager profilini etkilemez.</span><span class="sxs-lookup"><span data-stu-id="feb07-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="feb07-353">**Oluştur**'u seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-353">Select **Create**.</span></span>

    ![Traffic Manager profili oluşturma](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="feb07-355">Traffic Manager profilinizin genel dağıtımı tamamlandığında, altında oluşturduğunuz kaynak grubu için kaynak listesinde gösterilir.</span><span class="sxs-lookup"><span data-stu-id="feb07-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="feb07-356">Traffic Manager uç noktalarını ekleme</span><span class="sxs-lookup"><span data-stu-id="feb07-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="feb07-357">Oluşturduğunuz Traffic Manager profilini arayın.</span><span class="sxs-lookup"><span data-stu-id="feb07-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="feb07-358">Profil için kaynak grubuna gezindiyseniz, profili seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="feb07-359">**Traffic Manager profilinde**, **Ayarlar**altında **uç noktalar**' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="feb07-360">**Ekle**'yi seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-360">Select **Add**.</span></span>

4. <span data-ttu-id="feb07-361">**Uç nokta Ekle**' de Azure Stack Hub için aşağıdaki ayarları kullanın:</span><span class="sxs-lookup"><span data-stu-id="feb07-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="feb07-362">**Tür**için **dış uç nokta**' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="feb07-363">Uç nokta için bir **ad** girin.</span><span class="sxs-lookup"><span data-stu-id="feb07-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="feb07-364">**Tam etki alanı adı (FQDN) veya IP**için, Azure Stack Hub web uygulamanızın dış URL 'sini girin.</span><span class="sxs-lookup"><span data-stu-id="feb07-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="feb07-365">**Ağırlık**için varsayılan değer olan **1**' i tutun.</span><span class="sxs-lookup"><span data-stu-id="feb07-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="feb07-366">Bu ağırlık, sağlıklı ise bu uç noktaya giden tüm trafik ile sonuçlanır.</span><span class="sxs-lookup"><span data-stu-id="feb07-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="feb07-367">**Eklemeyi devre dışı** bırak işareti kaldırılmış olarak bırakın.</span><span class="sxs-lookup"><span data-stu-id="feb07-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="feb07-368">Azure Stack hub uç noktasını kaydetmek için **Tamam ' ı** seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="feb07-369">Sonraki Azure uç noktasını yapılandıracaksınız.</span><span class="sxs-lookup"><span data-stu-id="feb07-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="feb07-370">**Traffic Manager profilinde** **uç noktalar**' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="feb07-371">**+ Ekle**' yi seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-371">Select **+Add**.</span></span>
3. <span data-ttu-id="feb07-372">**Uç nokta Ekle**sayfasında Azure için aşağıdaki ayarları kullanın:</span><span class="sxs-lookup"><span data-stu-id="feb07-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="feb07-373">**Tür**için **Azure uç noktası**' nı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="feb07-374">Uç nokta için bir **ad** girin.</span><span class="sxs-lookup"><span data-stu-id="feb07-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="feb07-375">**Hedef kaynak türü**için **App Service**seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="feb07-376">**Hedef kaynak**için, aynı abonelikte Web Apps listesini görmek üzere **bir App Service seçin** öğesini seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="feb07-377">**Kaynak** bölümünde ilk uç nokta olarak kullanmak istediğiniz uygulama hizmetini seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="feb07-378">**Ağırlık**için **2**' yi seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="feb07-379">Bu ayar, birincil uç noktanın sağlıksız olması durumunda ya da tetiklendiğinde trafiği yeniden yönlendiren bir kuralınız/uyarıınız varsa, bu uç noktaya giden tüm trafiğe neden olur.</span><span class="sxs-lookup"><span data-stu-id="feb07-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="feb07-380">**Eklemeyi devre dışı** bırak işareti kaldırılmış olarak bırakın.</span><span class="sxs-lookup"><span data-stu-id="feb07-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="feb07-381">Azure uç noktasını kaydetmek için **Tamam ' ı** seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="feb07-382">Her iki uç nokta yapılandırıldıktan sonra, **uç noktalar**' ı seçerken **Traffic Manager profilde** listelenir.</span><span class="sxs-lookup"><span data-stu-id="feb07-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="feb07-383">Aşağıdaki ekran yakalamadaki örnek, her biri için durum ve yapılandırma bilgileriyle iki uç nokta gösterir.</span><span class="sxs-lookup"><span data-stu-id="feb07-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Traffic Manager profilindeki uç noktalar](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a><span data-ttu-id="feb07-385">Application Insights izlemeyi ve uyarı ayarlamayı ayarlama</span><span class="sxs-lookup"><span data-stu-id="feb07-385">Set up Application Insights monitoring and alerting</span></span>

<span data-ttu-id="feb07-386">Azure Application Insights, uygulamanızı izlemenizi ve yapılandırdığınız koşullara göre uyarı göndermenizi sağlar.</span><span class="sxs-lookup"><span data-stu-id="feb07-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="feb07-387">Bazı örnekler şunlardır: uygulama kullanılamıyor, hatalarla karşılaşıyor veya performans sorunlarını gösteriyor.</span><span class="sxs-lookup"><span data-stu-id="feb07-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="feb07-388">Uyarı oluşturmak için Application Insights ölçümleri kullanacaksınız.</span><span class="sxs-lookup"><span data-stu-id="feb07-388">You'll use Application Insights metrics to create alerts.</span></span> <span data-ttu-id="feb07-389">Bu uyarılar tetiklenmesi durumunda, Web uygulamanızın örneği ölçeği genişletmek için otomatik olarak Azure Stack hub 'dan Azure 'a geçiş yapar ve sonra ölçeklendirmek üzere Azure Stack hub 'a geri yüklenir.</span><span class="sxs-lookup"><span data-stu-id="feb07-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="feb07-390">Ölçülerden uyarı oluşturma</span><span class="sxs-lookup"><span data-stu-id="feb07-390">Create an alert from metrics</span></span>

<span data-ttu-id="feb07-391">Bu öğretici için kaynak grubuna gidin ve **Application Insights**açmak için Application Insights örneğini seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-391">Go to the resource group for this tutorial and then select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="feb07-393">Bu görünümü, bir genişleme uyarısı ve bir ölçek genişletme uyarısı oluşturmak için kullanacaksınız.</span><span class="sxs-lookup"><span data-stu-id="feb07-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="feb07-394">Genişleme uyarısı oluşturma</span><span class="sxs-lookup"><span data-stu-id="feb07-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="feb07-395">**Yapılandır**altında **Uyarılar (klasik)** öğesini seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="feb07-396">**Ölçüm uyarısı Ekle (klasik)** seçeneğini belirleyin.</span><span class="sxs-lookup"><span data-stu-id="feb07-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="feb07-397">**Kural Ekle**' de, aşağıdaki ayarları yapılandırın:</span><span class="sxs-lookup"><span data-stu-id="feb07-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="feb07-398">**Ad**Için **Azure buluta veri bloğu**girin.</span><span class="sxs-lookup"><span data-stu-id="feb07-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="feb07-399">**Açıklama** isteğe bağlıdır.</span><span class="sxs-lookup"><span data-stu-id="feb07-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="feb07-400">**Kaynak**  >  **uyarısı**' nın altında, **ölçümler**' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="feb07-401">**Ölçüt**altında, aboneliğinizi, Traffic Manager profilinizin kaynak grubunu ve kaynak için Traffic Manager profilinin adını seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="feb07-402">**Ölçüm**Için **istek hızı**' nı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="feb07-403">**Koşul**Için **şundan büyüktür**' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="feb07-404">**Eşik**için **2**girin.</span><span class="sxs-lookup"><span data-stu-id="feb07-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="feb07-405">**Süre**için **son 5 dakikayı**seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="feb07-406">**Bildirim**altında:</span><span class="sxs-lookup"><span data-stu-id="feb07-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="feb07-407">**E-posta sahipleri, katkıda bulunanlar ve okuyucular**için onay kutusunu işaretleyin.</span><span class="sxs-lookup"><span data-stu-id="feb07-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="feb07-408">**Ek yönetici e-postaları**için e-posta adresinizi girin.</span><span class="sxs-lookup"><span data-stu-id="feb07-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="feb07-409">Menü çubuğunda **Kaydet**' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="feb07-410">Ölçek Genişletme uyarısını oluşturma</span><span class="sxs-lookup"><span data-stu-id="feb07-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="feb07-411">**Yapılandır**altında **Uyarılar (klasik)** öğesini seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="feb07-412">**Ölçüm uyarısı Ekle (klasik)** seçeneğini belirleyin.</span><span class="sxs-lookup"><span data-stu-id="feb07-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="feb07-413">**Kural Ekle**' de, aşağıdaki ayarları yapılandırın:</span><span class="sxs-lookup"><span data-stu-id="feb07-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="feb07-414">**Ad**için **ölçeği Azure Stack hub 'a geri**girin.</span><span class="sxs-lookup"><span data-stu-id="feb07-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="feb07-415">**Açıklama** isteğe bağlıdır.</span><span class="sxs-lookup"><span data-stu-id="feb07-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="feb07-416">**Kaynak**  >  **uyarısı**' nın altında, **ölçümler**' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="feb07-417">**Ölçüt**altında, aboneliğinizi, Traffic Manager profilinizin kaynak grubunu ve kaynak için Traffic Manager profilinin adını seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="feb07-418">**Ölçüm**Için **istek hızı**' nı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="feb07-419">**Koşul** **için küçüktür ' i seçin.**</span><span class="sxs-lookup"><span data-stu-id="feb07-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="feb07-420">**Eşik**için **2**girin.</span><span class="sxs-lookup"><span data-stu-id="feb07-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="feb07-421">**Süre**için **son 5 dakikayı**seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="feb07-422">**Bildirim**altında:</span><span class="sxs-lookup"><span data-stu-id="feb07-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="feb07-423">**E-posta sahipleri, katkıda bulunanlar ve okuyucular**için onay kutusunu işaretleyin.</span><span class="sxs-lookup"><span data-stu-id="feb07-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="feb07-424">**Ek yönetici e-postaları**için e-posta adresinizi girin.</span><span class="sxs-lookup"><span data-stu-id="feb07-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="feb07-425">Menü çubuğunda **Kaydet**' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="feb07-426">Aşağıdaki ekran görüntüsünde, genişleme ve ölçek genişletme uyarıları gösterilmektedir.</span><span class="sxs-lookup"><span data-stu-id="feb07-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Application Insights uyarılar (klasik)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="feb07-428">Azure ile Azure Stack hub arasında trafiği yeniden yönlendirme</span><span class="sxs-lookup"><span data-stu-id="feb07-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="feb07-429">Azure ile Azure Stack hub arasında Web uygulaması trafiğiniz için el ile veya otomatik geçişi yapılandırabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="feb07-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="feb07-430">Azure ile Azure Stack hub arasında el ile geçiş yapılandırma</span><span class="sxs-lookup"><span data-stu-id="feb07-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="feb07-431">Web siteniz yapılandırdığınız eşiklere ulaştığında bir uyarı alırsınız.</span><span class="sxs-lookup"><span data-stu-id="feb07-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="feb07-432">Trafiği Azure 'a el ile yönlendirmek için aşağıdaki adımları kullanın.</span><span class="sxs-lookup"><span data-stu-id="feb07-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="feb07-433">Azure portal Traffic Manager profilinizi seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Azure portal uç noktaları Traffic Manager](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="feb07-435">**Uç noktaları**seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="feb07-436">**Azure uç noktasını**seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="feb07-437">**Durum**altında, **etkin**' i seçin ve ardından **Kaydet**' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Azure portal 'de Azure uç noktasını etkinleştirme](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="feb07-439">Traffic Manager profilinin **uç noktalarında** **dış uç nokta**' ı seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="feb07-440">**Durum**altında **devre dışı**' yı seçin ve ardından **Kaydet**' i seçin.</span><span class="sxs-lookup"><span data-stu-id="feb07-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Azure portal Azure Stack hub uç noktasını devre dışı bırak](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="feb07-442">Uç noktalar yapılandırıldıktan sonra, uygulama trafiği Azure Stack Hub web uygulaması yerine Azure genişleme Web uygulamanıza gider.</span><span class="sxs-lookup"><span data-stu-id="feb07-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Azure Web uygulaması trafiği 'nde uç noktalar değişti](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="feb07-444">Akışı Azure Stack hub 'a geri çevirmek için önceki adımları kullanın:</span><span class="sxs-lookup"><span data-stu-id="feb07-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="feb07-445">Azure Stack hub uç noktasını etkinleştirin.</span><span class="sxs-lookup"><span data-stu-id="feb07-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="feb07-446">Azure uç noktasını devre dışı bırakın.</span><span class="sxs-lookup"><span data-stu-id="feb07-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="feb07-447">Azure ile Azure Stack hub arasında otomatik geçiş yapılandırma</span><span class="sxs-lookup"><span data-stu-id="feb07-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="feb07-448">Uygulamanız Azure Işlevleri tarafından sunulan [sunucusuz](https://azure.microsoft.com/overview/serverless-computing/) bir ortamda çalıştırılıyorsa Application Insights izlemeyi da kullanabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="feb07-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="feb07-449">Bu senaryoda, bir işlev uygulaması çağıran bir Web kancasını kullanmak için Application Insights yapılandırabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="feb07-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="feb07-450">Bu uygulama, bir uyarıya yanıt olarak bir uç noktayı otomatik olarak sağlar veya devre dışı bırakır.</span><span class="sxs-lookup"><span data-stu-id="feb07-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="feb07-451">Otomatik trafik geçişini yapılandırmak için aşağıdaki adımları kılavuz olarak kullanın.</span><span class="sxs-lookup"><span data-stu-id="feb07-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="feb07-452">Bir Azure Işlev uygulaması oluşturun.</span><span class="sxs-lookup"><span data-stu-id="feb07-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="feb07-453">HTTP ile tetiklenen bir işlev oluşturun.</span><span class="sxs-lookup"><span data-stu-id="feb07-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="feb07-454">Kaynak Yöneticisi, Web Apps ve Traffic Manager için Azure SDK 'larını içeri aktarın.</span><span class="sxs-lookup"><span data-stu-id="feb07-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="feb07-455">Kodu şu şekilde geliştirin:</span><span class="sxs-lookup"><span data-stu-id="feb07-455">Develop code to:</span></span>

   - <span data-ttu-id="feb07-456">Azure aboneliğinizde kimlik doğrulaması yapın.</span><span class="sxs-lookup"><span data-stu-id="feb07-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="feb07-457">Traffic Manager uç noktalarına trafiği Azure 'a veya Azure Stack hub 'a yönlendirmek için geçiş yapan bir parametre kullanın.</span><span class="sxs-lookup"><span data-stu-id="feb07-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="feb07-458">Kodunuzu kaydedin ve işlev uygulamasının URL 'sini, Application Insights uyarı kuralı ayarlarının **Web kancası** bölümüne uygun parametrelerle ekleyin.</span><span class="sxs-lookup"><span data-stu-id="feb07-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="feb07-459">Application Insights bir uyarı tetiklendiğinde trafik otomatik olarak yönlendirilir.</span><span class="sxs-lookup"><span data-stu-id="feb07-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="feb07-460">Sonraki adımlar</span><span class="sxs-lookup"><span data-stu-id="feb07-460">Next steps</span></span>

- <span data-ttu-id="feb07-461">Azure bulut desenleri hakkında daha fazla bilgi edinmek için bkz. [bulut tasarım desenleri](https://docs.microsoft.com/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="feb07-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
