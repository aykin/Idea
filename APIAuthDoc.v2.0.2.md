# IdeaShop API 
## Yetkilendirme Dökümanı
*Versiyon 2.0.1*

### Tanım
   IdeaShop API, authentication için OAuth2 standardını kullanır. Oauth2; kullanıcıların kullandığı uygulamalardan kendi özel hesaplarına (Facebook,github vs) sınırlı olarak ulaşmalarına yardımcı olan bir yetkilendirme sistemidir. Burada asıl amaç, kullanıcının kaynak dosyasından bilgilerini çekmek için kullanıcı adı ve parolasını almadan, gerekli parametreleri göndererek bilgilerine ulaşmayı sağlamaktır. 


### Gerekli Bilgiler
Uygulamanızın yetkilendirilmesi üç bilgiye ihtiyacınız vardır:

  * Client ID 
    - Uygulamanız için atanan rastgele yaratılmış bir string. Bu bilgi açıktır.
    - *Örn:* `7_7d67dc7597f034d63775c1d9ae5d9ac7f5750197f`
  * Client Secret
  	- Uygulamanız için özel yaratılmış bir string. **Bu gizli tutulmalıdır**
    - *Örn:* `1sowg0oogc4wg4w4o4gh4va57gggwskkgo08m44ksog8kmu88o`
  * Redirect URI
    - Kullanıcının uygulamanıza yetki verdikten sonra geri dönmesi gereken adres.
    - *Örn:* `http://client.example-app.com/auth/`

*Not: Bu üç bilginin de önceden sistemimizde kayıtlı olması gereklidir*

### Yetkilendirme İşlemleri
IdeaShop API yetkilendirmesi için iki ayrı yetkilendirme yöntemi bulunmaktadır, **authorization_code** ve **password**:
 - `authorization_code` yönteminda kullanıcı özel bir sayfaya yönlendirilir ve uygulamaya izin vermesi istenir. Kullanıcı adı ve parola gerekmez
 - `password` yönteminda kullanıcı adı ve parolası gereklidir. Uygulama, kullanıcı adı ve parolasını doğrudan ileterek yetki alabilir.

**authorization_code** yöntemi için işlem dört aşamada gerçekleştirilir. **password** yöntemi için sadece dördüncü aşama yeterlidir.

#### 1. Yetki sayfasına geliş
Uygulamanıza yetki vermesini istediğiniz IdeaShop kullanıcısını, uygulamanızı tanımlayan bilgiler ile beraber yetki sayfasına yönlendirmeniz gerekmektedir.
 
```http
GET http://<site_adresi>/admin/user/auth/ HTTP/1.1
```

Parametre | Açıklama
--------- | --------
client_id | Yukarıda tanımlanan *Client ID* değeri
response_type | Dogrulama için kullanılacak yöntem. Geçerli değerler: (authorization_code)
state | İstek için rastgele yaratılmış bir kod. (Üçüncü adımda kontrol edilmelidir)
redirect_uri | Yukarıda tanımlanan *Redirect URI* değeri. Onaydan sonra kullanıcının bu adrese yönlendirilecektir
  
  
*Örnek Sorgu:*
```http
GET http://www.ideashopgiyim.com/admin/user/auth/?client_id=7_7d67dc7597f034d63775c1d9ae5d9ac7f5750197f&response_type=authorization_code&state=2b33fdd45jbevd6nam&redirect_uri=http%3A%2F%2Fclient.example-app.com%2Fauth%2F HTTP/1.1
```

#### 2. Yetki izni sayfası
Kullanıcı, IdeaShop admin panelindeki yetki izin ekranına gelir. Burada uygulamanızın adı ve izinleri görülür. Kullanıcı isterse uygulamaya yetki verebilir veya reddedebilir.

#### 3. Uygulama sayfasına geri geliş
Bu aşamada, kullanıcı daha önce verdiğiniz `redirect_uri` adresine yönlendirilir. Duruma göre `GET` parametrelerinde aşağıdaki değerler bulunacaktır. Uygulamanız, bu değerleri dikkate alarak token almak, uyarı vermek vb. işlemleri gerçekleştirmelidir.

```http
GET http://<redirect_uri'de_verdiğiniz_adres>  HTTP/1.1
```
Parametre | Açıklama
--------- | --------
error | Hata kodu
error_description | Hata açıklaması
state | Birinci adımda verilen kod. (*Man In The Middle* tipi saldırılardan korunmak için bu değerin doğruluğunu kontrol etmeniz gereklidir)
code | Dördüncü adımda token almak için kullanılacak kimlik doğrulama kodu. **Geçerlilik süresi 30 saniyedir**

*Örnek Sorgu (Başarılı):*
```http
GET http://client.example-app.com/auth/?state=2b33fdd45jbevd6nam&code=Q0ODMI2OGYjMjBkN2mJmYzNkOTE4gzMGRhZDZTcykyOGQ3M2M2YTU2ZGVlMzE2MzB2MYkc5NWzE0ZWNiYjI2MA HTTP/1.1
```

`error` ve `error_description` değerleri sadece hata durumunda gönderilir. Gönderilebilecek hatalar aşağıdaki gibidir

error | error_description | Açıklama
------| ----------------- | --------
invalid_grant | | Kullanılan yetkilendirme yöntemi geçersiz
access_denied | The user denied access to your application | Kullanıcı uygulamanıza yetki vermemeyi seçtiğinde gönderilir


