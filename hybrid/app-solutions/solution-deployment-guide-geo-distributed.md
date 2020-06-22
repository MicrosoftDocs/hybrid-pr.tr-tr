---
title: Azure ve Azure Stack hub kullanarak coğrafi olarak dağıtılmış bir uygulamayla doğrudan trafik
description: Azure ve Azure Stack hub kullanarak coğrafi olarak dağıtılmış uygulama çözümü ile trafiği belirli uç noktalara nasıl yönlendireceğinizi öğrenin.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 8f2b7e48a62896acfce7293dcd4f18d5a43add01
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84912044"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a>Azure ve Azure Stack hub kullanarak coğrafi olarak dağıtılmış bir uygulamayla doğrudan trafik

Coğrafi olarak dağıtılmış uygulamalar modelini kullanarak çeşitli ölçümleri temel alarak trafiğin belirli uç noktalara nasıl yönlendirileceğini öğrenin. Coğrafi tabanlı Yönlendirme ve uç nokta yapılandırmasıyla Traffic Manager profili oluşturma, bilgilerin bölgesel gereksinimlere, şirkete ve uluslararası yönetmelere ve veri gereksinimlerinize göre uç noktalara yönlendirilmesini sağlar.

Bu çözümde, aşağıdakileri yapmak için bir örnek ortam oluşturacaksınız:

> [!div class="checklist"]
> - Coğrafi olarak dağıtılmış bir uygulama oluşturun.
> - Uygulamanızı hedeflemek için Traffic Manager kullanın.

## <a name="use-the-geo-distributed-apps-pattern"></a>Coğrafi olarak dağıtılmış uygulamalar modelini kullanma

Coğrafi olarak dağıtılmış düzende, uygulamanız bölgeleri kapsar. Genel bulutu varsayılan olarak kullanabilirsiniz, ancak bazı kullanıcılarınız, verilerinin bölgesinde kalmasını gerektirebilir. Kullanıcıları gereksinimlerine göre en uygun buluta yönlendirebilirsiniz.

### <a name="issues-and-considerations"></a>Sorunlar ve dikkat edilmesi gerekenler

#### <a name="scalability-considerations"></a>Ölçeklenebilirlik konusunda dikkat edilmesi gerekenler

Bu makalede oluşturacağınız çözüm, ölçeklenebilirlik sağlamak için değildir. Ancak, diğer Azure ve şirket içi çözümlerle birlikte kullanılırsa ölçeklenebilirlik gereksinimlerine uyum sağlayabilirsiniz. Traffic Manager aracılığıyla otomatik ölçeklendirmeyle karma çözüm oluşturma hakkında bilgi için bkz. [Azure ile platformlar arası ölçeklendirme çözümleri oluşturma](solution-deployment-guide-cross-cloud-scaling.md).

#### <a name="availability-considerations"></a>Kullanılabilirlik konusunda dikkat edilmesi gerekenler

Ölçeklenebilirlik konusunda olduğu gibi bu çözüm, kullanılabilirliği doğrudan ele almaz. Ancak, Azure ve şirket içi çözümler bu çözüm içinde, dahil edilen tüm bileşenler için yüksek kullanılabilirlik sağlamak üzere uygulanabilir.

### <a name="when-to-use-this-pattern"></a>Bu düzenin kullanılacağı durumlar

- Kuruluşunuzun özel bölgesel güvenlik ve dağıtım ilkeleri gerektiren Uluslararası dalları vardır.

- Kuruluşunuzun ofislerinin her biri, her yerel yönetmeliğe ve saat dilimlerine göre raporlama etkinliği gerektiren çalışan, iş ve tesis verilerini çeker.

- Yüksek ölçekli gereksinimler, tek bir bölgede birden çok uygulama dağıtımına sahip uygulamaları yatay olarak ölçeklendirerek ve çok fazla yük gereksinimini işlemek için bölgeler arasında karşılanır.

### <a name="planning-the-topology"></a>Topolojiyi planlama

Dağıtılmış bir uygulama ayak izi oluşturmadan önce, aşağıdaki şeyleri öğrenmenize yardımcı olur:

- **Uygulama Için özel etki alanı:** Müşterilerin uygulamaya erişmek için kullanacağı özel etki alanı adı nedir? Örnek uygulama için, özel etki alanı adı *www \. scalableasedemo.com* ' dir.

- **Traffic Manager etki alanı:** Bir [Azure Traffic Manager profili](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-profiles)oluştururken bir etki alanı adı seçilir. Bu ad, Traffic Manager tarafından yönetilen bir etki alanı girdisini kaydetmek için *trafficmanager.net* sonekiyle birleştirilir. Örnek uygulama için, seçilen ad *ölçeklenebilir-Ao-demo*' dir. Sonuç olarak, Traffic Manager tarafından yönetilen tam etki alanı adı *Scalable-ASE-demo.trafficmanager.net*' dir.

