
# **Azure Service Bus ve Azure Functions**

Bu makale, Azure Service Bus ve Azure Functions hizmetlerini detaylı şekilde inceleyen, gerçek dünya senaryolarıyla desteklenen kapsamlı bir teknik rehberdir. Olay tabanlı dağıtık sistemler geliştirmek isteyen yazılım geliştiriciler ve sistem mimarları için profesyonel düzeyde bilgi sağlar.

---

## **📌 İçindekiler**
1. [Giriş](#giriş)
2. [Azure Service Bus Nedir?](#azure-service-bus-nedir)
    - [2.1. Temel Kavramlar](#21-temel-kavramlar)
    - [2.2. Azure Service Bus Bileşenleri](#22-azure-service-bus-bileşenleri)
    - [2.3. Queue ve Topic Karşılaştırması](#23-queue-ve-topic-karşılaştırması)
    - [2.4. Kod Örneği: Service Bus Mesaj Gönderme](#24-kod-örneği-service-bus-mesaj-gönderme)
3. [Azure Functions Nedir?](#azure-functions-nedir)
    - [3.1. Azure Functions Türleri](#31-azure-functions-türleri)
    - [3.2. Azure Functions Mimarisi](#32-azure-functions-mimarisi)
    - [3.3. Kod Örneği: HTTP Trigger Kullanımı](#33-kod-örneği-http-trigger-kullanımı)
4. [Azure Functions ve Service Bus Entegrasyonu](#azure-functions-ve-service-bus-entegrasyonu)
    - [4.1. Service Bus Trigger ile Mesaj Alma](#41-service-bus-trigger-ile-mesaj-alma)
5. [Best Practices](#best-practices)
6. [Sonuç ve Kaynaklar](#sonuç-ve-kaynaklar)

---

## **1. Giriş**

Modern dağıtık sistemlerde, yüksek güvenilirlik ve ölçeklenebilirlik sağlamak için olay tabanlı mimariler kritik öneme sahiptir. Azure Service Bus ve Azure Functions, kurumsal uygulamalarda olayları işleme ve mesajları teslim etme sürecini kolaylaştırır.

Azure Service Bus güvenli ve sıralı mesaj teslimatı sunarken, Azure Functions küçük işlevleri sunucusuz olarak çalıştırarak hızlı ve esnek işleme imkanı sağlar.

---

## **2. Azure Service Bus Nedir?**

Azure Service Bus, Microsoft Azure’un kurumsal mesajlaşma platformudur. Yüksek güvenilirlik, mesaj sıralaması ve asenkron mesaj işleme gibi özellikler sunarak sistemler arası veri akışını güvenli hale getirir.

### **2.1. Temel Kavramlar**

- **Mesaj (Message):** Gönderilen veri birimi. JSON, XML veya düz metin olabilir.
- **Queue (Kuyruk):** Mesajların bir sıralama içinde beklediği yerdir. İlk giren ilk çıkar (FIFO) prensibi uygulanır.
- **Topic & Subscription (Başlık ve Abonelik):** Yayın-abone modelini destekler. Bir mesaj birden fazla aboneye ulaşabilir.
- **Dead-Letter Queue (DLQ):** Teslim edilemeyen mesajlar için ayrılmış özel kuyruktur.

### **2.2. Azure Service Bus Bileşenleri**

| **Bileşen**         | **Açıklama**                      |
|--------------------|-----------------------------------|
| Namespace          | Service Bus kaynaklarını gruplandırır. |
| Queue (Kuyruk)     | Tek alıcı için sıralı mesajlar sunar. |
| Topic & Subscription | Çoklu alıcılar için mesajları dağıtır. |
| Message (Mesaj)    | Service Bus’a gönderilen veri paketidir. |
| Dead-Letter Queue  | İşlenemeyen mesajlar için özel kuyruktur. |

### **2.3. Queue ve Topic Karşılaştırması**

| **Queue (Kuyruk)**             | **Topic & Subscription (Başlık-Abonelik)** |
|--------------------------------|-----------------------------------------------|
| Tek bir alıcıya mesaj gönderir.| Birden fazla alıcıya mesaj gönderir.          |
| FIFO (First In, First Out) kuralı | Yayın-Abone modelini destekler.            |
| Mesajlar işlenmeden önce bekler.| Abonelik tabanlı işleme yapılır.             |

### **2.4. Kod Örneği: Service Bus Mesaj Gönderme**

```csharp
using Azure.Messaging.ServiceBus;

var connectionString = "YourServiceBusConnectionString";
var queueName = "orders";

await using var client = new ServiceBusClient(connectionString);
var sender = client.CreateSender(queueName);

var message = new ServiceBusMessage("Sipariş oluşturuldu!");
await sender.SendMessageAsync(message);

Console.WriteLine("Mesaj gönderildi.");
```

---

## **3. Azure Functions Nedir?**

Azure Functions, sunucusuz (serverless) işlevleri bulutta çalıştırarak ölçeklenebilir, olay tabanlı işleme sağlar.

### **3.1. Azure Functions Türleri**

- **HTTP Trigger:** Web API'leri ve istekleri işler.
- **Service Bus Trigger:** Kuyruk ve Topic mesajlarını işler.
- **Timer Trigger:** Zamanlanmış görevler.
- **Blob Trigger:** Dosya işleme işlevleri.

### **3.2. Azure Functions Mimarisi**

| **Bileşen**      | **Açıklama**                   |
|------------------|-------------------------------|
| Function App     | Tüm işlevlerin barındığı yapı.|
| Trigger          | Olayları tetikleyen unsur.   |
| Bindings         | Veri giriş-çıkış işlemleri.   |
| Durable Functions | Uzun süreli iş süreçlerini destekler. |

### **3.3. Kod Örneği: HTTP Trigger Kullanımı**

```csharp
[FunctionName("HttpExample")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
    ILogger log)
{
    log.LogInformation("HTTP Trigger function çalıştırıldı.");
    string name = req.Query["name"];
    return name != null
        ? new OkObjectResult($"Merhaba, {name}")
        : new BadRequestObjectResult("Lütfen bir isim belirtin.");
}
```

---

## **4. Azure Functions ve Service Bus Entegrasyonu**

```csharp
[FunctionName("OrderProcessor")]
public static async Task Run(
    [ServiceBusTrigger("orders", Connection = "AzureServiceBusConnection")] string message,
    ILogger log)
{
    log.LogInformation($"Sipariş alındı: {message}");
}
```

---

## **5. Best Practices**  

Azure Service Bus ve Azure Functions projeleri için önerilen **en iyi uygulamalar** şunlardır:  

- **Bağlantı Yönetimi:** Azure Key Vault ile bağlantı dizelerini güvenli şekilde saklayın.  
- **Hata Yönetimi:** Dead Letter Queue kullanarak mesaj hatalarını izleyin ve gerektiğinde manuel müdahale yapın.  
- **Loglama ve İzleme:** Azure Monitor ve Application Insights kullanarak iş süreçlerini ve uygulama performansını izleyin.  
- **Ölçeklenebilirlik:** Service Bus **Auto-Scaling (Otomatik Ölçekleme)** özelliklerini etkinleştirin.  
- **Güvenlik:** Kimlik doğrulama ve yetkilendirme için Azure AD ve Managed Identity kullanın.  
- **Retry Policy:** Yeniden deneme politikaları oluşturarak bağlantı hatalarından kaynaklanan veri kaybını önleyin.

---

## **6. Sonuç ve Kaynaklar**

Azure Service Bus ve Azure Functions kullanarak olay tabanlı bir sistem nasıl oluşturulacağını öğrendik. Daha fazla bilgi için resmi Azure belgelerine başvurabilirsiniz.

---
