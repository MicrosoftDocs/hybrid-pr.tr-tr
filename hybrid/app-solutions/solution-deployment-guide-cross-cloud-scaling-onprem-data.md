---
title: Çapraz bulutu ölçeklendirirken şirket içi verilerle karma uygulama dağıtma
description: Şirket içi verileri kullanan bir uygulamayı dağıtmayı ve Azure ve Azure Stack hub kullanarak çapraz bulutu ölçeklendireceğinizi öğrenin.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ecc42a94e2c59531b2a2e933772b0d8ce8c58609
ms.sourcegitcommit: 0d5b5336bdb969588d0b92e04393e74b8f682c3b
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92353487"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a>Çapraz bulutu ölçeklendirirken şirket içi verilerle karma uygulama dağıtma

Bu çözüm kılavuzunda, hem Azure hem de Azure Stack hub 'ına yayılan ve tek bir şirket içi veri kaynağı kullanan bir karma uygulamanın nasıl dağıtılacağı gösterilir.

Karma bulut çözümünü kullanarak, özel bir bulutun uyumluluk avantajlarını genel bulutun ölçeklenebilirliği ile birleştirebilirsiniz. Geliştiricileriniz ayrıca Microsoft Developer ekosisteminden yararlanabilir ve yeteneklerini buluta ve şirket içi ortamlara uygulayabilir.

## <a name="overview-and-assumptions"></a>Genel bakış ve varsayımlar

Geliştiricilerin aynı Web uygulamasını ortak bir buluta ve özel buluta dağıtmasına imkan tanıyan bir iş akışı ayarlamak için bu öğreticiyi izleyin. Bu uygulama, özel bulutta barındırılan internet 'e ait olmayan yönlendirilebilir ağa erişebilir. Bu Web uygulamaları izlenir ve trafikte ani bir artış olduğunda, bir program trafiği genel buluta yönlendirmek için DNS kayıtlarını değiştirir. Trafik, ani bir düzeye düştüğünde trafik, özel buluta geri yönlendirilir.

Bu öğretici aşağıdaki görevleri kapsar:

> [!div class="checklist"]
> - Karma bağlantılı SQL Server veritabanı sunucusu dağıtın.
> - Küresel Azure 'da bir Web uygulamasını karma ağa bağlayın.
> - DNS 'i platformlar arası ölçekleme için yapılandırın.
> - Platformlar arası ölçeklendirme için SSL sertifikaları yapılandırın.
> - Web uygulamasını yapılandırın ve dağıtın.
> - Traffic Manager profili oluşturun ve bulut tabanlı ölçeklendirme için yapılandırın.
> - Artan trafik için Application Insights izlemeyi ve uyarı ayarlamayı ayarlayın.
> - Genel Azure ve Azure Stack hub arasında otomatik trafik geçişini yapılandırın.