- **Uygulama parmak izini ölçeklendirmeye yönelik strateji:** Uygulama parmak izin tek bir bölgede, birden çok bölgede veya her iki yaklaşımın bir karışımında birden çok App Service ortamına dağıtılıp dağıtılmayacağına karar verin. Karar, müşteri trafiğinin nereden kaynaklanacaktır ve bir uygulamanın destekleme arka uç altyapısının ne kadar iyi ölçeklendirilebileceğine ilişkin beklentileri temel almalıdır. Örneğin, %100 durum bilgisi içermeyen bir uygulamayla, Azure bölgesi başına birden çok App Service ortamının bir birleşimi kullanılarak, birden fazla Azure bölgesinde dağıtılan App Service ortamları ile çarpılarak bir uygulama daha büyük bir şekilde ölçeklendirilebilir. Üzerinde seçim yapabileceğiniz 15 + küresel Azure bölgesi sayesinde müşteriler gerçek anlamda dünya genelinde bir hiper ölçekli uygulama ayak izi oluşturabilir. Burada kullanılan örnek uygulama için, tek bir Azure bölgesinde üç App Service ortamı oluşturulmuştur (Orta Güney ABD).

- **App Service ortamları Için adlandırma kuralı:** Her App Service ortamı benzersiz bir ad gerektirir. Bir veya iki App Service ortamının ötesinde, her bir App Service ortamının tanımlanmasına yardımcı olmak için bir adlandırma kuralı olması yararlı olacaktır. Burada kullanılan örnek uygulama için basit bir adlandırma kuralı kullanılmıştır. Üç App Service ortamının adları *fe1ase*, *fe2ase*ve *fe3ase*.

- **Uygulamalar Için adlandırma kuralı:** Uygulamanın birden çok örneği dağıtılırsa, dağıtılan uygulamanın her örneği için bir ad gereklidir. Power Apps için App Service Ortamı ile aynı uygulama adı birden çok ortamda kullanılabilir. Her App Service ortamı benzersiz bir etki alanı sonekine sahip olduğundan, geliştiriciler her ortamda tam olarak aynı uygulama adını kullanmayı seçebilir. Örneğin, bir geliştirici şu şekilde adlandırılan uygulamalara sahip olabilir: *MyApp.foo1.p.azurewebsites.net*, *MyApp.Foo2.p.azurewebsites.net*, *MyApp.Foo3.p.azurewebsites.net*, vb. Burada kullanılan uygulama için, her bir uygulama örneğinin benzersiz bir adı vardır. Kullanılan uygulama örneği adları *webfrontend1*, *webfrontend2*ve *webfrontend3*.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub, Azure uzantısıdır. Azure Stack hub, bulut bilgi işlemin çevikliğini ve yeniliklerini şirket içi ortamınıza sunarak, karma uygulamaları her yerde derleyip dağıtmanıza imkan tanıyan tek karma bulutu etkinleştirir.  
> 
> [Karma uygulama tasarımı ile ilgili önemli noktalar](overview-app-design-considerations.md) , karma uygulamalar tasarlamak, dağıtmak ve çalıştırmak için yazılım kalitesinin (yerleştirme, ölçeklenebilirlik, kullanılabilirlik, dayanıklılık, yönetilebilirlik ve güvenlik) aynı şekilde gözden geçirmeleri inceler. Tasarım konuları karma uygulama tasarımını iyileştirirken, üretim ortamlarındaki zorlukları en aza indirmeyle ilgili olarak size yardımcı olur.

## <a name="part-1-create-a-geo-distributed-app"></a>1. kısım: coğrafi olarak dağıtılmış bir uygulama oluşturma

Bu bölümde, bir Web uygulaması oluşturacaksınız.

> [!div class="checklist"]
> - Web uygulamaları oluşturun ve yayımlayın.
> - Azure Repos kod ekleyin.
> - Uygulamanın yapısını birden çok bulut hedeflerine işaret edin.
> - CD işlemini yönetin ve yapılandırın.

### <a name="prerequisites"></a>Ön koşullar

Bir Azure aboneliği ve Azure Stack hub yüklemesi gereklidir.

### <a name="geo-distributed-app-steps"></a>Coğrafi olarak dağıtılmış uygulama adımları

### <a name="obtain-a-custom-domain-and-configure-dns"></a>Özel etki alanı edinme ve DNS 'yi yapılandırma

Etki alanı için DNS bölge dosyasını güncelleştirin. Daha sonra Azure AD, özel etki alanı adının sahipliğini doğrulayabilirler. Azure 'da Azure/Office 365/dış DNS kayıtları için [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) kullanın veya DNS girişini [farklı bir DNS kaydedicisinde](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)ekleyin.

1. Özel bir etki alanını ortak bir kayıt defteri ile kaydedin.

2. Etki alanına ilişkin etki alanı adı kayıt şirketinde oturum açın. DNS güncelleştirmelerini yapmak için onaylanan yönetici gerekli olabilir.

3. Azure AD tarafından belirtilen DNS girişini ekleyerek etki alanı için DNS bölge dosyasını güncelleştirin. DNS girişi, posta yönlendirme veya Web barındırma gibi davranışları değiştirmez.

### <a name="create-web-apps-and-publish"></a>Web uygulamaları oluşturma ve yayımlama

Web uygulamasını Azure 'a ve Azure Stack hub 'a dağıtmak için karma sürekli tümleştirme/sürekli teslim (CI/CD) ayarlayın ve değişiklikleri her iki bulutda otomatik olarak gönderin.

