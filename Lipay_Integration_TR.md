# LiPayKripto API Entegrasyonu

Bu dokümantasyon, LiPayKripto API'sini kullanarak ödeme (payment) ve çekim (withdraw) işlemlerini entegre etmeniz için gereken adımları detaylı olarak anlatmaktadır.

## Genel Bakış

LiPayKripto external-payment-request API'si, kullanıcılarınızın kripto para (TRX, USDT, ETH) ile ödeme yapmasını sağlayan güvenli bir ödeme geçidi sunar. 

**ÖNEMLİ:** 
- Her yeni ödeme isteği için yeni bir JWT token alınmalıdır. Tokenler sınırlı bir süre geçerlidir.
- Tüm API istekleri **SADECE POST** metodu ile yapılmalıdır. GET, PUT veya diğer HTTP metotları desteklenmez.

## API Endpoint'leri

1. **Token Alma:** `https://lipaykripto.com/api/auth/token`
2. **Ödeme Talebi:** `https://lipaykripto.com/api/external-payment-request`
3. **Çekim Talebi:** `https://lipaykripto.com/api/withdraw`

## 1. Token Alma Süreci

### İstek (Request)

**HTTP Metodu:** **POST** (❗ Yalnızca POST metodu desteklenir)

**Headers:**
```
Content-Type: application/json
Accept: application/json
```

**Body:**
```json
{
  "clientId": "YOUR_CLIENT_ID",
  "clientSecret": "YOUR_CLIENT_SECRET"
}
```

### Yanıt (Response)