> [!Tip]  
> ![Karma paragraf diyagramı](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub, Azure uzantısıdır. Azure Stack hub, bulut bilgi işlemin çevikliğini ve yeniliklerini şirket içi ortamınıza sunarak, karma uygulamaları her yerde derleyip dağıtmanıza imkan tanıyan tek karma bulutu etkinleştirir.  
> 
> [Karma uygulama tasarımı ile ilgili önemli noktalar](overview-app-design-considerations.md) , karma uygulamalar tasarlamak, dağıtmak ve çalıştırmak için yazılım kalitesinin (yerleştirme, ölçeklenebilirlik, kullanılabilirlik, dayanıklılık, yönetilebilirlik ve güvenlik) aynı şekilde gözden geçirmeleri inceler. Tasarım konuları karma uygulama tasarımını iyileştirirken, üretim ortamlarındaki zorlukları en aza indirmeyle ilgili olarak size yardımcı olur.

### <a name="assumptions"></a>Varsayımlar

Bu öğreticide, genel Azure ve Azure Stack hub ile ilgili temel bilgilere sahip olduğunuz varsayılmaktadır. Öğreticiyi başlatmadan önce daha fazla bilgi edinmek istiyorsanız, şu makaleleri gözden geçirin:

- [Azure’a giriş](https://azure.microsoft.com/overview/what-is-azure/)
- [Azure Stack hub anahtar kavramları](/azure-stack/operator/azure-stack-overview.md)

Bu öğreticide Ayrıca bir Azure aboneliğiniz olduğunu varsaymaktadır. Aboneliğiniz yoksa başlamadan önce [ücretsiz bir hesap oluşturun](https://azure.microsoft.com/free/) .

## <a name="prerequisites"></a>Ön koşullar

Bu çözüme başlamadan önce, aşağıdaki gereksinimleri karşıladığınızdan emin olun:

- Bir Azure Stack Geliştirme Seti (ASDK) veya Azure Stack hub ile tümleşik sistemdeki bir abonelik. ASDK 'yi dağıtmak için, [yükleyiciyi kullanarak asdk dağıtma](/azure-stack/asdk/asdk-install.md)bölümündeki yönergeleri izleyin.
- Azure Stack hub yüklemenizde aşağıdakilerin yüklü olması gerekir:
  - Azure App Service. Ortamınızdaki Azure App Service dağıtmak ve yapılandırmak için Azure Stack hub Işletmeniniz ile çalışın. Bu öğreticide, App Service için en az bir (1) ayrılmış çalışan rolü olması gerekir.
  - Bir Windows Server 2016 görüntüsü.
  - Microsoft SQL Server bir görüntüyle Windows Server 2016.
  - Uygun planlar ve teklifler.
  - Web uygulamanız için bir etki alanı adı. Bir etki alanı adınız yoksa, GoDaddy, BlueHost ve Inmotion gibi bir etki alanı sağlayıcısından bir tane satın alabilirsiniz.
- Letiniz gibi güvenilir bir sertifika yetkilisinden etki alanınız için bir SSL sertifikası.
- SQL Server veritabanıyla iletişim kuran ve Application Insights destekleyen bir Web uygulaması. [Dotnetcore-SQLDB-öğreticisi](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) örnek uygulamasını GitHub 'dan indirebilirsiniz.
- Azure sanal ağı ile Azure Stack hub sanal ağı arasındaki karma ağ. Ayrıntılı yönergeler için bkz. [Azure ve Azure Stack hub ile karma bulut bağlantısını yapılandırma](solution-deployment-guide-connectivity.md).

- Azure Stack hub 'ında özel yapı aracısına sahip karma bir sürekli tümleştirme/sürekli dağıtım (CI/CD) işlem hattı. Ayrıntılı yönergeler için bkz. [Azure ve Azure Stack hub uygulamalarıyla karma bulut kimliğini yapılandırma](solution-deployment-guide-identity.md).

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a>Karma bağlantılı SQL Server veritabanı sunucusu dağıtma

1. Azure Stack hub Kullanıcı portalında oturum açın.

2. **Panoda** **Market**' i seçin.

    ![Azure Stack hub marketi](media/solution-deployment-guide-hybrid/image1.png)

3. **Market**' te **işlem**' i seçin ve **daha sonra diğer**' i seçin. **Daha fazla bilgi**Için, **Windows Server görüntüsünde ücretsiz SQL Server lisansı: SQL Server 2017 Developer** ' ı seçin.

    ![Azure Stack hub Kullanıcı portalında bir sanal makine görüntüsü seçin](media/solution-deployment-guide-hybrid/image2.png)

4. **Ücretsiz SQL Server Lisansı: Windows Server 'da SQL Server 2017 geliştiricisi** **Oluştur**' u seçin.

5. **Temel ayarları yapılandırma > temel bilgiler**, sanal makıne (VM) Için bir **ad** , SQL Server SA için BIR **Kullanıcı adı** ve SA için bir **parola** sağlayın.  **Abonelik** açılan listesinden, dağıtmakta olduğunuz aboneliği seçin. **Kaynak grubu**için **var olanı seç** ' i kullanın ve VM 'yi Azure Stack Hub web uygulamanızla aynı kaynak grubuna yerleştirin.

    ![Azure Stack hub Kullanıcı portalında VM için temel ayarları yapılandırma](media/solution-deployment-guide-hybrid/image3.png)

6. **Boyut**bölümünde VM 'niz için bir boyut seçin. Bu öğretici için A2_Standard veya DS2_V2_Standard önerilir.

7. **Ayarlar > isteğe bağlı özellikleri yapılandırmak**için aşağıdaki ayarları yapılandırın:

   - **Depolama hesabı**: ihtiyacınız varsa yeni bir hesap oluşturun.
   - **Sanal ağ**:

     > [!Important]  
     > SQL Server VM VPN ağ geçitleri ile aynı sanal ağda dağıtıldığından emin olun.

   - **Genel IP adresi**: varsayılan ayarları kullanın.
   - **Ağ güvenlik grubu**: (NSG). Yeni bir NSG oluşturun.
   - **Uzantılar ve izleme**: varsayılan ayarları tutun.
   - **Tanılama depolama hesabı**: ihtiyacınız varsa yeni bir hesap oluşturun.
   - Yapılandırmanızı kaydetmek için **Tamam ' ı** seçin.

     ![Azure Stack hub Kullanıcı portalında isteğe bağlı VM özelliklerini yapılandırma](media/solution-deployment-guide-hybrid/image4.png)

8. **SQL Server ayarları**altında aşağıdaki ayarları yapılandırın:

   - **SQL bağlantısı**için **Genel (Internet)** seçeneğini belirleyin.
   - **Bağlantı noktası**için, varsayılan **1433**' ı koruyun.
   - **SQL kimlik doğrulaması**için **Etkinleştir**' i seçin.

     > [!Note]  
     > SQL kimlik doğrulamasını etkinleştirdiğinizde, **temel**bilgiler ' de yapılandırdığınız "SQLAdmin" bilgileri ile otomatik olarak doldurulur.

   - Ayarların geri kalanı için varsayılan değerleri tutun. **Tamam**’ı seçin.

     ![Azure Stack hub Kullanıcı portalında SQL Server ayarlarını yapılandırma](media/solution-deployment-guide-hybrid/image5.png)

9. **Özet**SAYFASıNDA, VM yapılandırmasını gözden geçirin ve ardından dağıtımı başlatmak için **Tamam** ' ı seçin.

    ![Azure Stack hub Kullanıcı portalında Yapılandırma Özeti](media/solution-deployment-guide-hybrid/image6.png)

10. Yeni VM 'yi oluşturmak biraz zaman alır. **Sanal makinelerde**VM 'nizin durumunu görüntüleyebilirsiniz.

    ![Azure Stack hub Kullanıcı portalında sanal makine durumu](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a>Azure 'da ve Azure Stack hub 'da Web uygulamaları oluşturma

Azure App Service, bir Web uygulamasını çalıştırmayı ve yönetmeyi basitleştirir. Azure Stack hub, Azure ile tutarlı olduğundan, App Service her iki ortamda da çalıştırılabilir. Uygulamanızı barındırmak için App Service kullanacaksınız.

### <a name="create-web-apps"></a>Web uygulamaları oluşturma

1. Azure 'da [App Service planını yönetme](/azure/app-service/app-service-plan-manage#create-an-app-service-plan)bölümündeki yönergeleri izleyerek Azure 'da bir Web uygulaması oluşturun. Web uygulamasını karma ağınızla aynı aboneliğe ve kaynak grubuna yerleştirdiğinizden emin olun.

2. Azure Stack hub 'ında önceki adımı (1) yineleyin.

### <a name="add-route-for-azure-stack-hub"></a>Azure Stack Hub için yol ekleme

Azure Stack hub üzerindeki App Service, kullanıcıların uygulamanıza erişmesine izin vermek için genel İnternet 'ten yönlendirilebilir olmalıdır. Azure Stack hub 'ınız Internet 'ten erişilebiliyorsa, Azure Stack Hub web uygulaması için herkese açık IP adresini veya URL 'YI bir yere unutmayın.

Bir ASDK kullanıyorsanız, sanal ortam dışında App Service göstermek için [statik BIR NAT eşlemesi yapılandırabilirsiniz](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) .

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a>Azure 'da bir Web uygulamasını karma ağa bağlama

Azure 'daki Web ön ucu ve Azure Stack hub 'ında SQL Server veritabanı arasında bağlantı sağlamak için, Web uygulamasının Azure ile Azure Stack hub arasında karma ağa bağlanması gerekir. Bağlantıyı etkinleştirmek için şunları yapmanız gerekir:

- Noktadan siteye bağlantıyı yapılandırın.
- Web uygulamasını yapılandırın.
- Azure Stack hub 'ında yerel ağ geçidini değiştirin.

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a>Noktadan siteye bağlantı için Azure sanal ağını yapılandırma

Karma ağın Azure tarafındaki sanal ağ geçidi, Azure App Service ile tümleştirilecek Noktadan siteye bağlantılara izin vermelidir.

1. Azure portal sanal ağ geçidi sayfasına gidin. **Ayarlar**' ın altında, **Noktadan siteye yapılandırma**' yı seçin.

    ![Azure sanal ağ geçidinde Noktadan siteye seçeneği](media/solution-deployment-guide-hybrid/image8.png)

2. Noktadan siteye yapılandırmak için **Şimdi Yapılandır** ' ı seçin.

    ![Azure sanal ağ geçidinde Noktadan siteye yapılandırmayı Başlat](media/solution-deployment-guide-hybrid/image9.png)

3. **Noktadan siteye** yapılandırma sayfasında, **adres havuzunda**kullanmak istediğiniz özel IP adresi aralığını girin.

   > [!Note]  
   > Belirttiğiniz aralığın, karma ağın genel Azure veya Azure Stack hub bileşenlerinde bulunan alt ağlar tarafından zaten kullanılmakta olan adres aralıklarıyla çakışmadığından emin olun.

   **Tünel türü**altında, **IKEv2 VPN**seçeneğinin işaretini kaldırın. Noktadan siteye yapılandırmayı sona **kazanmak Için kaydet** ' i seçin.

   ![Azure sanal ağ geçidinde Noktadan siteye ayarları](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a>Azure App Service uygulamasını karma ağla tümleştirme

1. Uygulamayı Azure VNet 'e bağlamak için [Ağ Geçidi gerekli VNET tümleştirmesi](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration)bölümündeki yönergeleri izleyin.

2. Web uygulamasını barındıran App Service planına yönelik **Ayarlar** ' a gidin. **Ayarlar**' da **ağ**' ı seçin.

    ![App Service planı için ağı yapılandırma](media/solution-deployment-guide-hybrid/image11.png)

3. **Sanal ağ tümleştirmesi**, **yönetmek Için buraya tıklayın ' ı**seçin.

    ![App Service planı için VNET tümleştirmesini yönetme](media/solution-deployment-guide-hybrid/image12.png)

4. Yapılandırmak istediğiniz VNET 'i seçin. **VNET 'e yönlendirilen IP adresleri**altında, Azure vnet, Azure Stack hub VNET ve Noktadan siteye adres ALANLARı için IP adresi aralığını girin. Bu ayarları doğrulamak ve kaydetmek için **Kaydet** ' i seçin.

    ![Sanal ağ tümleştirmesinde yönlendirileceği IP adresi aralıkları](media/solution-deployment-guide-hybrid/image13.png)

App Service Azure sanal ağları ile tümleştirme hakkında daha fazla bilgi edinmek için bkz. [uygulamanızı bir Azure sanal ağı Ile tümleştirme](/azure/app-service/web-sites-integrate-with-vnet).

### <a name="configure-the-azure-stack-hub-virtual-network"></a>Azure Stack hub sanal ağını yapılandırma

Azure Stack hub sanal ağındaki yerel ağ geçidinin, trafiği App Service Noktadan siteye adres aralığından yönlendirmek üzere yapılandırılması gerekir.

1. Azure Stack hub portalında **yerel ağ geçidi**' ne gidin. **Ayarlar** bölümünde **Yapılandırma**‘yı seçin.

    ![Azure Stack Merkezi yerel ağ geçidinde ağ geçidi yapılandırma seçeneği](media/solution-deployment-guide-hybrid/image14.png)

2. **Adres alanı**' nda, Azure 'daki sanal ağ geçidi için Noktadan siteye adres aralığını girin.

    ![Azure Stack hub yerel ağ geçidinde Noktadan siteye adres alanı](media/solution-deployment-guide-hybrid/image15.png)

3. Yapılandırmayı doğrulamak ve kaydetmek için **Kaydet** ' i seçin.

## <a name="configure-dns-for-cross-cloud-scaling"></a>DNS 'i platformlar arası ölçeklendirme için yapılandırma

Kullanıcılar, platformlar arası uygulamalar için düzgün şekilde yapılandırılarak, Web uygulamanızın Genel Azure ve Azure Stack hub örneklerine erişebilir. Bu öğreticinin DNS yapılandırması, Yük arttıkça veya azaldıkça Azure Traffic Manager trafiği yönlendirmeye de olanak tanır.

Bu öğretici, DNS 'yi yönetmek için Azure DNS kullanır, çünkü App Service etki alanları çalışmıyor.

### <a name="create-subdomains"></a>Alt etki alanları oluşturma

Traffic Manager DNS CNAMEs ' i kullandığından, trafiği uç noktalara doğru bir şekilde yönlendirmek için bir alt etki alanı gerekir. DNS kayıtları ve etki alanı eşlemesi hakkında daha fazla bilgi için bkz. [Traffic Manager etki alanlarını eşleme](/azure/app-service/web-sites-traffic-manager-custom-domain-name).

Azure uç noktası için kullanıcıların Web uygulamanıza erişmek için kullanabileceği bir alt etki alanı oluşturacaksınız. Bu öğretici için **app.Northwind.com**kullanabilir, ancak bu değeri kendi etki alanınızı temel alarak özelleştirmeniz gerekir.

Ayrıca, Azure Stack hub uç noktası için bir kayıt içeren bir alt etki alanı oluşturmanız gerekir. **Azurestack.Northwind.com**kullanabilirsiniz.

### <a name="configure-a-custom-domain-in-azure"></a>Azure 'da özel bir etki alanı yapılandırma

1. [CNAME 'i Azure App Service ile eşleyerek](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record) **app.Northwind.com** ana bilgisayar adını Azure Web uygulamasına ekleyin.

### <a name="configure-custom-domains-in-azure-stack-hub"></a>Azure Stack hub 'da özel etki alanlarını yapılandırma

1. [Bir kaydı Azure App Service ile eşleyerek](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record)Azure Stack Hub web uygulamasına **azurestack.Northwind.com** ana bilgisayar adını ekleyin. App Service uygulaması için internet yönlendirilebilir IP adresini kullanın.

2. [CNAME 'i Azure App Service ile eşleyerek](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)Azure Stack Hub web uygulamasına **app.Northwind.com** ana bilgisayar adını ekleyin. Önceki adımda yapılandırdığınız ana bilgisayar adını (1) CNAME için hedef olarak kullanın.

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a>Platformlar arası ölçeklendirme için SSL sertifikalarını yapılandırma

Web uygulamanız tarafından toplanan gizli verilerin, SQL veritabanında depolanan ve ' de yapılan aktarım sırasında güvenli olduğundan emin olmanız önemlidir.

Azure ve Azure Stack Hub web uygulamalarınızı tüm gelen trafik için SSL sertifikaları kullanacak şekilde yapılandıracaksınız.

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a>Azure ve Azure Stack hub 'a SSL ekleme

Azure 'a SSL eklemek için:

1. Aldığınız SSL sertifikasının oluşturduğunuz alt etki alanı için geçerli olduğundan emin olun. (Joker karakter sertifikaları kullanmak normaldir.)

2. Azure portal, **Web uygulamanızı hazırlama** ve [var olan özel bir SSL sertifikasını Azure 'a](/azure/app-service/app-service-web-tutorial-custom-ssl) **bağlama bölümündeki** yönergeleri izleyin Web Apps makalesine gidin. **SSL türü**olarak **SNı tabanlı SSL** ' yi seçin.

3. Tüm trafiği HTTPS bağlantı noktasına yönlendir. [Var olan bir özel SSL sertifikası 'nı Web Apps Azure 'A bağlama](/azure/app-service/app-service-web-tutorial-custom-ssl) makalesindeki **https 'yi zorla** bölümünde bulunan yönergeleri izleyin.

Azure Stack hub 'ına SSL eklemek için:

1. Azure Stack hub portalını kullanarak Azure için kullandığınız 1-3 adımları yineleyin.

## <a name="configure-and-deploy-the-web-app"></a>Web uygulamasını yapılandırma ve dağıtma

Uygulama kodunu, telemetri doğru Application Insights örneğine bildirmek ve Web uygulamalarını doğru bağlantı dizeleri ile yapılandırmak için yapılandıracaksınız. Application Insights hakkında daha fazla bilgi edinmek için bkz. [Application Insights nedir?](/azure/application-insights/app-insights-overview)

### <a name="add-application-insights"></a>Application Insights Ekle

1. Web uygulamanızı Microsoft Visual Studio açın.

2. Web trafiği arttıkça veya azaldıkça, Application Insights tarafından uyarı oluşturmak için kullanılan Telemetriyi iletmek üzere projenize [Application Insights ekleyin](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) .

### <a name="configure-dynamic-connection-strings"></a>Dinamik bağlantı dizelerini yapılandırma

Web uygulamasının her örneği SQL veritabanına bağlanmak için farklı bir yöntem kullanacaktır. Azure 'daki uygulama SQL Server VM özel IP adresini kullanır ve Azure Stack hub 'ındaki uygulama SQL Server VM genel IP adresini kullanır.

> [!Note]  
> Azure Stack hub ile tümleşik bir sistemde, genel IP adresi internet yönlendirilebilir olmamalıdır. Bir ASDK 'de genel IP adresi, ASDK dışında yönlendirilebilir değildir.

App Service ortam değişkenlerini, uygulamanın her örneğine farklı bir bağlantı dizesi geçirmek için kullanabilirsiniz.

1. Uygulamayı Visual Studio 'da açın.

2. Startup.cs açın ve aşağıdaki kod bloğunu bulun:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. Önceki kod bloğunu, dosyasında *appsettings.js* tanımlı bir bağlantı dizesi kullanan aşağıdaki kodla değiştirin:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a>App Service uygulama ayarlarını yapılandırma

1. Azure ve Azure Stack Hub için bağlantı dizeleri oluşturun. Dizeler, kullanılan IP adresleri dışında, aynı olmalıdır.

2. Azure ve Azure Stack hub 'da, ad içinde ön ek olarak kullanarak, Web uygulamasında uygun bağlantı dizesini [bir uygulama ayarı olarak](/azure/app-service/web-sites-configure) ekleyin `SQLCONNSTR\_` .

3. Web uygulaması ayarlarını **kaydedin** ve uygulamayı yeniden başlatın.

## <a name="enable-automatic-scaling-in-global-azure"></a>Küresel Azure 'da otomatik ölçeklendirmeyi etkinleştirme

Web uygulamanızı bir App Service ortamında oluşturduğunuzda, tek bir örnekle başlar. Uygulamanız için daha fazla işlem kaynağı sağlamak üzere örnekleri eklemek için otomatik olarak ölçeği değiştirebilirsiniz. Benzer şekilde, uygulamanızda otomatik olarak ölçeklendirme yapabilir ve uygulamanızın ihtiyaç duyacağı örnek sayısını azaltabilirsiniz.

> [!Note]  
> Ölçek Genişletme ve ölçekleme yapılandırmak için bir App Service planına sahip olmanız gerekir. Bir planınız yoksa, sonraki adımları başlatmadan önce bir tane oluşturun.

### <a name="enable-automatic-scale-out"></a>Otomatik ölçeklendirmeyi etkinleştir

1. Azure portal, genişletmek istediğiniz siteler için App Service planı bulun ve ardından **genişleme (App Service planı)** öğesini seçin.

    ![Ölçeği genişletme Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. **Otomatik ölçeklendirmeyi etkinleştir**' i seçin.

    ![Azure App Service otomatik ölçeklendirmeyi etkinleştir](media/solution-deployment-guide-hybrid/image17.png)

3. **Otomatik ölçeklendirme ayarı adı**için bir ad girin. **Varsayılan** otomatik ölçeklendirme kuralı için, **ölçüm temelinde ölçek**' i seçin. **Örnek sınırlarını** **En düşük: 1**, **en fazla: 10**ve **Varsayılan: 1**olarak ayarlayın.

    ![Azure App Service otomatik ölçeklendirmeyi yapılandırma](media/solution-deployment-guide-hybrid/image18.png)

4. **+ Kural Ekle**' yi seçin.

5. **Ölçüm kaynağı**' nda **geçerli kaynak**' ı seçin. Kural için aşağıdaki ölçütleri ve eylemleri kullanın.

#### <a name="criteria"></a>Ölçütler

1. Zaman toplama altında **Ortalama** **' ı** seçin.

2. **Ölçüm adı**altında **CPU yüzdesi**' ni seçin.

3. **İşleç**altında, **büyüktür**' i seçin.

   - **Eşiği** **50**olarak ayarlayın.
   - **Süreyi** **10**olarak ayarlayın.

#### <a name="action"></a>Eylem

1. **İşlem**altında, **sayıyı göre artır**' ı seçin.

2. **Örnek sayısını** **2**olarak ayarlayın.

3. **Cool tuşuna** **5**olarak ayarlayın.

4. **Ekle**’yi seçin.

5. **+ Kural Ekle**' yi seçin.

6. **Ölçüm kaynağı**' nda **geçerli kaynak** ' ı seçin.

   > [!Note]  
   > Geçerli kaynak App Service planınızın adını/GUID 'sini, **kaynak türü** ve **kaynak** açılan listelerini de kullanılamaz hale alacak.

### <a name="enable-automatic-scale-in"></a>Otomatik ölçeklendirmeyi etkinleştir

Trafik azaldıkça Azure Web uygulaması, maliyetleri azaltmak için etkin örnek sayısını otomatik olarak azaltabilir. Bu eylem, genişleme sıkından daha az agresif ve uygulama kullanıcıları üzerindeki etkiyi en aza indirir.

1. **Varsayılan** genişleme durumuna gidin, ardından **+ Kural Ekle**' yi seçin. Kural için aşağıdaki ölçütleri ve eylemleri kullanın.

#### <a name="criteria"></a>Ölçütler

1. Zaman toplama altında **Ortalama** **' ı** seçin.

2. **Ölçüm adı**altında **CPU yüzdesi**' ni seçin.

3. **İşleç** **altında küçüktür ' i seçin.**

   - **Eşiği** **30**olarak ayarlayın.
   - **Süreyi** **10**olarak ayarlayın.

#### <a name="action"></a>Eylem

1. **İşlem**altında **sayıyı göre azalt**' ı seçin.

   - **Örnek sayısını** **1**olarak ayarlayın.
   - **Cool tuşuna** **5**olarak ayarlayın.

2. **Ekle**’yi seçin.

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a>Traffic Manager profili oluşturma ve platformlar arası ölçeklendirmeyi yapılandırma

Azure portal kullanarak bir Traffic Manager profili oluşturun ve sonra platformlar arası ölçeklendirmeyi etkinleştirmek için uç noktaları yapılandırın.

### <a name="create-traffic-manager-profile"></a>Traffic Manager profili oluşturma

1. **Kaynak oluştur**’u seçin.
2. **Ağ**'ı seçin.
3. **Traffic Manager profil** ' i seçin ve aşağıdaki ayarları yapılandırın:

   - **Ad**alanına profiliniz için bir ad girin. Bu ad trafficmanager.net bölgesinde benzersiz **olmalıdır** ve yenı bir DNS adı oluşturmak için kullanılır (örneğin, northwindstore.trafficmanager.net).
   - **Yönlendirme yöntemi**için **ağırlıklı**' ı seçin.
   - **Abonelik**için, bu profili oluşturmak istediğiniz aboneliği seçin.
   - **Kaynak grubu**' nda, bu profil için yeni bir kaynak grubu oluşturun.
   - **Kaynak grubu konumu** alanında kaynak grubunun konumunu seçin. Bu ayar, kaynak grubunun konumunu ifade eder ve genel olarak dağıtılan Traffic Manager profilini etkilemez.

4. **Oluştur**’u seçin.

    ![Traffic Manager profili oluşturma](media/solution-deployment-guide-hybrid/image19.png)

   Traffic Manager profilinizin genel dağıtımı tamamlandığında, altında oluşturduğunuz kaynak grubu için kaynak listesinde gösterilir.

### <a name="add-traffic-manager-endpoints"></a>Traffic Manager uç noktalarını ekleme

1. Oluşturduğunuz Traffic Manager profilini arayın. Profil için kaynak grubuna gezindiyseniz, profili seçin.

2. **Traffic Manager profilinde**, **Ayarlar**altında **uç noktalar**' ı seçin.

3. **Ekle**’yi seçin.

4. **Uç nokta Ekle**' de Azure Stack Hub için aşağıdaki ayarları kullanın:

   - **Tür**için **dış uç nokta**' ı seçin.
   - Uç nokta için bir **ad** girin.
   - **Tam etki alanı adı (FQDN) veya IP**için, Azure Stack Hub web uygulamanızın dış URL 'sini girin.
   - **Ağırlık**için varsayılan değer olan **1**' i tutun. Bu ağırlık, sağlıklı ise bu uç noktaya giden tüm trafik ile sonuçlanır.
   - **Eklemeyi devre dışı** bırak işareti kaldırılmış olarak bırakın.

5. Azure Stack hub uç noktasını kaydetmek için **Tamam ' ı** seçin.

Sonraki Azure uç noktasını yapılandıracaksınız.

1. **Traffic Manager profilinde** **uç noktalar**' ı seçin.
2. **+Ekle**’yi seçin.
3. **Uç nokta Ekle**sayfasında Azure için aşağıdaki ayarları kullanın:

   - **Tür**için **Azure uç noktası**' nı seçin.
   - Uç nokta için bir **ad** girin.
   - **Hedef kaynak türü**için **App Service**seçin.
   - **Hedef kaynak**için, aynı abonelikte Web Apps listesini görmek üzere **bir App Service seçin** öğesini seçin.
   - **Kaynak** bölümünde ilk uç nokta olarak kullanmak istediğiniz uygulama hizmetini seçin.
   - **Ağırlık**için **2**' yi seçin. Bu ayar, birincil uç noktanın sağlıksız olması durumunda ya da tetiklendiğinde trafiği yeniden yönlendiren bir kuralınız/uyarıınız varsa, bu uç noktaya giden tüm trafiğe neden olur.
   - **Eklemeyi devre dışı** bırak işareti kaldırılmış olarak bırakın.

4. Azure uç noktasını kaydetmek için **Tamam ' ı** seçin.

Her iki uç nokta yapılandırıldıktan sonra, **uç noktalar**' ı seçerken **Traffic Manager profilde** listelenir. Aşağıdaki ekran yakalamadaki örnek, her biri için durum ve yapılandırma bilgileriyle iki uç nokta gösterir.

![Traffic Manager profilindeki uç noktalar](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting-in-azure"></a>Azure 'da Application Insights izlemeyi ve uyarı ayarlamayı ayarlama

Azure Application Insights, uygulamanızı izlemenizi ve yapılandırdığınız koşullara göre uyarı göndermenizi sağlar. Bazı örnekler şunlardır: uygulama kullanılamıyor, hatalarla karşılaşıyor veya performans sorunlarını gösteriyor.

Uyarı oluşturmak için Azure Application Insights ölçümlerini kullanacaksınız. Bu uyarılar tetiklenmesi durumunda, Web uygulamanızın örneği ölçeği genişletmek için otomatik olarak Azure Stack hub 'dan Azure 'a geçiş yapar ve sonra ölçeklendirmek üzere Azure Stack hub 'a geri yüklenir.

### <a name="create-an-alert-from-metrics"></a>Ölçülerden uyarı oluşturma

Azure portal, Bu öğreticinin kaynak grubuna gidin ve **Application Insights**açmak için Application Insights örneğini seçin.

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

Bu görünümü, bir genişleme uyarısı ve bir ölçek genişletme uyarısı oluşturmak için kullanacaksınız.

### <a name="create-the-scale-out-alert"></a>Genişleme uyarısı oluşturma

1. **Yapılandır**altında **Uyarılar (klasik)** öğesini seçin.
2. **Ölçüm uyarısı Ekle (klasik)** seçeneğini belirleyin.
3. **Kural Ekle**' de, aşağıdaki ayarları yapılandırın:

   - **Ad**Için **Azure buluta veri bloğu**girin.
   - **Açıklama** isteğe bağlıdır.
   - **Kaynak**  >  **uyarısı**' nın altında, **ölçümler**' i seçin.
   - **Ölçüt**altında, aboneliğinizi, Traffic Manager profilinizin kaynak grubunu ve kaynak için Traffic Manager profilinin adını seçin.

4. **Ölçüm**Için **istek hızı**' nı seçin.
5. **Koşul**Için **şundan büyüktür**' i seçin.
6. **Eşik**için **2**girin.
7. **Süre**için **son 5 dakikayı**seçin.
8. **Bildirim**altında:
   - **E-posta sahipleri, katkıda bulunanlar ve okuyucular**için onay kutusunu işaretleyin.
   - **Ek yönetici e-postaları**için e-posta adresinizi girin.

9. Menü çubuğunda **Kaydet**' i seçin.

### <a name="create-the-scale-in-alert"></a>Ölçek Genişletme uyarısını oluşturma

1. **Yapılandır**altında **Uyarılar (klasik)** öğesini seçin.
2. **Ölçüm uyarısı Ekle (klasik)** seçeneğini belirleyin.
3. **Kural Ekle**' de, aşağıdaki ayarları yapılandırın:

   - **Ad**için **ölçeği Azure Stack hub 'a geri**girin.
   - **Açıklama** isteğe bağlıdır.
   - **Kaynak**  >  **uyarısı**' nın altında, **ölçümler**' i seçin.
   - **Ölçüt**altında, aboneliğinizi, Traffic Manager profilinizin kaynak grubunu ve kaynak için Traffic Manager profilinin adını seçin.

4. **Ölçüm**Için **istek hızı**' nı seçin.
5. **Koşul** **için küçüktür ' i seçin.**
6. **Eşik**için **2**girin.
7. **Süre**için **son 5 dakikayı**seçin.
8. **Bildirim**altında:
   - **E-posta sahipleri, katkıda bulunanlar ve okuyucular**için onay kutusunu işaretleyin.
   - **Ek yönetici e-postaları**için e-posta adresinizi girin.

9. Menü çubuğunda **Kaydet**' i seçin.

Aşağıdaki ekran görüntüsünde, genişleme ve ölçek genişletme uyarıları gösterilmektedir.

   ![Application Insights uyarılar (klasik)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a>Azure ile Azure Stack hub arasında trafiği yeniden yönlendirme

Azure ile Azure Stack hub arasında Web uygulaması trafiğiniz için el ile veya otomatik geçişi yapılandırabilirsiniz.

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a>Azure ile Azure Stack hub arasında el ile geçiş yapılandırma

Web siteniz yapılandırdığınız eşiklere ulaştığında bir uyarı alırsınız. Trafiği Azure 'a el ile yönlendirmek için aşağıdaki adımları kullanın.

1. Azure portal Traffic Manager profilinizi seçin.

    ![Azure portal uç noktaları Traffic Manager](media/solution-deployment-guide-hybrid/image20.png)

2. **Uç noktaları**seçin.
3. **Azure uç noktasını**seçin.
4. **Durum**altında, **etkin**' i seçin ve ardından **Kaydet**' i seçin.

    ![Azure portal 'de Azure uç noktasını etkinleştirme](media/solution-deployment-guide-hybrid/image23.png)

5. Traffic Manager profilinin **uç noktalarında** **dış uç nokta**' ı seçin.
6. **Durum**altında **devre dışı**' yı seçin ve ardından **Kaydet**' i seçin.

    ![Azure portal Azure Stack hub uç noktasını devre dışı bırak](media/solution-deployment-guide-hybrid/image24.png)

Uç noktalar yapılandırıldıktan sonra, uygulama trafiği Azure Stack Hub web uygulaması yerine Azure genişleme Web uygulamanıza gider.

 ![Azure Web uygulaması trafiği 'nde uç noktalar değişti](media/solution-deployment-guide-hybrid/image25.png)

Akışı Azure Stack hub 'a geri çevirmek için önceki adımları kullanın:

- Azure Stack hub uç noktasını etkinleştirin.
- Azure uç noktasını devre dışı bırakın.

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a>Azure ile Azure Stack hub arasında otomatik geçiş yapılandırma

Uygulamanız Azure Işlevleri tarafından sunulan [sunucusuz](https://azure.microsoft.com/overview/serverless-computing/) bir ortamda çalıştırılıyorsa Application Insights izlemeyi da kullanabilirsiniz.

Bu senaryoda, bir işlev uygulaması çağıran bir Web kancasını kullanmak için Application Insights yapılandırabilirsiniz. Bu uygulama, bir uyarıya yanıt olarak bir uç noktayı otomatik olarak sağlar veya devre dışı bırakır.

Otomatik trafik geçişini yapılandırmak için aşağıdaki adımları kılavuz olarak kullanın.

1. Bir Azure Işlev uygulaması oluşturun.
2. HTTP ile tetiklenen bir işlev oluşturun.
3. Kaynak Yöneticisi, Web Apps ve Traffic Manager için Azure SDK 'larını içeri aktarın.
4. Kodu şu şekilde geliştirin:

   - Azure aboneliğinizde kimlik doğrulaması yapın.
   - Traffic Manager uç noktalarına trafiği Azure 'a veya Azure Stack hub 'a yönlendirmek için geçiş yapan bir parametre kullanın.

5. Kodunuzu kaydedin ve işlev uygulamasının URL 'sini, Application Insights uyarı kuralı ayarlarının **Web kancası** bölümüne uygun parametrelerle ekleyin.
6. Application Insights bir uyarı tetiklendiğinde trafik otomatik olarak yönlendirilir.

## <a name="next-steps"></a>Sonraki adımlar

- Azure bulut desenleri hakkında daha fazla bilgi edinmek için bkz. [bulut tasarım desenleri](/azure/architecture/patterns).