> [!Note]  
> Çalıştırmak için (Windows Server ve SQL) dağıtılmış uygun görüntülerle Azure Stack hub ve App Service dağıtımı gereklidir. Daha fazla bilgi için, [Azure Stack hub 'ında App Service dağıtmaya yönelik önkoşullar](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)bölümüne bakın.

#### <a name="add-code-to-azure-repos"></a>Azure Repos kod ekleme

1. Azure Repos üzerinde **Proje oluşturma hakları** olan bir hesapla Visual Studio 'da oturum açın.

    CI/CD, hem uygulama kodu hem de altyapı kodu için uygulanabilir. Hem özel hem de barındırılan bulut geliştirmesi için [Azure Resource Manager şablonları](https://azure.microsoft.com/resources/templates/) kullanın.

    ![Visual Studio 'da bir projeye bağlanma](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. Varsayılan Web uygulamasını oluşturarak ve açarak **depoyu kopyalayın** .

    ![Visual Studio 'da depoyu Kopyala](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a>Her iki bulutta Web uygulaması dağıtımı oluşturma

1. **WebApplication. csproj** dosyasını düzenleyin: seçin `Runtimeidentifier` ve ekleyin `win10-x64` . (Bkz. [kendi Içinde dağıtım](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) belgeleri.)

    ![Visual Studio 'da Web uygulaması proje dosyasını düzenleme](media/solution-deployment-guide-geo-distributed/image3.png)

2. Takım Gezgini kullanarak **Azure Repos kodu Iade edin** .

3. **Uygulama kodunun** Azure Repos işaretli olduğunu doğrulayın.

### <a name="create-the-build-definition"></a>Derleme tanımı oluşturma

1. Derleme tanımları oluşturma yeteneğini onaylamak için **Azure pipelines ' de oturum açın** .

2. `-r win10-x64`Kod ekleyin. Bu ek, .NET Core ile bağımsız bir dağıtımı tetiklemek için gereklidir.

    ![Azure Pipelines içindeki derleme tanımına kod ekleme](media/solution-deployment-guide-geo-distributed/image4.png)

3. **Derlemeyi çalıştırın**. [Kendi içinde çalışan dağıtım oluşturma](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) Işlemi, Azure 'da ve Azure Stack hub 'da çalışabilecek yapıtları yayımlar.

#### <a name="using-an-azure-hosted-agent"></a>Azure barındırılan Aracısı kullanma

Azure Pipelines barındırılan bir aracının kullanılması, Web uygulamaları oluşturmak ve dağıtmak için kullanışlı bir seçenektir. Bakım ve yükseltmeler, kesintisiz geliştirme, test ve dağıtım sağlayan Microsoft Azure tarafından otomatik olarak gerçekleştirilir.

### <a name="manage-and-configure-the-cd-process"></a>CD işlemini yönetme ve yapılandırma

Azure DevOps Services, yayınlar için geliştirme, hazırlık, QA ve üretim ortamları gibi birden çok ortama yüksek düzeyde yapılandırılabilir ve yönetilebilir bir işlem hattı sağlar; belirli aşamalarda onay gerektirme de dahil.

## <a name="create-release-definition"></a>Yayın tanımı oluştur

1. Azure DevOps Services **derleme ve yayınlama** bölümündeki **yayınlar** sekmesinin altına yeni bir yayın eklemek için **artı** düğmesini seçin.

    ![Azure DevOps Services bir yayın tanımı oluşturun](media/solution-deployment-guide-geo-distributed/image5.png)

2. Azure App Service Dağıtım şablonunu uygulayın.

   ![Azure DevOps Services Azure App Service dağıtım şablonu Uygula](media/solution-deployment-guide-geo-distributed/image6.png)

3. **Yapıt Ekle**' nin altında, Azure Cloud Build uygulaması için yapıt ekleyin.

   ![Azure DevOps Services içinde Azure Cloud derlemesine yapıt ekleme](media/solution-deployment-guide-geo-distributed/image7.png)

4. İşlem hattı sekmesi altında, ortamın **aşamasını, görev** bağlantısını seçin ve Azure bulut ortamı değerlerini ayarlayın.

   ![Azure DevOps Services 'de Azure bulut ortamı değerlerini ayarlama](media/solution-deployment-guide-geo-distributed/image8.png)

5. **Ortam adını** ayarlayın ve Azure bulut uç noktası için **Azure aboneliğini** seçin.

      ![Azure DevOps Services 'de Azure bulut uç noktası için Azure aboneliğini seçin](media/solution-deployment-guide-geo-distributed/image9.png)

6. **App Service adı**altında, gerekli Azure App Service adını ayarlayın.

      ![Azure App Service adını Azure DevOps Services ayarla](media/solution-deployment-guide-geo-distributed/image10.png)

7. Azure bulut barındırılan ortamı için **Aracı kuyruğu** altında "barındırılan VS2017" yazın.

      ![Azure DevOps Services 'de Azure bulut barındırılan ortamı için aracı kuyruğunu ayarla](media/solution-deployment-guide-geo-distributed/image11.png)

8. Azure App Service dağıt menüsünde, ortam için geçerli **paketi veya klasörü** seçin. **Klasör konumuna** **Tamam ' ı** seçin.
  
      ![Azure DevOps Services Azure App Service ortam için paket veya klasör seçin](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Azure DevOps Services Azure App Service ortam için paket veya klasör seçin](media/solution-deployment-guide-geo-distributed/image13.png)

9. Tüm değişiklikleri kaydedin ve **yayın ardışık düzenine**geri dönün.

    ![Yayın işlem hattındaki değişiklikleri Azure DevOps Services Kaydet](media/solution-deployment-guide-geo-distributed/image14.png)

10. Azure Stack Hub uygulaması için yapıyı seçerek yeni bir yapıt ekleyin.

    ![Azure DevOps Services Azure Stack Hub uygulaması için yeni yapıt ekleme](media/solution-deployment-guide-geo-distributed/image15.png)


11. Azure App Service dağıtımını uygulayarak bir ortam daha ekleyin.

    ![Azure DevOps Services içinde Azure App Service dağıtımına ortam ekleme](media/solution-deployment-guide-geo-distributed/image16.png)

12. Yeni ortam Azure Stack hub 'ına adlandırın.

    ![Azure DevOps Services içinde Azure App Service dağıtımında ad ortamı](media/solution-deployment-guide-geo-distributed/image17.png)

13. **Görev** sekmesinde Azure Stack hub ortamını bulun.

    ![Azure DevOps Services Azure DevOps Services Azure Stack hub ortamı](media/solution-deployment-guide-geo-distributed/image18.png)

14. Azure Stack hub uç noktası için aboneliği seçin.

    ![Azure DevOps Services Azure Stack hub uç noktası için abonelik seçin](media/solution-deployment-guide-geo-distributed/image19.png)

15. Azure Stack Hub web uygulaması adını App Service adı olarak ayarlayın.

    ![Azure DevOps Services Azure Stack Hub web uygulaması adını ayarlama](media/solution-deployment-guide-geo-distributed/image20.png)

16. Azure Stack hub aracısını seçin.

    ![Azure DevOps Services Azure Stack hub aracısını seçin](media/solution-deployment-guide-geo-distributed/image21.png)

17. Dağıtım Azure App Service bölümünde, ortam için geçerli **paketi veya klasörü** seçin. Klasör konumuna **Tamam ' ı** seçin.

    ![Azure DevOps Services Azure App Service dağıtım için klasör seçin](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Azure DevOps Services Azure App Service dağıtım için klasör seçin](media/solution-deployment-guide-geo-distributed/image23.png)

18. Değişken sekmesi altında adlı bir değişken ekleyin `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , değerini **doğru**olarak ayarlayın ve Azure Stack hub ile kapsamını belirleyin.

    ![Azure DevOps Services Azure Uygulama dağıtımına değişken ekleme](media/solution-deployment-guide-geo-distributed/image24.png)

19. Her iki yapıt içinde **sürekli** dağıtım tetikleme simgesini seçin ve **devam** eden dağıtım tetikleyicisini etkinleştirin.

    ![Azure DevOps Services içinde sürekli dağıtım tetikleyicisi seçin](media/solution-deployment-guide-geo-distributed/image25.png)

20. Azure Stack hub ortamında **dağıtım öncesi** koşullar simgesini seçin ve tetikleyiciyi **yayından sonra** olarak ayarlayın.

    ![Azure DevOps Services dağıtım öncesi koşullarını seçin](media/solution-deployment-guide-geo-distributed/image26.png)

21. Tüm değişiklikleri kaydedin.

> [!Note]  
> Görevler için bazı ayarlar, bir şablondan bir yayın tanımı oluşturulurken otomatik olarak [ortam değişkenleri](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) olarak tanımlanabilir. Bu ayarlar görev ayarlarında değiştirilemez; Bunun yerine, bu ayarları düzenlemek için üst ortam öğesinin seçilmesi gerekir.

## <a name="part-2-update-web-app-options"></a>2. Bölüm: Web uygulaması seçeneklerini güncelleştirme

[Azure App Service](https://docs.microsoft.com/azure/app-service/overview), yüksek oranda ölçeklenebilen, kendi kendine düzeltme eki uygulayan bir web barındırma hizmeti sunar.

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - Mevcut bir özel DNS adını Azure Web Apps eşleştirin.
> - Özel bir DNS adını App Service eşlemek için bir **CNAME kaydı** ve **a kaydı** kullanın.

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a>Mevcut bir özel DNS adını Azure Web Apps ile eşleme

> [!Note]  
> Kök etki alanı dışındaki tüm özel DNS adları için CNAME kullanın (örneğin, northwind.com).

Canlı siteyi ve onun DNS etki alanı adını App Service'e geçirmek için, bkz. [Etkin DNS adını Azure App Service'e geçirme](https://docs.microsoft.com/azure/app-service/manage-custom-dns-migrate-domain).

### <a name="prerequisites"></a>Ön koşullar

Bu çözümü gerçekleştirmek için:

- [App Service bir uygulama oluşturun](https://docs.microsoft.com/azure/app-service/)veya başka bir çözüm için oluşturulmuş bir uygulama kullanın.

- Etki alanı adı satın alıp etki alanı sağlayıcısı için DNS kayıt defterine erişim sağlayın.

Etki alanı için DNS bölge dosyasını güncelleştirin. Azure AD, özel etki alanı adının sahipliğini doğrulayacaktır. Azure 'da Azure/Office 365/dış DNS kayıtları için [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) kullanın veya DNS girişini [farklı bir DNS kaydedicisinde](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)ekleyin.

- Özel bir etki alanını ortak bir kayıt defteri ile kaydedin.

- Etki alanına ilişkin etki alanı adı kayıt şirketinde oturum açın. (DNS güncelleştirmeleri yapmak için onaylanan yönetici gerekli olabilir.)

- Azure AD tarafından belirtilen DNS girişini ekleyerek etki alanı için DNS bölge dosyasını güncelleştirin.

Örneğin, northwindcloud.com ve www northwindcloud.com için DNS girişleri eklemek için \. , northwindcloud.com kök etki alanı IÇIN DNS ayarlarını yapılandırın.

> [!Note]  
> [Azure Portal](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain)kullanarak bir etki alanı adı satın alınabilir. Özel DNS adını web uygulamasına eşlemek için, web uygulamasının [App Service planı](https://azure.microsoft.com/pricing/details/app-service/) ücretli bir katmanda (**Paylaşılan**, **Temel**, **Standart** veya **Premium** olmalıdır).

### <a name="create-and-map-cname-and-a-records"></a>CNAME ve A kayıtlarını oluşturma ve eşleme

#### <a name="access-dns-records-with-domain-provider"></a>Etki alanı sağlayıcısı ile DNS kayıtlarına erişme

> [!Note]  
>  Azure Web Apps için özel bir DNS adı yapılandırmak üzere Azure DNS kullanın. Daha fazla bilgi için bkz. [Bir Azure hizmeti için özel etki alanı ayarları sağlamak üzere Azure DNS'yi kullanma](https://docs.microsoft.com/azure/dns/dns-custom-domain).

1. Ana sağlayıcının web sitesinde oturum açın.

2. DNS kayıtlarını yönetme sayfasını bulun. Her etki alanı sağlayıcısının kendi DNS kayıtları arabirimi vardır. Sitede **Domain Name**, **DNS** veya **Name Server Management** etiketli alanları bulun.

DNS kayıtları sayfası, **etki Alanlarmda**görüntülenebilir. **Bölge dosyası**, **DNS kayıtları**veya **Gelişmiş yapılandırma**adlı bağlantıyı bulun.

DNS kayıtları sayfasının bir örneğini aşağıdaki ekran görüntüsünde görebilirsiniz:

![Örnek DNS kayıtları sayfası](media/solution-deployment-guide-geo-distributed/image28.png)

1. Kayıt oluşturmak için etki alanı adı kaydedicisi ' nde **Ekle veya oluştur** ' u seçin. Bazı sağlayıcıların farklı kayıt türlerini eklemek için farklı bağlantıları vardır. Sağlayıcının belgelerine başvurun.

2. Bir alt etki alanını uygulamanın varsayılan ana bilgisayar adına eşlemek için bir CNAME kaydı ekleyin.

   Www \. northwindcloud.com etki alanı örneği için, adı ile eşleyen BIR CNAME kaydı ekleyin `<app_name>.azurewebsites.net` .

CNAME eklendikten sonra DNS kayıtları sayfası aşağıdaki örneğe benzer şekilde görünür:

![Azure uygulamasına portal gezintisi](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a>Azure'da CNAME kaydı eşlemesini etkinleştirme

1. Yeni bir sekmede Azure portal oturum açın.

2. Uygulama Hizmetleri'ne gidin.

3. Web uygulaması ' nı seçin.

4. Azure Portal'daki uygulama sayfasının sol gezintisinde **Özel etki alanları**'nı seçin.

5. **+** **Konak adı Ekle**' nin yanındaki simgeyi seçin.

6. Tam etki alanı adını (gibi) yazın `www.northwindcloud.com` .

7. **Doğrula**'yı seçin.

8. Belirtilmişse, `A` `TXT` etki alanı adı kayıt şirketlerinde DNS kayıtlarına diğer türlerin (veya) ek kayıtlarını ekleyin. Azure, bu kayıtların değerlerini ve türlerini sağlar:

   a.  Uygulamanın IP adresini eşlemek için bir **A** kaydı.

   b.  Uygulamanın varsayılan konak adını (`<app_name>.azurewebsites.net`) eşlemek için bir **TXT** kaydı. App Service, özel etki alanı sahipliğini doğrulamak için bu kaydı yalnızca yapılandırma zamanında kullanır. Doğrulamadan sonra TXT kaydını silin.

9. Bu görevi etki alanı kaydedici sekmesinde doldurun ve **ana bilgisayar adı Ekle** düğmesi etkinleştirilinceye kadar yeniden doğrulayın.

10. **Ana bilgisayar adı kayıt türünün** **CNAME** (www.example.com veya herhangi bir alt etki alanı) olarak ayarlandığından emin olun.

11. **Konak adı ekle**'yi seçin.

12. Tam etki alanı adını (gibi) yazın `northwindcloud.com` .

13. **Doğrula**'yı seçin. **Ekleme** etkinleştirilir.

14. **Ana bilgisayar adı kayıt türünün** **bir kayıt** (example.com) olarak ayarlandığından emin olun.

15. **Konak adı Ekle**.

    Yeni ana bilgisayar adlarının uygulamanın **özel etki alanları** sayfasında yansıtılması biraz zaman alabilir. Verileri güncelleştirmek için tarayıcıyı yenilemeyi deneyin.
  
    ![Özel etki alanları](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    Bir hata oluşursa, sayfanın alt kısmında bir doğrulama hatası bildirimi görüntülenir. ![Etki alanı doğrulama hatası](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  Yukarıdaki adımlar, joker bir etki alanını ( \* . northwindcloud.com) eşlemek için yinelenebilir. Bu, her biri için ayrı bir CNAME kaydı oluşturmak zorunda kalmadan bu App Service 'e ek alt etki alanları eklenmesine izin verir. Bu ayarı yapılandırmak için kaydedici yönergelerini izleyin.

#### <a name="test-in-a-browser"></a>Bir tarayıcıda test etme

Daha önce yapılandırılan DNS adlarını (örneğin, `northwindcloud.com` veya `www.northwindcloud.com` ) inceleyin.

## <a name="part-3-bind-a-custom-ssl-cert"></a>3. kısım: özel bir SSL sertifikası bağlama

Bu bölümde şunları göndereceğiz:

> [!div class="checklist"]
> - Özel SSL sertifikasını App Service bağlayın.
> - Uygulama için HTTPS 'yi zorunlu tutun.
> - SSL sertifikası bağlamasını betiklerle otomatikleştirin.

> [!Note]  
> Gerekirse, Azure portal bir müşteri SSL sertifikası alın ve Web uygulamasına bağlayın. Daha fazla bilgi için [App Service sertifikaları öğreticisine](https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site)bakın.

### <a name="prerequisites"></a>Ön koşullar

Bu çözümü gerçekleştirmek için:

- [App Service uygulaması oluşturun.](https://docs.microsoft.com/azure/app-service/)
- [Özel bir DNS adını Web uygulamanıza eşleyin.](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)
- Güvenilir bir sertifika yetkilisinden SSL sertifikası alın ve bu anahtarı kullanarak isteği imzalayın.

### <a name="requirements-for-your-ssl-certificate"></a>SSL sertifikanıza yönelik gereksinimler

Bir sertifikayı App Service’te kullanabilmek için sertifikanın aşağıdaki tüm gereksinimleri karşılaması gerekir:

- Güvenilen bir sertifika yetkilisi tarafından imzalandı.

- Parola korumalı bir PFX dosyası olarak verildi.

- En az 2048 bit uzunluğunda özel anahtar içerir.

- Sertifika zincirindeki tüm ara sertifikaları içerir.

> [!Note]  
> **Eliptik Eğri Şifreleme (ECC) sertifikaları** App Service ile çalışır, ancak bu kılavuza dahil değildir. ECC sertifikaları oluşturma konusunda yardım almak için bir sertifika yetkilisine başvurun.

#### <a name="prepare-the-web-app"></a>Web uygulamasını hazırlama

Özel bir SSL sertifikasını Web uygulamasına bağlamak için [App Service planının](https://azure.microsoft.com/pricing/details/app-service/) **temel**, **Standart**veya **Premium** katmanda olması gerekir.

#### <a name="sign-in-to-azure"></a>Azure'da oturum açma

1. [Azure Portal](https://portal.azure.com/) açın ve Web uygulamasına gidin.

2. Sol menüden **uygulama hizmetleri**' ni ve ardından Web uygulaması adını seçin.

![Azure portal web uygulaması seçin](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a>Fiyatlandırma katmanını denetleme

1. Web uygulaması sayfasının sol tarafında, **Ayarlar** bölümüne gidin ve **ölçeği büyütme (App Service planı)** seçeneğini belirleyin.

    ![Web uygulamasındaki ölçek menüsü](media/solution-deployment-guide-geo-distributed/image34.png)

1. Web uygulamasının **ücretsiz** veya **paylaşılan** katmanda olmadığından emin olun. Web uygulamasının geçerli katmanı, koyu mavi bir kutu içinde vurgulanır.

    ![Web uygulamasındaki fiyatlandırma katmanını denetle](media/solution-deployment-guide-geo-distributed/image35.png)

Özel SSL, **ücretsiz** veya **paylaşılan** katmanda desteklenmez. Üst ölçekte, sonraki bölümde bulunan adımları izleyin veya **fiyatlandırma katmanınızı seçin** SAYFASıNDA, [SSL sertifikanızı karşıya yüklemek ve bağlamak](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl)için atlayın.

#### <a name="scale-up-your-app-service-plan"></a>App Service planınızın ölçeğini artırma

1. **Temel**, **Standart** veya **Premium** katmanlarından birini seçin.

2. **Seç**’i seçin.

![Web uygulamanız için fiyatlandırma katmanı seçin](media/solution-deployment-guide-geo-distributed/image36.png)

Bildirim görüntülenirken, ölçek işlemi tamamlanır.

![Ölçek artırma bildirimi](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a>SSL sertifikanızı bağlama ve ara sertifikaları birleştirme

Zincirdeki birden çok sertifikayı birleştirin.

1. Bir metin düzenleyicisinde aldığınız **her sertifikayı açın** .

2. *Birleştirilebilen sertifika. CRT*adlı birleştirilmiş sertifika için bir dosya oluşturun. Bir metin düzenleyicisinde her bir sertifikanın içeriğini bu dosyaya kopyalayın. Sertifikalarınızın sırası, sertifikanızla başlayıp kök sertifika ile sona ererek sertifika zincirindeki sırayla aynı olmalıdır. Aşağıdaki örneğe benzer şekilde görünür:

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a>Sertifikayı PFX dosyasına aktarma

Birleştirilmiş SSL sertifikasını sertifika tarafından oluşturulan özel anahtarla dışarı aktarın.

OpenSSL aracılığıyla bir özel anahtar dosyası oluşturulur. Sertifikayı PFX 'e aktarmak için aşağıdaki komutu çalıştırın ve yer tutucuları `<private-key-file>` ve `<merged-certificate-file>` özel anahtar yolu ve birleştirilmiş sertifika dosyası ile değiştirin:

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

İstendiğinde, daha sonra App Service SSL sertifikanızı karşıya yüklemek için bir dışarı aktarma parolası tanımlayın.

Sertifika isteğini oluşturmak için IIS veya **Certreq.exe** kullanıldığında, sertifikayı yerel bir makineye yükler ve ardından [sertifikayı PFX 'e dışarı aktarın](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx).

#### <a name="upload-the-ssl-certificate"></a>SSL sertifikasını karşıya yükleme

1. Web uygulamasının sol gezinti bölmesinde **SSL ayarları** ' nı seçin.

2. **Sertifikayı karşıya yükle**' yi seçin.

3. **PFX Sertifika dosyasında**pfx dosyası ' nı seçin.

4. **Sertifika parolası**' nda, pfx dosyası dışarı aktarılırken oluşturulan parolayı yazın.

5. **Karşıya Yükle**'yi seçin.

    ![SSL sertifikasını karşıya yükle](media/solution-deployment-guide-geo-distributed/image38.png)

App Service sertifikayı karşıya yüklemeyi bitirdiğinde, bu, **SSL ayarları** sayfasında görüntülenir.

![SSL Ayarları](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a>SSL sertifikanızı bağlama

1. **SSL bağlamaları** bölümünde **bağlama Ekle**' yi seçin.

    > [!Note]  
    >  Sertifika karşıya yüklenmişse, ancak **ana bilgisayar** adı açılır listesinde etki alanı adında görünmüyorsa, tarayıcı sayfasını yenilemeyi deneyin.

2. **SSL bağlaması Ekle** sayfasında, güvenli hale getirmek için etki alanı adını ve kullanılacak sertifikayı seçmek için açılan listeleri kullanın.

3. **SSL Türü** menüsünde [**Sunucu Adı Belirtme (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) veya IP tabanlı SSL seçeneklerinden hangisini kullanacağınızı belirleyin.

    - **SNI tabanlı SSL**: birden çok SNı tabanlı SSL bağlamaları eklenebilir. Bu seçenek, aynı IP adresi üzerinde birden fazla SSL sertifikası ile birden fazla etki alanının güvenliğini sağlamaya olanak tanır. Çoğu modern tarayıcı (Internet Explorer, Chrome, Firefox ve Opera dahil) SNI’yi destekler (daha kapsamlı tarayıcı desteği bilgilerini [Sunucu Adı Belirtimi](https://wikipedia.org/wiki/Server_Name_Indication) bölümünde bulabilirsiniz).

    - **IP tabanlı SSL**: yalnızca bir IP tabanlı SSL bağlaması eklenebilir. Bu seçenek yalnızca bir SSL sertifikası ile ayrılmış bir genel IP adresinin güvenliğini sağlamaya olanak tanır. Birden çok etki alanının güvenliğini sağlamak için, bunların tümünü aynı SSL sertifikası ile güvenli hale getirin. IP tabanlı SSL, SSL bağlama için geleneksel bir seçenektir.

4. **Bağlama Ekle**' yi seçin.

    ![SSL bağlaması Ekle](media/solution-deployment-guide-geo-distributed/image40.png)

App Service sertifikayı karşıya yüklemeyi bitirdiğinde, **SSL bağlamaları** bölümlerinde görüntülenir.

![SSL bağlamaları karşıya yüklemeyi bitirdi](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a>IP SSL için bir kaydı yeniden eşleyin

Web uygulamasında IP tabanlı SSL kullanılmıyorsa, [özel etki alanınız Için test https](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl)'ye atlayın.

Varsayılan olarak, Web uygulaması paylaşılan bir genel IP adresi kullanır. Sertifika, IP tabanlı SSL ile bağlandığında, App Service Web uygulaması için yeni ve ayrılmış bir IP adresi oluşturur.

Bir kayıt Web uygulamasına eşlendiğinde, etki alanı kayıt defteri ayrılmış IP adresi ile birlikte güncelleştirilmeleri gerekir.

**Özel etki alanı** sayfası, yeni ve ayrılmış IP adresi ile güncelleştirilir. Bu [IP adresini](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)kopyalayın ve ardından [bir kaydı](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain) bu yeni IP adresiyle yeniden eşleyin.

#### <a name="test-https"></a>HTTPS’yi test etme

Farklı tarayıcılarda, `https://<your.custom.domain>` Web uygulamasının sunulmasını sağlamak için bölümüne gidin.

![Web uygulamasına gidin](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> Sertifika doğrulama hataları oluşursa, kendinden imzalı bir sertifika neden olabilir ya da PFX dosyasına aktarılırken ara sertifikalar bırakılmış olabilir.

#### <a name="enforce-https"></a>HTTPS'yi zorunlu tutma

Varsayılan olarak, herkes HTTP kullanarak Web uygulamasına erişebilir. HTTPS bağlantı noktasına yapılan tüm HTTP istekleri yeniden yönlendirilebilir.

Web uygulaması sayfasında **SL ayarları**' nı seçin. Ardından **Yalnızca HTTPS** menüsünde **Açık**’ı seçin.

![HTTPS'yi zorunlu tutma](media/solution-deployment-guide-geo-distributed/image43.png)

İşlem tamamlandığında, uygulamayı işaret eden HTTP URL 'Lerinden birine gidin. Örneğin:

- https://<app_name>. azurewebsites.net
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a>TLS 1.1/1.2 zorlama

Uygulama varsayılan olarak, artık sektör standartları ( [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)gibi) tarafından güvenli olarak kabul edilen [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1,0 ' i sağlar. Daha yüksek TLS sürümlerini zorlamak için şu adımları izleyin:

1. Web uygulaması sayfasında, sol gezinti bölmesinde **SSL ayarları**' nı seçin.

2. **TLS sürümü**' nde, en düşük TLS sürümünü seçin.

    ![TLS 1.1 veya 1.2’yi zorlama](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a>Traffic Manager profili oluşturma

1. **Create a resource**  >  **Networking**  >  **Profil**  >  **Oluştur**Traffic Manager kaynak ağı oluştur ' u seçin.

2. **Traffic Manager profili oluştur** dikey penceresini aşağıdaki gibi doldurun:

    1. **Ad**alanına profil için bir ad girin. Bu adın trafik manager.net bölgesinde benzersiz olması gerekir ve Traffic Manager profiline erişmek için kullanılan DNS adı, trafficmanager.net ile sonuçlanır.

    2. **Yönlendirme yöntemi**' nde **coğrafi yönlendirme yöntemini**seçin.

    3. **Abonelikte**, bu profilin oluşturulacağı aboneliği seçin.

    4. **Kaynak Grubu** alanında bu profilin yerleştirileceği yeni bir kaynak grubu oluşturun.

    5. **Kaynak grubu konumu** alanında kaynak grubunun konumunu seçin. Bu ayar, kaynak grubunun konumunu ifade eder ve genel olarak dağıtılan Traffic Manager profilini etkilemez.

    6. **Oluştur**'u seçin.

    7. Traffic Manager profilinin genel dağıtımı tamamlandığında, ilgili kaynak grubunda kaynaklardan biri olarak listelenir.

        ![Traffic Manager profili oluşturma içindeki kaynak grupları](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a>Traffic Manager uç noktalarını ekleme

1. Portal arama çubuğunda, yukarıdaki bölümde oluşturulan **Traffic Manager profili** adını arayın ve görünen sonuçlarda Traffic Manager profilini seçin.

2. **Traffic Manager profili**' nde, **Ayarlar** bölümünde **uç noktalar**' ı seçin.

3. **Ekle**'yi seçin.

4. Azure Stack hub uç noktası ekleniyor.

5. **Tür**için **dış uç nokta**' ı seçin.

6. Bu uç **nokta için,** ideal olarak Azure Stack merkezinin adını girin.

7. Tam etki alanı adı (**FQDN**) için, Azure Stack Hub web uygulaması için dış URL 'yi kullanın.

8. Coğrafi eşleme altında kaynağın bulunduğu bölgeyi/kıta seçin. Örneğin, **Avrupa.**

9. Görüntülenen ülke/bölge açılır kısmında, bu uç nokta için geçerli olan ülkeyi seçin. Örneğin, **Almanya**.

10. **Devre dışı olarak ekle** seçeneğini işaretsiz bırakın.

11. **Tamam**’ı seçin.

12. Azure Uç Noktası Ekleme:

    1. **Tür**için **Azure uç noktası**' nı seçin.

    2. Uç nokta için bir **ad** girin.

    3. **Hedef kaynak türü**için **App Service**seçin.

    4. **Hedef kaynak**için, aynı abonelikte Web Apps listesini göstermek üzere **bir App Service seçin** öğesini seçin. **Kaynak**bölümünde ilk uç nokta olarak kullanılan App Service 'i seçin.

13. Coğrafi eşleme altında kaynağın bulunduğu bölgeyi/kıta seçin. Örneğin, **Kuzey Amerika/Orta Amerika/Karayipler.**

14. Görüntülenen ülke/bölge açılır listesi altında, yukarıdaki bölgesel gruplamayı seçmek için bu noktayı boş bırakın.

15. **Devre dışı olarak ekle** seçeneğini işaretsiz bırakın.

16. **Tamam**’ı seçin.

    > [!Note]  
    >  Kaynak için varsayılan uç nokta olarak kullanılacak bir coğrafi kapsamı (Dünya) olan en az bir uç nokta oluşturun.

17. Her iki uç noktanın eklenmesi tamamlandığında, bunlar **Traffic Manager profilinde** görüntülenir ve bunların Izleme durumu **çevrimiçi**olarak gösterilir.

    ![Traffic Manager profili uç noktası durumu](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a>Küresel kurumsal, Azure coğrafi dağıtım özelliklerini kullanır

Azure Traffic Manager ve coğrafi konuma özgü uç noktalar aracılığıyla veri trafiğini yönlendirmek, küresel kuruluşların bölgesel yönetmeliklere erişmesini ve verilerin uyumlu ve güvenli kalmasını sağlar ve bu da yerel ve uzak iş konumlarının başarısı için önemlidir.

## <a name="next-steps"></a>Sonraki adımlar

- Azure bulut desenleri hakkında daha fazla bilgi edinmek için bkz. [bulut tasarım desenleri](https://docs.microsoft.com/azure/architecture/patterns).