**Başarılı yanıt:**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 3600  // Token geçerlilik süresi (saniye)
}
```

**Hata yanıtı:**

```json
{
  "error": "Geçersiz kimlik bilgileri"
}
```

## 2. Ödeme Talebi Oluşturma

### İstek (Request)

**HTTP Metodu:** **POST** (❗ Yalnızca POST metodu desteklenir)

**Headers:**
```
Content-Type: application/json
Accept: application/json
Authorization: Bearer [ALINAN_TOKEN]
```

**Body:**
```json
{
  "tryAmount": 100.00,                  // TL cinsinden tutar
  "paymentId": "ORDER_123",             // Benzersiz sipariş numarası
  "webhookUrl": "https://example.com/webhook"  // Webhook URL (zorunlu)
}
```

### Yanıt (Response)

**Başarılı yanıt:**

```json
{
  "success": true,
  "paymentUrl": "https://lipaykripto.com/payment/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "paymentId": "ORDER_123",
  "clientId": "YOUR_CLIENT_ID",
  "paymentRequestId": 505,
  "status": "pending_coin_selection"
}
```

**Hata yanıtı:**

```json
{
  "success": false,
  "error": "Geçersiz tutar veya paymentId"
}
```

## 3. Kullanıcı Deneyimi

1. Kullanıcı, uygulamanızda ödeme işlemini başlatır
2. Sisteminiz LiPayKripto'dan **her ödeme işlemi için yeni** bir token alır
3. Token ile bir ödeme talebi oluşturur
4. Kullanıcıyı `paymentUrl` adresine yönlendirir
5. Kullanıcı ödeme sayfasında kripto para cinsini seçer ve ödeme yapar
6. Ödeme tamamlandığında, LiPayKripto webhook URL'nize bir bildirim gönderir
7. Kullanıcı, ödeme sonrası sayfanıza yönlendirilir

## 4. Webhook Bildirimleri

Ödeme durumu değiştiğinde, webhook URL'nize aşağıdaki formatta bir **POST** isteği gönderilir:

```json
{
  "try_amount": 100.00,
  "client_id": "YOUR_CLIENT_ID",
  "payment_id": "ORDER_123",
  "status": "confirmed"  // veya "failed"
}
```

Webhook bildirimini aldığınızda, HTTP 200 yanıtı döndürmelisiniz.

## 5. Güvenlik Tavsiyeleri

1. Client Secret'ı hiçbir zaman istemci tarafında (front-end) kullanmayın
2. Her ödeme için benzersiz bir `paymentId` kullanın
3. Webhook URL'niz HTTPS olmalı ve dışarıdan erişilebilir olmalıdır
4. Aynı ödemenin tekrar işlenmesini önlemek için payment_id'yi kontrol edin
5. Webhook bildirimlerinin bütünlüğünü doğrulamak için, bildirim içeriğindeki client_id'nin sizin client_id'niz olduğunu kontrol edin

## 6. Test İşlemleri

Entegrasyonunuzu test etmek için, aşağıdaki adımları izleyebilirsiniz:

1. Test hesabı bilgilerinizi [support@lipaykripto.com](mailto:support@lipaykripto.com) adresinden talep edin
2. Test ortamında küçük tutarlarla ödeme taleplerini test edin
3. Webhook URL'nizin bildirimleri düzgün aldığını doğrulayın
4. Ödeme başarılı, başarısız ve zaman aşımı senaryolarını test edin

## 7. Hata Kodları ve Açıklamaları

| HTTP Kodu | Hata Mesajı                      | Açıklama                                        |
|-----------|----------------------------------|------------------------------------------------|
| 400       | Geçersiz tutar                   | TL tutarı sıfırdan büyük olmalıdır              |
| 400       | Geçersiz paymentId               | paymentId benzersiz ve boş olmamalıdır          |
| 400       | Webhook URL gerekli              | Webhook URL zorunlu bir alandır                 |
| 401       | Geçersiz token                   | Token geçersiz veya süresi dolmuş               |
| 401       | Geçersiz kimlik bilgileri        | Client ID veya Client Secret hatalı             |
| 429       | Çok fazla istek                  | İstek limiti aşıldı, daha sonra tekrar deneyin  |
| 500       | Sunucu hatası                    | Sunucu hatası, daha sonra tekrar deneyin        |

## 8. Çekim Talebi Oluşturma

Kullanıcılarınıza kripto para (TRX, USDT, ETH) çekim işlemi yapabilmek için bu endpoint'i kullanabilirsiniz.

### İstek (Request)

**HTTP Metodu:** **POST** (❗ Yalnızca POST metodu desteklenir)

**Headers:**
```
Content-Type: application/json
Accept: application/json
```

**Body:**
```json
{
  "try_amount": 100.00,                  // TL cinsinden çekilecek tutar
  "clientId": "YOUR_CLIENT_ID",          // Client ID
  "requestId": "WITHDRAW_123",           // Benzersiz çekim numarası
  "wallet_address": "TXBCD12345...",     // Kullanıcının çekim yapacağı cüzdan adresi
  "coin_type": "TRX",                    // Çekim yapılacak kripto para birimi (TRX, USDT veya ETH)
  "signature": "f58a92bcde...",          // HMAC imzası
  "webhook_url": "https://example.com/webhook",  // Webhook URL (zorunlu)
  "created_at": "2025-04-20T14:30:00Z"   // İşlem oluşturma tarihi (ISO 8601 formatında)
}
```

**HMAC İmzası Oluşturma:**

İmza, aşağıdaki alanların belirtilen sırada birleştirilmesiyle oluşturulur:
```
try_amount + clientId + requestId + wallet_address + coin_type + webhook_url + created_at
```

Örneğin:
```
100.00test12345WITHDRAW_123TXBCD12345...TRXhttps://example.com/webhook2025-04-20T14:30:00Z
```

Bu birleştirilmiş string, HMAC-SHA256 algoritması ve size verilen HMAC Secret ile imzalanır.

### Yanıt (Response)

**Başarılı yanıt:**

```json
{
  "success": true,
  "data": {
    "withdrawId": 123,
    "status": "pending"
  }
}
```

**Hata yanıtı:**

```json
{
  "success": false,
  "error": "Geçersiz imza",
  "details": [...]
}
```

### Çekim Webhook Bildirimleri

Çekim durumu değiştiğinde, webhook URL'nize aşağıdaki formatta bir **POST** isteği gönderilir:

```json
{
  "success": true,
  "clientId": "YOUR_CLIENT_ID",
  "status": "confirmed",  // veya "failed", "manual_process"
  "tryAmount": 100.00,
  "requestId": "WITHDRAW_123"
}
```

## 9. İletişim ve Destek

Entegrasyon sürecinde herhangi bir sorunla karşılaşırsanız, [support@lipaykripto.com](mailto:support@lipaykripto.com) adresinden bizimle iletişime geçebilirsiniz.