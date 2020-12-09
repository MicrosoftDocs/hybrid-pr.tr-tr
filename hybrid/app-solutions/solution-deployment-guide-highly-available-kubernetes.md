---
title: Azure Stack hub 'da yüksek oranda kullanılabilir Kubernetes kümesi dağıtma
description: Azure ve Azure Stack hub kullanarak yüksek kullanılabilirlik için bir Kubernetes küme çözümünü dağıtmayı öğrenin.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911740"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="125c9-103">Azure Stack hub 'da yüksek kullanılabilirliğe sahip bir Kubernetes kümesi dağıtma</span><span class="sxs-lookup"><span data-stu-id="125c9-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="125c9-104">Bu makalede, farklı fiziksel konumlarda birden çok Azure Stack hub örneğine dağıtılan yüksek oranda kullanılabilir bir Kubernetes küme ortamı oluşturma işleminin nasıl yapılacağı gösterilir.</span><span class="sxs-lookup"><span data-stu-id="125c9-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="125c9-105">Bu çözüm dağıtımı kılavuzunda, şunları nasıl yapacağınızı öğreneceksiniz:</span><span class="sxs-lookup"><span data-stu-id="125c9-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="125c9-106">AKS altyapısını indirme ve hazırlama</span><span class="sxs-lookup"><span data-stu-id="125c9-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="125c9-107">AKS motoru Yardımcısı VM 'sine bağlanma</span><span class="sxs-lookup"><span data-stu-id="125c9-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="125c9-108">Kubernetes kümesi dağıtma</span><span class="sxs-lookup"><span data-stu-id="125c9-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="125c9-109">Kubernetes kümesine bağlanma</span><span class="sxs-lookup"><span data-stu-id="125c9-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="125c9-110">Kubernetes kümesine Azure Pipelines bağlama</span><span class="sxs-lookup"><span data-stu-id="125c9-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="125c9-111">İzlemeyi yapılandırma</span><span class="sxs-lookup"><span data-stu-id="125c9-111">Configure monitoring</span></span>
> - <span data-ttu-id="125c9-112">Uygulamayı dağıtma</span><span class="sxs-lookup"><span data-stu-id="125c9-112">Deploy application</span></span>
> - <span data-ttu-id="125c9-113">Otomatik ölçeklendirme uygulaması</span><span class="sxs-lookup"><span data-stu-id="125c9-113">Autoscale application</span></span>
> - <span data-ttu-id="125c9-114">Traffic Manager'ı Yapılandırma</span><span class="sxs-lookup"><span data-stu-id="125c9-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="125c9-115">Kubernetes’i yükseltme</span><span class="sxs-lookup"><span data-stu-id="125c9-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="125c9-116">Kubernetes 'i ölçeklendirme</span><span class="sxs-lookup"><span data-stu-id="125c9-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="125c9-117">![Karma paragraf](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="125c9-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="125c9-118">Microsoft Azure Stack hub, Azure uzantısıdır.</span><span class="sxs-lookup"><span data-stu-id="125c9-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="125c9-119">Azure Stack hub, bulut bilgi işlemin çevikliğini ve yeniliklerini şirket içi ortamınıza sunarak, karma uygulamaları her yerde derleyip dağıtmanıza imkan tanıyan tek karma bulutu etkinleştirir.</span><span class="sxs-lookup"><span data-stu-id="125c9-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="125c9-120">[Karma uygulama tasarımı ile ilgili önemli noktalar](overview-app-design-considerations.md) , karma uygulamalar tasarlamak, dağıtmak ve çalıştırmak için yazılım kalitesinin (yerleştirme, ölçeklenebilirlik, kullanılabilirlik, dayanıklılık, yönetilebilirlik ve güvenlik) aynı şekilde gözden geçirmeleri inceler.</span><span class="sxs-lookup"><span data-stu-id="125c9-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="125c9-121">Tasarım konuları karma uygulama tasarımını iyileştirirken, üretim ortamlarındaki zorlukları en aza indirmeyle ilgili olarak size yardımcı olur.</span><span class="sxs-lookup"><span data-stu-id="125c9-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="125c9-122">Önkoşullar</span><span class="sxs-lookup"><span data-stu-id="125c9-122">Prerequisites</span></span>