*Örnek Sorgu (Başarısız):*
```http
GET http://client.example-app.com/auth/?error=access_denied&error_description=The%20user%20denied%20access%20to%20your%20application&state=2b33fdd45jbevd6nam HTTP/1.1
```

#### 4. Token isteği
Bu adımda uygulama, kullanıcıdan bağımsız olarak sunucuya doğrudan istek göndererek "*Access Token*" ve "*Refresh Token*" bilgilerini alır. 

```http
POST http://<site-adresi>/oauth/v2/token HTTP/1.1
```
Parametre | Açıklama | authorization_code yöntemi | password yöntemi 
--------- | -------- | :-----: | :----:
grant_type | Dogrulama için kullanılmakta olan yöntem. Geçerli değerler: (authorization_code, password) | X | X
client_id | Yukarıda tanımlanan *Client ID* değeri | X | X
client_secret | Yukarıda tanımlanan *Client Secret* değeri | X | X
code | Üçüncü adımda elde edilen kimlik doğrulama kodu | X | 
redirect_uri | Yukarıda tanımlanan *Redirect URI* değeri | X | 
username | Kullanıcı adı |  | X
password | Kullanıcı parolası |  | X


*Örnek Sorgu (`authorization_code` yöntemi için):*
```http
POST http://www.ideashopgiyim.com/oauth/v2/token HTTP/1.1

:
grant_type=authorization_code&
client_id=7_7d67dc7597f034d63775c1d9ae5d9ac7f5750197f&
client_secret=1sowg0oogc4wg4w4o4gh4va57gggwskkgo08m44ksog8kmu88o&
code=Q0ODMI2OGYjMjBkN2mJmYzNkOTE4gzMGRhZDZTcykyOGQ3M2M2YTU2ZGVlMzE2MzB2MYkc5NWzE0ZWNiYjI2MA&
redirect_uri=http%3A//client.example-app.com%2Fauth%2F
```

*Örnek Sorgu (`password` yöntemi için):*
```http
POST http://www.ideashopgiyim.com/oauth/v2/token HTTP/1.1

:
grant_type=password&
client_id=7_7d67dc7597f034d63775c1d9ae5d9ac7f5750197f&
client_secret=1sowg0oogc4wg4w4o4gh4va57gggwskkgo08m44ksog8kmu88o&
username=kullanici_adi&
password=parola
```

Token isteği başarılı olursa **HTTP 200** kodu ile aşağıdakine benzer bir JSON objesi dönecektir:

```json
{
"access_token": "OGE3MzkxNDlhOGY0M2RjZWM1MWI2MWIxZjRmNDJhZThiOGJkOWZlMWIwNzVkYWFlOWFiMDAxYzkwMDlmMDhlMw",
"expires_in": 3600,
"token_type": "bearer",
"scope": null,
"refresh_token": "NjczY2E2NzZiNDJjZWI5NTE2YjZhNTlhYTJmOTQ5MjljN2MwNmUxM2MzODc5YTE4OGVjMDdlYTBiMzY1MWI1Mw"
}
```
Başarısız istek **HTTP 400** kodu ile dönecek ve `error` ve `error_description` parametrelerini içeren bir JSON objesi bulunduracaktır. Muhtemel hata kodları aşağıdaki gibidir:

error | error_description | Açıklama 
----- | ------------------| ---------
invalid_request | Invalid grant_type parameter or parameter missing | `grant_type` değeri eksik veya geçersiz
invalid_request | Missing parameter. "code" is required | `code` değeri eksik
invalid_request | Missing parameters. "username" and "password" required | `username` veya `password` değeri eksik
invalid_request | The redirect URI parameter is required | `redirect_uri` değeri eksik
unauthorized_client | The grant type is unauthorized for this client_id | Kullandığınız `grant_type` uygulamanız için geçersiz
invalid_client | The client credentials are invalid | Verilen `client_id` veya `client_secret` değeri doğru değil
invalid_grant | Invalid username and password combination | Verilen `username` veya `password` değeri doğru değil
invalid_grant | Code doesn't exist or is invalid for the client | Verilen `code` değeri doğru değil
redirect_uri_mismatch | The redirect URI is missing or do not match | `redirect_uri` değeri doğru değil
invalid_grant | The authorization code has expired | Verilen `code` değerinin kullanım süresi dolmuş


Alınan *Access Token* bundan sonraki bütün API çağrılarında kullanılacaktır. Geçerlilik süresi **1 Saat** 'dir.


*Access Token*'in geçerlilik süresi dolduğunda *Refresh Token* kullanılarak yeni bir *Access Token* alınabilir. 

*Örnek Sorgu (`refresh_token` yöntemi için):*
```http
POST http://www.ideashopgiyim.com/oauth/v2/token HTTP/1.1

:
grant_type=refresh_token&
client_id=7_7d67dc7597f034d63775c1d9ae5d9ac7f5750197f&
client_secret=1sowg0oogc4wg4w4o4gh4va57gggwskkgo08m44ksog8kmu88o&
refresh_token=NjczY2E2NzZiNDJjZWI5NTE2YjZhNTlhYTJmOTQ5MjljN2MwNmUxM2MzODc5YTE4OGVjMDdlYTBiMzY1MWI1Mw
```

Refresh Token geçerlilik süresi **14 Gün** 'dür. Bu süreden sonra, tekrar yetki verilmesi gereklidir