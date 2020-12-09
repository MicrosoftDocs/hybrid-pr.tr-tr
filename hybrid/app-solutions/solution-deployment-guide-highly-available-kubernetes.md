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
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>Azure Stack hub 'da yüksek kullanılabilirliğe sahip bir Kubernetes kümesi dağıtma

Bu makalede, farklı fiziksel konumlarda birden çok Azure Stack hub örneğine dağıtılan yüksek oranda kullanılabilir bir Kubernetes küme ortamı oluşturma işleminin nasıl yapılacağı gösterilir.

Bu çözüm dağıtımı kılavuzunda, şunları nasıl yapacağınızı öğreneceksiniz:

> [!div class="checklist"]
> - AKS altyapısını indirme ve hazırlama
> - AKS motoru Yardımcısı VM 'sine bağlanma
> - Kubernetes kümesi dağıtma
> - Kubernetes kümesine bağlanma
> - Kubernetes kümesine Azure Pipelines bağlama
> - İzlemeyi yapılandırma
> - Uygulamayı dağıtma
> - Otomatik ölçeklendirme uygulaması
> - Traffic Manager'ı Yapılandırma
> - Kubernetes’i yükseltme
> - Kubernetes 'i ölçeklendirme

> [!Tip]  
> ![Karma paragraf](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub, Azure uzantısıdır. Azure Stack hub, bulut bilgi işlemin çevikliğini ve yeniliklerini şirket içi ortamınıza sunarak, karma uygulamaları her yerde derleyip dağıtmanıza imkan tanıyan tek karma bulutu etkinleştirir.  
> 
> [Karma uygulama tasarımı ile ilgili önemli noktalar](overview-app-design-considerations.md) , karma uygulamalar tasarlamak, dağıtmak ve çalıştırmak için yazılım kalitesinin (yerleştirme, ölçeklenebilirlik, kullanılabilirlik, dayanıklılık, yönetilebilirlik ve güvenlik) aynı şekilde gözden geçirmeleri inceler. Tasarım konuları karma uygulama tasarımını iyileştirirken, üretim ortamlarındaki zorlukları en aza indirmeyle ilgili olarak size yardımcı olur.

## <a name="prerequisites"></a>Önkoşullar

Bu dağıtım kılavuzunu kullanmaya başlamadan önce şunları yaptığınızdan emin olun:

- [Yüksek kullanılabilirlik Kubernetes kümesi örüntüsünün](pattern-highly-available-kubernetes.md) makalesini gözden geçirin.
- Bu makalede başvurulan ek varlıkları içeren [yardımcı GitHub deposunun](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)içeriğini gözden geçirin.
- En az ["katkıda bulunan" izinlerle](/azure-stack/user/azure-stack-manage-permissions) [Azure Stack hub Kullanıcı portalına](/azure-stack/user/azure-stack-use-portal)erişebilen bir hesabınız olmalıdır.

## <a name="download-and-prepare-aks-engine"></a>AKS altyapısını indirme ve hazırlama

AKS altyapısı, Azure Stack hub Azure Resource Manager uç noktalarına ulaşan herhangi bir Windows veya Linux ana bilgisayardan kullanılabilen bir ikili dosyaysa. Bu kılavuzda Azure Stack hub 'da yeni bir Linux (veya Windows) VM dağıtımı açıklanmaktadır. AKS altyapısı Kubernetes kümelerini dağıttığında daha sonra kullanılacaktır.

> [!NOTE]
> Ayrıca, AKS altyapısını kullanarak Azure Stack hub 'a bir Kubernetes kümesi dağıtmak için mevcut bir Windows veya Linux sanal makinesini kullanabilirsiniz.

AKS altyapısının adım adım işlemi ve gereksinimleri burada belgelenmiştir:

* [Linux üzerinde AKS altyapısını Azure Stack hub 'a](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (veya [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows)kullanarak) yükler

AKS motoru, (yönetilmeyen) Kubernetes kümelerini (Azure 'da ve Azure Stack hub 'da) dağıtmak ve çalıştırmak için yardımcı bir araçtır.

Azure Stack hub 'ındaki AKS altyapısının ayrıntıları ve farklılıkları aşağıda açıklanmıştır:

* [Azure Stack hub 'daki AKS Altyapısı nedir?](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [Azure Stack hub 'Daki aks altyapısı](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (GitHub üzerinde)

Örnek ortam, AKS motoru VM 'sini otomatik hale getirmek için Terrayform kullanacaktır. [Ayrıntıları ve kodu yardımcı GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md)deposunda bulabilirsiniz.

Bu adımın sonucu, AKS altyapısı Yardımcısı VM 'sini ve ilgili kaynakları içeren Azure Stack hub 'ında yeni bir kaynak grubudur:

![Azure Stack hub 'da AKS motoru VM kaynakları](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> AKS altyapısını bağlantısız bir AIR-gapped ortamında dağıtmanız gerekiyorsa, daha fazla bilgi edinmek için [bağlantısı kesilen Azure Stack hub örneklerini](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) gözden geçirin.

Bir sonraki adımda, bir Kubernetes kümesi dağıtmak için yeni dağıtılan AKS altyapısı sanal makinesini kullanacağız.

## <a name="connect-to-the-aks-engine-helper-vm"></a>AKS motoru Yardımcısı VM 'sine bağlanma

Önce, önceden oluşturulmuş AKS altyapısı Yardımcısı VM 'sine bağlanmanız gerekir.

VM 'nin genel bir IP adresi olmalıdır ve SSH (bağlantı noktası 22/TCP) üzerinden erişilebilir olması gerekir.

![AKS motoru VM 'ye Genel Bakış sayfası](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> SSH kullanarak bir Linux VM 'sine bağlanmak için Windows 10 ' da MobaXterm, puTTY veya PowerShell gibi tercih ettiğiniz bir aracı kullanabilirsiniz.

```console
ssh <username>@<ipaddress>
```

Bağlandıktan sonra komutunu çalıştırın `aks-engine` . AKS altyapısı ve Kubernetes sürümleri hakkında daha fazla bilgi edinmek için [desteklenen AKS altyapısı sürümlerine](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) gidin.

![aks-Engine komut satırı örneği](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>Kubernetes kümesi dağıtma

AKS motor Yardımcısı VM 'sinin kendisi Azure Stack hub 'imizde bir Kubernetes kümesi oluşturmadı. Kümeyi oluşturmak, AKS motoru Yardımcısı VM 'sinde gerçekleştirilecek ilk eylemdir.

Adım adım işlem burada belgelenmiştir:

* [Bir Kubernetes kümesini Azure Stack hub 'ında AKS altyapısı ile dağıtma](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

Komutun nihai sonucu `aks-engine deploy` ve önceki adımlarda bulunan hazırlıklar, ilk Azure Stack hub örneğinin kiracı alanına dağıtılan tam özellikli bir Kubernetes kümesidir. Küme, VM 'Ler, yük dengeleyiciler, sanal ağlar, diskler vb. gibi Azure IaaS bileşenlerinden oluşur.

![Hub portalı Azure Stack küme IaaS bileşenleri](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Azure yük dengeleyici (K8s API uç noktası)
2) Çalışan düğümleri (aracı havuzu)
3) Ana düğümler

Küme artık çalışır durumdadır ve bir sonraki adımda bu sunucuya bağlanacağız.

## <a name="connect-to-the-kubernetes-cluster"></a>Kubernetes kümesine bağlanma

Artık daha önce oluşturulmuş Kubernetes kümesine SSH aracılığıyla (dağıtımın parçası olarak belirtilen SSH anahtarını kullanarak) veya aracılığıyla `kubectl` (önerilir) bağlanabilirsiniz. Kubernetes komut satırı aracı `kubectl` [burada](https://kubernetes.io/docs/tasks/tools/install-kubectl/)Windows, Linux ve MacOS 'ta kullanılabilir. Zaten önceden yüklenmiş ve kümeimizin ana düğümlerinde yapılandırılmış.

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Ana düğümde kubectl yürütme](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

Yönetim görevleri için ana düğümü bir sıçrama kutusu olarak kullanmanız önerilmez. `kubectl`Yapılandırma, `.kube/config` ana düğümlerde ve aks motoru VM 'sinde depolanır. Yapılandırmayı Kubernetes kümesine bağlantısı olan bir yönetim makinesine kopyalayabilir ve `kubectl` komutunu burada kullanabilirsiniz. Bu `.kube/config` dosya daha sonra Azure pipelines ' de bir hizmet bağlantısı yapılandırmak için de kullanılır.

> [!IMPORTANT]
> Kubernetes kümeniz için kimlik bilgilerini içerdikleri için bu dosyaları güvende tutun. Dosyaya erişimi olan bir saldırganın, yönetici erişimi kazanmak için yeterli bilgi vardır. İlk dosya kullanılarak gerçekleştirilen tüm eylemler, `.kube/config` Küme Yöneticisi hesabı kullanılarak yapılır.

Şimdi, `kubectl` kümenizin durumunu denetlemek için kullanarak çeşitli komutları deneyebilirsiniz.

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
> Kubernetes, ayrıntılı rol tanımları ve rol bağlamaları oluşturmanıza olanak sağlayan kendi _ *rol tabanlı Access Control (RBAC)** modeline sahiptir. Bu, Küme Yöneticisi izinleri teslim etmek yerine kümeye erişimi denetlemek için tercih edilen yoldur.

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>Kubernetes kümelerine Azure Pipelines bağlama

Yeni dağıtılan Kubernetes kümesine Azure Pipelines bağlamak için, `.kube/config` önceki adımda açıklandığı gibi Kume config () dosyasına ihtiyacımız var.

* Kubernetes kümenizin ana düğümlerinden birine bağlanın.
* Dosyanın içeriğini kopyalayın `.kube/config` .
* Yeni bir "Kubernetes" hizmet bağlantısı oluşturmak için Azure DevOps > proje ayarları > hizmet bağlantıları 'na gidin (bir kimlik doğrulama yöntemi olarak KubeConfig kullanın)

> [!IMPORTANT]
> Azure Pipelines (veya yapı aracıları) Kubernetes API 'sine erişebilmelidir. Azure Pipelines Azure Stack Merkez Kubernetes clusetr 'e Internet bağlantısı varsa, şirket içinde barındırılan bir Azure Pipelines derleme Aracısı dağıtmanız gerekir.

Azure Pipelines için şirket içinde barındırılan aracılar dağıtırken, Azure Stack hub 'ında veya tüm gerekli yönetim uç noktalarına ağ bağlantısı olan bir makinede dağıtabilirsiniz. Ayrıntıları burada görebilirsiniz:

* [Windows](/azure/devops/pipelines/agents/v2-windows) veya [Linux](/azure/devops/pipelines/agents/v2-linux) üzerinde [Azure Pipelines aracıları](/azure/devops/pipelines/agents/agents)

Model [dağıtımı (CI/CD) konuları](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) bölümü, Microsoft tarafından barındırılan aracıların mi yoksa şirket içinde barındırılan aracıların mi kullanılacağını anlamanıza yardımcı olan bir karar akışı içerir:

[![karar akışı kendiliğinden barındırılan aracılar](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

Bu örnek çözümde topoloji, her bir Azure Stack hub örneğinde şirket içinde barındırılan bir yapı Aracısı içerir. Aracı Azure Stack hub yönetim uç noktalarına ve Kubernetes kümesi API uç noktalarına erişebilir.

[![yalnızca giden trafik](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

Bu tasarım, uygulama çözümünden yalnızca giden bağlantılara sahip olmak için ortak bir düzenleme gereksinimini karşılar.

## <a name="configure-monitoring"></a>İzlemeyi yapılandırma

Çözümdeki kapsayıcıları izlemek için kapsayıcılar için [Azure İzleyicisi](/azure/azure-monitor/) 'ni kullanabilirsiniz. Bu, Azure Izleyicisini Azure Stack hub 'daki AKS altyapısına dağıtılan Kubernetes kümesine yönlendirir.

Kümenizde Azure Izleyicisini etkinleştirmenin iki yolu vardır. Her iki şekilde de Azure 'da bir Log Analytics çalışma alanı ayarlamanız gerekir.

* [Yöntem tek](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) bir Held grafiği kullanır
* AKS motor kümesi belirtiminin parçası olarak [Iki yöntemi](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two)

Örnek topolojide "yöntem bir" kullanılır ve bu işlem, işlemin otomatikleştirilmesine ve güncelleştirmelerin daha kolay yüklenebilmesini sağlar.

Sonraki adımda, bir Azure LogAnalytics çalışma alanına (KIMLIK ve anahtar) ( `Helm` sürüm 3) ve makinenizde ihtiyacınız vardır `kubectl` .

Helk, macOS, Windows ve Linux üzerinde çalışan bir ikili olarak kullanılabilen bir Kubernetes paket yöneticisidir. Buradan indirilebilir: [Helm.sh](https://helm.sh/docs/intro/quickstart/) Held, komut Için kullanılan Kubernetes yapılandırma dosyasını kullanır `kubectl` .

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

Bu komut, Kubernetes kümenize Azure Izleyici aracısını yükler:

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

Kubernetes kümenizdeki Operations Management Suite (OMS) Aracısı, izleme verilerini Azure Log Analytics çalışma alanınıza gönderir (giden HTTPS kullanarak). Artık Azure Stack hub 'ındaki Kubernetes kümeleriniz hakkında daha ayrıntılı Öngörüler almak için Azure Izleyici 'yi kullanabilirsiniz. Bu tasarım, uygulamanızın kümeleriyle otomatik olarak dağıtılabilecek analiz gücünü göstermek için güçlü bir yoldur.

[![Azure izleyici 'de Azure Stack hub kümeleri](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Azure Izleyici kümesi ayrıntıları](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> Azure Izleyici herhangi bir Azure Stack hub verisi göstermezse, lütfen [bir Azure Loganalytics çalışma alanına nasıl AzureMonitor-Containers çözüm ekleneceğini](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) açıklayan yönergeleri izlediğinizden emin olun.

## <a name="deploy-the-application"></a>Uygulamayı dağıtma

Örnek uygulamamızı yüklemeden önce, Kubernetes kümenizdeki NGINX tabanlı giriş denetleyicisini yapılandırmak için başka bir adım vardır. Giriş denetleyicisi, ana bilgisayar, yol veya protokole göre kümizdeki trafiği yönlendirmek için katman 7 yük dengeleyici olarak kullanılır. NGINX-giriş, Held grafik olarak kullanılabilir. Ayrıntılı yönergeler için, [Helu grafik GitHub deposuna](https://github.com/helm/charts/tree/master/stable/nginx-ingress)bakın.

Örnek uygulamamız, önceki adımda yer aldığı [Azure Izleme Aracısı](#configure-monitoring) gibi bir Helm grafiği olarak da paketlenmiştir. Bu nedenle, uygulamayı Kubernetes kümemize dağıtmak basittir. [Held grafik dosyalarını yardımcı GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm) deposunda bulabilirsiniz

Örnek uygulama, iki Azure Stack hub örneğinin her birinde bir Kubernetes kümesine dağıtılan üç katmanlı bir uygulamadır. Uygulama bir MongoDB veritabanı kullanır. Verilerin, model verilerinde birden çok örneğe nasıl çoğaltılacağı hakkında daha fazla bilgi edinebilirsiniz [ve depolama konuları](pattern-highly-available-kubernetes.md#data-and-storage-considerations).

Uygulama için HELI grafiğini dağıttıktan sonra, uygulamanızın dağıtım ve durum bilgisi olan (veritabanı için) tek bir pod ile temsil edilen üç katmanını görürsünüz:

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

Hizmetler 'de, tarafında NGINX tabanlı giriş denetleyicisini ve genel IP adresini bulacaksınız:

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

"Dış IP" adresi "uygulama uç noktasıdır". Kullanıcılar uygulamayı açmak için bağlanacaktır ve sonraki adım [yapılandırma Traffic Manager](#configure-traffic-manager)uç noktası olarak da kullanılır.

## <a name="autoscale-the-application"></a>Uygulamayı otomatik ölçeklendirme
İsteğe bağlı olarak, CPU kullanımı gibi belirli ölçümlere göre ölçeği genişletmek veya düşürmek için [yatay Pod otomatik Scaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) 'ı yapılandırabilirsiniz. Aşağıdaki komut, derecelendirme Web dağıtımı tarafından denetlenen pods 'lerin 1 ila 10 çoğaltmasını tutan bir yatay Pod otomatik Scaler oluşturacaktır. HPA, %80 tüm yığınlarındaki ortalama CPU kullanımını sürdürmek için çoğaltma sayısını (dağıtım aracılığıyla) artırıp azaltacaktır.

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
Şu komutu çalıştırarak otomatik Scaler 'ın geçerli durumunu denetleyebilirsiniz:

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>Traffic Manager'ı Yapılandırma

Uygulamanın iki (veya daha fazla) dağıtımı arasında trafik dağıtmak için [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview)kullanacağız. Azure Traffic Manager, Azure 'da DNS tabanlı bir trafik yük dengeleyicidir.

> [!NOTE]
> Traffic Manager, istemci isteklerini bir trafik yönlendirme yöntemine ve uç noktaların durumuna göre en uygun hizmet uç noktasına yönlendirmek için DNS kullanır.

Azure Traffic Manager kullanmak yerine, şirket içinde barındırılan diğer genel yük dengeleme çözümlerini de kullanabilirsiniz. Örnek senaryoda, uygulamanızın iki örneği arasında trafik dağıtmak için Azure Traffic Manager kullanacağız. Azure Stack hub örnekleri üzerinde aynı veya farklı konumlarda çalıştırılabilir:

![Şirket içi Traffic Manager](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

Azure 'da, Traffic Manager uygulamamız iki farklı örneğine işaret etmek üzere yapılandırdık:

[![TM uç noktası profili](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

Gördüğünüz gibi, iki uç nokta, [önceki bölümden](#deploy-the-application)dağıtılan uygulamanın iki örneğine işaret edebilir.

Şu noktada:
- Kubernetes altyapısı, giriş denetleyicisi de dahil oluşturulmuştur.
- Kümeler iki Azure Stack hub örneğine dağıtıldı.
- İzleme yapılandırıldı.
- Azure Traffic Manager, trafiği iki Azure Stack hub örneği üzerinden dengeleyebilir.
- Bu altyapının en üstünde örnek üç katmanlı uygulama, HELI grafikleri kullanılarak otomatik olarak dağıtılır. 

Çözüm artık kullanıcılara açık ve erişilebilir olmalıdır!

Ayrıca, sonraki iki bölümde ele alınan bazı dağıtım sonrası işletimsel hususlar de vardır.

## <a name="upgrade-kubernetes"></a>Kubernetes’i yükseltme

Kubernetes kümesini yükseltirken aşağıdaki konuları göz önünde bulundurun:

- Kubernetes kümesini yükseltmek, AKS altyapısı kullanılarak yapılabilecek karmaşık gün 2 işlemidir. Daha fazla bilgi için bkz. [Azure Stack hub 'da bir Kubernetes kümesini yükseltme](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).
- AKS motoru, kümeleri daha yeni Kubernetes ve temel işletim sistemi görüntü sürümlerine yükseltmenize olanak tanır. Daha fazla bilgi için bkz. [Yeni bir Kubernetes sürümüne yükseltme adımları](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version). 
- Yalnızca daha yeni temel işletim sistemi görüntü sürümlerine de yalnızca düşük olan düğümleri yükseltebilirsiniz. Daha fazla bilgi için bkz. [işletim sistemi görüntüsünü yükseltmek Için adımlar](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).

Daha yeni temel işletim sistemi görüntüleri güvenlik ve çekirdek güncelleştirmelerini içerir. Daha yeni Kubernetes sürümlerinin ve işletim sistemi görüntülerinin kullanılabilirliğini izlemek, küme işlecinin sorumluluğundadır. Operatör, AKS altyapısını kullanarak bu yükseltmeleri planlayıp yürütmelidir. Temel işletim sistemi görüntüleri, Azure Stack hub Işleci tarafından Azure Stack hub marketi 'nden indirilmelidir.

## <a name="scale-kubernetes"></a>Kubernetes 'i ölçeklendirme

Ölçek, AKS altyapısı kullanılarak kullanılabilecek başka bir gün 2 işlemidir.

Ölçek komutu, yeni bir Azure Resource Manager dağıtımına giriş olarak, çıkış dizinindeki küme yapılandırma dosyanızı (apimodel.js) yeniden kullanır. AKS altyapısı, belirli bir aracı havuzunda ölçek işlemini yürütür. Ölçeklendirme işlemi tamamlandığında, AKS motoru dosyadaki aynı apimodel.jsküme tanımını güncelleştirir. Küme tanımı, güncelleştirilmiş, geçerli küme yapılandırmasını yansıtmak için yeni düğüm sayısını yansıtır.

- [Azure Stack hub 'da bir Kubernetes kümesini ölçeklendirme](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>Sonraki adımlar

- [Karma uygulama tasarımı konuları](overview-app-design-considerations.md) hakkında daha fazla bilgi edinin
- [GitHub 'da Bu örnek için koda](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)yönelik geliştirmeleri gözden geçirin ve önerin.