<span data-ttu-id="125c9-123">Bu dağıtım kılavuzunu kullanmaya başlamadan önce şunları yaptığınızdan emin olun:</span><span class="sxs-lookup"><span data-stu-id="125c9-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="125c9-124">[Yüksek kullanılabilirlik Kubernetes kümesi örüntüsünün](pattern-highly-available-kubernetes.md) makalesini gözden geçirin.</span><span class="sxs-lookup"><span data-stu-id="125c9-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="125c9-125">Bu makalede başvurulan ek varlıkları içeren [yardımcı GitHub deposunun](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)içeriğini gözden geçirin.</span><span class="sxs-lookup"><span data-stu-id="125c9-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="125c9-126">En az ["katkıda bulunan" izinlerle](/azure-stack/user/azure-stack-manage-permissions) [Azure Stack hub Kullanıcı portalına](/azure-stack/user/azure-stack-use-portal)erişebilen bir hesabınız olmalıdır.</span><span class="sxs-lookup"><span data-stu-id="125c9-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="125c9-127">AKS altyapısını indirme ve hazırlama</span><span class="sxs-lookup"><span data-stu-id="125c9-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="125c9-128">AKS altyapısı, Azure Stack hub Azure Resource Manager uç noktalarına ulaşan herhangi bir Windows veya Linux ana bilgisayardan kullanılabilen bir ikili dosyaysa.</span><span class="sxs-lookup"><span data-stu-id="125c9-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="125c9-129">Bu kılavuzda Azure Stack hub 'da yeni bir Linux (veya Windows) VM dağıtımı açıklanmaktadır.</span><span class="sxs-lookup"><span data-stu-id="125c9-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="125c9-130">AKS altyapısı Kubernetes kümelerini dağıttığında daha sonra kullanılacaktır.</span><span class="sxs-lookup"><span data-stu-id="125c9-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="125c9-131">Ayrıca, AKS altyapısını kullanarak Azure Stack hub 'a bir Kubernetes kümesi dağıtmak için mevcut bir Windows veya Linux sanal makinesini kullanabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="125c9-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="125c9-132">AKS altyapısının adım adım işlemi ve gereksinimleri burada belgelenmiştir:</span><span class="sxs-lookup"><span data-stu-id="125c9-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="125c9-133">[Linux üzerinde AKS altyapısını Azure Stack hub 'a](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (veya [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows)kullanarak) yükler</span><span class="sxs-lookup"><span data-stu-id="125c9-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="125c9-134">AKS motoru, (yönetilmeyen) Kubernetes kümelerini (Azure 'da ve Azure Stack hub 'da) dağıtmak ve çalıştırmak için yardımcı bir araçtır.</span><span class="sxs-lookup"><span data-stu-id="125c9-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="125c9-135">Azure Stack hub 'ındaki AKS altyapısının ayrıntıları ve farklılıkları aşağıda açıklanmıştır:</span><span class="sxs-lookup"><span data-stu-id="125c9-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="125c9-136">Azure Stack hub 'daki AKS Altyapısı nedir?</span><span class="sxs-lookup"><span data-stu-id="125c9-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="125c9-137">[Azure Stack hub 'Daki aks altyapısı](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (GitHub üzerinde)</span><span class="sxs-lookup"><span data-stu-id="125c9-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="125c9-138">Örnek ortam, AKS motoru VM 'sini otomatik hale getirmek için Terrayform kullanacaktır.</span><span class="sxs-lookup"><span data-stu-id="125c9-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="125c9-139">[Ayrıntıları ve kodu yardımcı GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md)deposunda bulabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="125c9-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="125c9-140">Bu adımın sonucu, AKS altyapısı Yardımcısı VM 'sini ve ilgili kaynakları içeren Azure Stack hub 'ında yeni bir kaynak grubudur:</span><span class="sxs-lookup"><span data-stu-id="125c9-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![Azure Stack hub 'da AKS motoru VM kaynakları](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="125c9-142">AKS altyapısını bağlantısız bir AIR-gapped ortamında dağıtmanız gerekiyorsa, daha fazla bilgi edinmek için [bağlantısı kesilen Azure Stack hub örneklerini](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) gözden geçirin.</span><span class="sxs-lookup"><span data-stu-id="125c9-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="125c9-143">Bir sonraki adımda, bir Kubernetes kümesi dağıtmak için yeni dağıtılan AKS altyapısı sanal makinesini kullanacağız.</span><span class="sxs-lookup"><span data-stu-id="125c9-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="125c9-144">AKS motoru Yardımcısı VM 'sine bağlanma</span><span class="sxs-lookup"><span data-stu-id="125c9-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="125c9-145">Önce, önceden oluşturulmuş AKS altyapısı Yardımcısı VM 'sine bağlanmanız gerekir.</span><span class="sxs-lookup"><span data-stu-id="125c9-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="125c9-146">VM 'nin genel bir IP adresi olmalıdır ve SSH (bağlantı noktası 22/TCP) üzerinden erişilebilir olması gerekir.</span><span class="sxs-lookup"><span data-stu-id="125c9-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![AKS motoru VM 'ye Genel Bakış sayfası](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="125c9-148">SSH kullanarak bir Linux VM 'sine bağlanmak için Windows 10 ' da MobaXterm, puTTY veya PowerShell gibi tercih ettiğiniz bir aracı kullanabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="125c9-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="125c9-149">Bağlandıktan sonra komutunu çalıştırın `aks-engine` .</span><span class="sxs-lookup"><span data-stu-id="125c9-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="125c9-150">AKS altyapısı ve Kubernetes sürümleri hakkında daha fazla bilgi edinmek için [desteklenen AKS altyapısı sürümlerine](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) gidin.</span><span class="sxs-lookup"><span data-stu-id="125c9-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![aks-Engine komut satırı örneği](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="125c9-152">Kubernetes kümesi dağıtma</span><span class="sxs-lookup"><span data-stu-id="125c9-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="125c9-153">AKS motor Yardımcısı VM 'sinin kendisi Azure Stack hub 'imizde bir Kubernetes kümesi oluşturmadı.</span><span class="sxs-lookup"><span data-stu-id="125c9-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="125c9-154">Kümeyi oluşturmak, AKS motoru Yardımcısı VM 'sinde gerçekleştirilecek ilk eylemdir.</span><span class="sxs-lookup"><span data-stu-id="125c9-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="125c9-155">Adım adım işlem burada belgelenmiştir:</span><span class="sxs-lookup"><span data-stu-id="125c9-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="125c9-156">Bir Kubernetes kümesini Azure Stack hub 'ında AKS altyapısı ile dağıtma</span><span class="sxs-lookup"><span data-stu-id="125c9-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="125c9-157">Komutun nihai sonucu `aks-engine deploy` ve önceki adımlarda bulunan hazırlıklar, ilk Azure Stack hub örneğinin kiracı alanına dağıtılan tam özellikli bir Kubernetes kümesidir.</span><span class="sxs-lookup"><span data-stu-id="125c9-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="125c9-158">Küme, VM 'Ler, yük dengeleyiciler, sanal ağlar, diskler vb. gibi Azure IaaS bileşenlerinden oluşur.</span><span class="sxs-lookup"><span data-stu-id="125c9-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![Hub portalı Azure Stack küme IaaS bileşenleri](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="125c9-160">Azure yük dengeleyici (K8s API uç noktası)</span><span class="sxs-lookup"><span data-stu-id="125c9-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="125c9-161">Çalışan düğümleri (aracı havuzu)</span><span class="sxs-lookup"><span data-stu-id="125c9-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="125c9-162">Ana düğümler</span><span class="sxs-lookup"><span data-stu-id="125c9-162">Master Nodes</span></span>

<span data-ttu-id="125c9-163">Küme artık çalışır durumdadır ve bir sonraki adımda bu sunucuya bağlanacağız.</span><span class="sxs-lookup"><span data-stu-id="125c9-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="125c9-164">Kubernetes kümesine bağlanma</span><span class="sxs-lookup"><span data-stu-id="125c9-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="125c9-165">Artık daha önce oluşturulmuş Kubernetes kümesine SSH aracılığıyla (dağıtımın parçası olarak belirtilen SSH anahtarını kullanarak) veya aracılığıyla `kubectl` (önerilir) bağlanabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="125c9-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="125c9-166">Kubernetes komut satırı aracı `kubectl` [burada](https://kubernetes.io/docs/tasks/tools/install-kubectl/)Windows, Linux ve MacOS 'ta kullanılabilir.</span><span class="sxs-lookup"><span data-stu-id="125c9-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="125c9-167">Zaten önceden yüklenmiş ve kümeimizin ana düğümlerinde yapılandırılmış.</span><span class="sxs-lookup"><span data-stu-id="125c9-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Ana düğümde kubectl yürütme](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="125c9-169">Yönetim görevleri için ana düğümü bir sıçrama kutusu olarak kullanmanız önerilmez.</span><span class="sxs-lookup"><span data-stu-id="125c9-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="125c9-170">`kubectl`Yapılandırma, `.kube/config` ana düğümlerde ve aks motoru VM 'sinde depolanır.</span><span class="sxs-lookup"><span data-stu-id="125c9-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="125c9-171">Yapılandırmayı Kubernetes kümesine bağlantısı olan bir yönetim makinesine kopyalayabilir ve `kubectl` komutunu burada kullanabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="125c9-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="125c9-172">Bu `.kube/config` dosya daha sonra Azure pipelines ' de bir hizmet bağlantısı yapılandırmak için de kullanılır.</span><span class="sxs-lookup"><span data-stu-id="125c9-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="125c9-173">Kubernetes kümeniz için kimlik bilgilerini içerdikleri için bu dosyaları güvende tutun.</span><span class="sxs-lookup"><span data-stu-id="125c9-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="125c9-174">Dosyaya erişimi olan bir saldırganın, yönetici erişimi kazanmak için yeterli bilgi vardır.</span><span class="sxs-lookup"><span data-stu-id="125c9-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="125c9-175">İlk dosya kullanılarak gerçekleştirilen tüm eylemler, `.kube/config` Küme Yöneticisi hesabı kullanılarak yapılır.</span><span class="sxs-lookup"><span data-stu-id="125c9-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="125c9-176">Şimdi, `kubectl` kümenizin durumunu denetlemek için kullanarak çeşitli komutları deneyebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="125c9-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> <span data-ttu-id="125c9-177">Kubernetes, ayrıntılı rol tanımları ve rol bağlamaları oluşturmanıza olanak sağlayan kendi _ *rol tabanlı Access Control (RBAC)*\* modeline sahiptir.</span><span class="sxs-lookup"><span data-stu-id="125c9-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="125c9-178">Bu, Küme Yöneticisi izinleri teslim etmek yerine kümeye erişimi denetlemek için tercih edilen yoldur.</span><span class="sxs-lookup"><span data-stu-id="125c9-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="125c9-179">Kubernetes kümelerine Azure Pipelines bağlama</span><span class="sxs-lookup"><span data-stu-id="125c9-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="125c9-180">Yeni dağıtılan Kubernetes kümesine Azure Pipelines bağlamak için, `.kube/config` önceki adımda açıklandığı gibi Kume config () dosyasına ihtiyacımız var.</span><span class="sxs-lookup"><span data-stu-id="125c9-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="125c9-181">Kubernetes kümenizin ana düğümlerinden birine bağlanın.</span><span class="sxs-lookup"><span data-stu-id="125c9-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="125c9-182">Dosyanın içeriğini kopyalayın `.kube/config` .</span><span class="sxs-lookup"><span data-stu-id="125c9-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="125c9-183">Yeni bir "Kubernetes" hizmet bağlantısı oluşturmak için Azure DevOps > proje ayarları > hizmet bağlantıları 'na gidin (bir kimlik doğrulama yöntemi olarak KubeConfig kullanın)</span><span class="sxs-lookup"><span data-stu-id="125c9-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="125c9-184">Azure Pipelines (veya yapı aracıları) Kubernetes API 'sine erişebilmelidir.</span><span class="sxs-lookup"><span data-stu-id="125c9-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="125c9-185">Azure Pipelines Azure Stack Merkez Kubernetes clusetr 'e Internet bağlantısı varsa, şirket içinde barındırılan bir Azure Pipelines derleme Aracısı dağıtmanız gerekir.</span><span class="sxs-lookup"><span data-stu-id="125c9-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="125c9-186">Azure Pipelines için şirket içinde barındırılan aracılar dağıtırken, Azure Stack hub 'ında veya tüm gerekli yönetim uç noktalarına ağ bağlantısı olan bir makinede dağıtabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="125c9-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="125c9-187">Ayrıntıları burada görebilirsiniz:</span><span class="sxs-lookup"><span data-stu-id="125c9-187">See the details here:</span></span>

* <span data-ttu-id="125c9-188">[Windows](/azure/devops/pipelines/agents/v2-windows) veya [Linux](/azure/devops/pipelines/agents/v2-linux) üzerinde [Azure Pipelines aracıları](/azure/devops/pipelines/agents/agents)</span><span class="sxs-lookup"><span data-stu-id="125c9-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="125c9-189">Model [dağıtımı (CI/CD) konuları](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) bölümü, Microsoft tarafından barındırılan aracıların mi yoksa şirket içinde barındırılan aracıların mi kullanılacağını anlamanıza yardımcı olan bir karar akışı içerir:</span><span class="sxs-lookup"><span data-stu-id="125c9-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="125c9-190">[![karar akışı kendiliğinden barındırılan aracılar](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="125c9-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="125c9-191">Bu örnek çözümde topoloji, her bir Azure Stack hub örneğinde şirket içinde barındırılan bir yapı Aracısı içerir.</span><span class="sxs-lookup"><span data-stu-id="125c9-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="125c9-192">Aracı Azure Stack hub yönetim uç noktalarına ve Kubernetes kümesi API uç noktalarına erişebilir.</span><span class="sxs-lookup"><span data-stu-id="125c9-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="125c9-193">[![yalnızca giden trafik](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="125c9-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="125c9-194">Bu tasarım, uygulama çözümünden yalnızca giden bağlantılara sahip olmak için ortak bir düzenleme gereksinimini karşılar.</span><span class="sxs-lookup"><span data-stu-id="125c9-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="125c9-195">İzlemeyi yapılandırma</span><span class="sxs-lookup"><span data-stu-id="125c9-195">Configure monitoring</span></span>

<span data-ttu-id="125c9-196">Çözümdeki kapsayıcıları izlemek için kapsayıcılar için [Azure İzleyicisi](/azure/azure-monitor/) 'ni kullanabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="125c9-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="125c9-197">Bu, Azure Izleyicisini Azure Stack hub 'daki AKS altyapısına dağıtılan Kubernetes kümesine yönlendirir.</span><span class="sxs-lookup"><span data-stu-id="125c9-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="125c9-198">Kümenizde Azure Izleyicisini etkinleştirmenin iki yolu vardır.</span><span class="sxs-lookup"><span data-stu-id="125c9-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="125c9-199">Her iki şekilde de Azure 'da bir Log Analytics çalışma alanı ayarlamanız gerekir.</span><span class="sxs-lookup"><span data-stu-id="125c9-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="125c9-200">[Yöntem tek](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) bir Held grafiği kullanır</span><span class="sxs-lookup"><span data-stu-id="125c9-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="125c9-201">AKS motor kümesi belirtiminin parçası olarak [Iki yöntemi](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two)</span><span class="sxs-lookup"><span data-stu-id="125c9-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="125c9-202">Örnek topolojide "yöntem bir" kullanılır ve bu işlem, işlemin otomatikleştirilmesine ve güncelleştirmelerin daha kolay yüklenebilmesini sağlar.</span><span class="sxs-lookup"><span data-stu-id="125c9-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="125c9-203">Sonraki adımda, bir Azure LogAnalytics çalışma alanına (KIMLIK ve anahtar) ( `Helm` sürüm 3) ve makinenizde ihtiyacınız vardır `kubectl` .</span><span class="sxs-lookup"><span data-stu-id="125c9-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="125c9-204">Helk, macOS, Windows ve Linux üzerinde çalışan bir ikili olarak kullanılabilen bir Kubernetes paket yöneticisidir.</span><span class="sxs-lookup"><span data-stu-id="125c9-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="125c9-205">Buradan indirilebilir: [Helm.sh](https://helm.sh/docs/intro/quickstart/) Held, komut Için kullanılan Kubernetes yapılandırma dosyasını kullanır `kubectl` .</span><span class="sxs-lookup"><span data-stu-id="125c9-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="125c9-206">Bu komut, Kubernetes kümenize Azure Izleyici aracısını yükler:</span><span class="sxs-lookup"><span data-stu-id="125c9-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="125c9-207">Kubernetes kümenizdeki Operations Management Suite (OMS) Aracısı, izleme verilerini Azure Log Analytics çalışma alanınıza gönderir (giden HTTPS kullanarak).</span><span class="sxs-lookup"><span data-stu-id="125c9-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="125c9-208">Artık Azure Stack hub 'ındaki Kubernetes kümeleriniz hakkında daha ayrıntılı Öngörüler almak için Azure Izleyici 'yi kullanabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="125c9-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="125c9-209">Bu tasarım, uygulamanızın kümeleriyle otomatik olarak dağıtılabilecek analiz gücünü göstermek için güçlü bir yoldur.</span><span class="sxs-lookup"><span data-stu-id="125c9-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="125c9-210">[![Azure izleyici 'de Azure Stack hub kümeleri](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="125c9-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="125c9-211">[![Azure Izleyici kümesi ayrıntıları](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="125c9-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="125c9-212">Azure Izleyici herhangi bir Azure Stack hub verisi göstermezse, lütfen [bir Azure Loganalytics çalışma alanına nasıl AzureMonitor-Containers çözüm ekleneceğini](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) açıklayan yönergeleri izlediğinizden emin olun.</span><span class="sxs-lookup"><span data-stu-id="125c9-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="125c9-213">Uygulamayı dağıtma</span><span class="sxs-lookup"><span data-stu-id="125c9-213">Deploy the application</span></span>

<span data-ttu-id="125c9-214">Örnek uygulamamızı yüklemeden önce, Kubernetes kümenizdeki NGINX tabanlı giriş denetleyicisini yapılandırmak için başka bir adım vardır.</span><span class="sxs-lookup"><span data-stu-id="125c9-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="125c9-215">Giriş denetleyicisi, ana bilgisayar, yol veya protokole göre kümizdeki trafiği yönlendirmek için katman 7 yük dengeleyici olarak kullanılır.</span><span class="sxs-lookup"><span data-stu-id="125c9-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="125c9-216">NGINX-giriş, Held grafik olarak kullanılabilir.</span><span class="sxs-lookup"><span data-stu-id="125c9-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="125c9-217">Ayrıntılı yönergeler için, [Helu grafik GitHub deposuna](https://github.com/helm/charts/tree/master/stable/nginx-ingress)bakın.</span><span class="sxs-lookup"><span data-stu-id="125c9-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="125c9-218">Örnek uygulamamız, önceki adımda yer aldığı [Azure Izleme Aracısı](#configure-monitoring) gibi bir Helm grafiği olarak da paketlenmiştir.</span><span class="sxs-lookup"><span data-stu-id="125c9-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="125c9-219">Bu nedenle, uygulamayı Kubernetes kümemize dağıtmak basittir.</span><span class="sxs-lookup"><span data-stu-id="125c9-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="125c9-220">[Held grafik dosyalarını yardımcı GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm) deposunda bulabilirsiniz</span><span class="sxs-lookup"><span data-stu-id="125c9-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="125c9-221">Örnek uygulama, iki Azure Stack hub örneğinin her birinde bir Kubernetes kümesine dağıtılan üç katmanlı bir uygulamadır.</span><span class="sxs-lookup"><span data-stu-id="125c9-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="125c9-222">Uygulama bir MongoDB veritabanı kullanır.</span><span class="sxs-lookup"><span data-stu-id="125c9-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="125c9-223">Verilerin, model verilerinde birden çok örneğe nasıl çoğaltılacağı hakkında daha fazla bilgi edinebilirsiniz [ve depolama konuları](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span><span class="sxs-lookup"><span data-stu-id="125c9-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="125c9-224">Uygulama için HELI grafiğini dağıttıktan sonra, uygulamanızın dağıtım ve durum bilgisi olan (veritabanı için) tek bir pod ile temsil edilen üç katmanını görürsünüz:</span><span class="sxs-lookup"><span data-stu-id="125c9-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

<span data-ttu-id="125c9-225">Hizmetler 'de, tarafında NGINX tabanlı giriş denetleyicisini ve genel IP adresini bulacaksınız:</span><span class="sxs-lookup"><span data-stu-id="125c9-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

<span data-ttu-id="125c9-226">"Dış IP" adresi "uygulama uç noktasıdır".</span><span class="sxs-lookup"><span data-stu-id="125c9-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="125c9-227">Kullanıcılar uygulamayı açmak için bağlanacaktır ve sonraki adım [yapılandırma Traffic Manager](#configure-traffic-manager)uç noktası olarak da kullanılır.</span><span class="sxs-lookup"><span data-stu-id="125c9-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="125c9-228">Uygulamayı otomatik ölçeklendirme</span><span class="sxs-lookup"><span data-stu-id="125c9-228">Autoscale the application</span></span>
<span data-ttu-id="125c9-229">İsteğe bağlı olarak, CPU kullanımı gibi belirli ölçümlere göre ölçeği genişletmek veya düşürmek için [yatay Pod otomatik Scaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) 'ı yapılandırabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="125c9-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="125c9-230">Aşağıdaki komut, derecelendirme Web dağıtımı tarafından denetlenen pods 'lerin 1 ila 10 çoğaltmasını tutan bir yatay Pod otomatik Scaler oluşturacaktır.</span><span class="sxs-lookup"><span data-stu-id="125c9-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="125c9-231">HPA, %80 tüm yığınlarındaki ortalama CPU kullanımını sürdürmek için çoğaltma sayısını (dağıtım aracılığıyla) artırıp azaltacaktır.</span><span class="sxs-lookup"><span data-stu-id="125c9-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="125c9-232">Şu komutu çalıştırarak otomatik Scaler 'ın geçerli durumunu denetleyebilirsiniz:</span><span class="sxs-lookup"><span data-stu-id="125c9-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="125c9-233">Traffic Manager'ı Yapılandırma</span><span class="sxs-lookup"><span data-stu-id="125c9-233">Configure Traffic Manager</span></span>

<span data-ttu-id="125c9-234">Uygulamanın iki (veya daha fazla) dağıtımı arasında trafik dağıtmak için [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview)kullanacağız.</span><span class="sxs-lookup"><span data-stu-id="125c9-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="125c9-235">Azure Traffic Manager, Azure 'da DNS tabanlı bir trafik yük dengeleyicidir.</span><span class="sxs-lookup"><span data-stu-id="125c9-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="125c9-236">Traffic Manager, istemci isteklerini bir trafik yönlendirme yöntemine ve uç noktaların durumuna göre en uygun hizmet uç noktasına yönlendirmek için DNS kullanır.</span><span class="sxs-lookup"><span data-stu-id="125c9-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="125c9-237">Azure Traffic Manager kullanmak yerine, şirket içinde barındırılan diğer genel yük dengeleme çözümlerini de kullanabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="125c9-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="125c9-238">Örnek senaryoda, uygulamanızın iki örneği arasında trafik dağıtmak için Azure Traffic Manager kullanacağız.</span><span class="sxs-lookup"><span data-stu-id="125c9-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="125c9-239">Azure Stack hub örnekleri üzerinde aynı veya farklı konumlarda çalıştırılabilir:</span><span class="sxs-lookup"><span data-stu-id="125c9-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![Şirket içi Traffic Manager](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="125c9-241">Azure 'da, Traffic Manager uygulamamız iki farklı örneğine işaret etmek üzere yapılandırdık:</span><span class="sxs-lookup"><span data-stu-id="125c9-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="125c9-242">[![TM uç noktası profili](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="125c9-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="125c9-243">Gördüğünüz gibi, iki uç nokta, [önceki bölümden](#deploy-the-application)dağıtılan uygulamanın iki örneğine işaret edebilir.</span><span class="sxs-lookup"><span data-stu-id="125c9-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="125c9-244">Şu noktada:</span><span class="sxs-lookup"><span data-stu-id="125c9-244">At this point:</span></span>
- <span data-ttu-id="125c9-245">Kubernetes altyapısı, giriş denetleyicisi de dahil oluşturulmuştur.</span><span class="sxs-lookup"><span data-stu-id="125c9-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="125c9-246">Kümeler iki Azure Stack hub örneğine dağıtıldı.</span><span class="sxs-lookup"><span data-stu-id="125c9-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="125c9-247">İzleme yapılandırıldı.</span><span class="sxs-lookup"><span data-stu-id="125c9-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="125c9-248">Azure Traffic Manager, trafiği iki Azure Stack hub örneği üzerinden dengeleyebilir.</span><span class="sxs-lookup"><span data-stu-id="125c9-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="125c9-249">Bu altyapının en üstünde örnek üç katmanlı uygulama, HELI grafikleri kullanılarak otomatik olarak dağıtılır.</span><span class="sxs-lookup"><span data-stu-id="125c9-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="125c9-250">Çözüm artık kullanıcılara açık ve erişilebilir olmalıdır!</span><span class="sxs-lookup"><span data-stu-id="125c9-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="125c9-251">Ayrıca, sonraki iki bölümde ele alınan bazı dağıtım sonrası işletimsel hususlar de vardır.</span><span class="sxs-lookup"><span data-stu-id="125c9-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="125c9-252">Kubernetes’i yükseltme</span><span class="sxs-lookup"><span data-stu-id="125c9-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="125c9-253">Kubernetes kümesini yükseltirken aşağıdaki konuları göz önünde bulundurun:</span><span class="sxs-lookup"><span data-stu-id="125c9-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="125c9-254">Kubernetes kümesini yükseltmek, AKS altyapısı kullanılarak yapılabilecek karmaşık gün 2 işlemidir.</span><span class="sxs-lookup"><span data-stu-id="125c9-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="125c9-255">Daha fazla bilgi için bkz. [Azure Stack hub 'da bir Kubernetes kümesini yükseltme](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span><span class="sxs-lookup"><span data-stu-id="125c9-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="125c9-256">AKS motoru, kümeleri daha yeni Kubernetes ve temel işletim sistemi görüntü sürümlerine yükseltmenize olanak tanır.</span><span class="sxs-lookup"><span data-stu-id="125c9-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="125c9-257">Daha fazla bilgi için bkz. [Yeni bir Kubernetes sürümüne yükseltme adımları](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span><span class="sxs-lookup"><span data-stu-id="125c9-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="125c9-258">Yalnızca daha yeni temel işletim sistemi görüntü sürümlerine de yalnızca düşük olan düğümleri yükseltebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="125c9-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="125c9-259">Daha fazla bilgi için bkz. [işletim sistemi görüntüsünü yükseltmek Için adımlar](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span><span class="sxs-lookup"><span data-stu-id="125c9-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="125c9-260">Daha yeni temel işletim sistemi görüntüleri güvenlik ve çekirdek güncelleştirmelerini içerir.</span><span class="sxs-lookup"><span data-stu-id="125c9-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="125c9-261">Daha yeni Kubernetes sürümlerinin ve işletim sistemi görüntülerinin kullanılabilirliğini izlemek, küme işlecinin sorumluluğundadır.</span><span class="sxs-lookup"><span data-stu-id="125c9-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="125c9-262">Operatör, AKS altyapısını kullanarak bu yükseltmeleri planlayıp yürütmelidir.</span><span class="sxs-lookup"><span data-stu-id="125c9-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="125c9-263">Temel işletim sistemi görüntüleri, Azure Stack hub Işleci tarafından Azure Stack hub marketi 'nden indirilmelidir.</span><span class="sxs-lookup"><span data-stu-id="125c9-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="125c9-264">Kubernetes 'i ölçeklendirme</span><span class="sxs-lookup"><span data-stu-id="125c9-264">Scale Kubernetes</span></span>

<span data-ttu-id="125c9-265">Ölçek, AKS altyapısı kullanılarak kullanılabilecek başka bir gün 2 işlemidir.</span><span class="sxs-lookup"><span data-stu-id="125c9-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="125c9-266">Ölçek komutu, yeni bir Azure Resource Manager dağıtımına giriş olarak, çıkış dizinindeki küme yapılandırma dosyanızı (apimodel.js) yeniden kullanır.</span><span class="sxs-lookup"><span data-stu-id="125c9-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="125c9-267">AKS altyapısı, belirli bir aracı havuzunda ölçek işlemini yürütür.</span><span class="sxs-lookup"><span data-stu-id="125c9-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="125c9-268">Ölçeklendirme işlemi tamamlandığında, AKS motoru dosyadaki aynı apimodel.jsküme tanımını güncelleştirir.</span><span class="sxs-lookup"><span data-stu-id="125c9-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="125c9-269">Küme tanımı, güncelleştirilmiş, geçerli küme yapılandırmasını yansıtmak için yeni düğüm sayısını yansıtır.</span><span class="sxs-lookup"><span data-stu-id="125c9-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="125c9-270">Azure Stack hub 'da bir Kubernetes kümesini ölçeklendirme</span><span class="sxs-lookup"><span data-stu-id="125c9-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="125c9-271">Sonraki adımlar</span><span class="sxs-lookup"><span data-stu-id="125c9-271">Next steps</span></span>

- <span data-ttu-id="125c9-272">[Karma uygulama tasarımı konuları](overview-app-design-considerations.md) hakkında daha fazla bilgi edinin</span><span class="sxs-lookup"><span data-stu-id="125c9-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="125c9-273">[GitHub 'da Bu örnek için koda](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)yönelik geliştirmeleri gözden geçirin ve önerin.</span><span class="sxs-lookup"><span data-stu-id="125c9-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>