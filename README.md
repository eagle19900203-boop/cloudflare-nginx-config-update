# Cloudflare-Nginx-Config-Update

==============

Small script to automatically update Cloudflare's public IPs in your nginx configuration.
You can schedule it with cron.

Current version is in beta stage!

Roughly it generates a similar output like this:
https://support.cloudflare.com/hc/en-us/articles/200170706-How-do-I-restore-original-visitor-IP-with-Nginx-
It uses these source urls: https://www.cloudflare.com/ips/

You have to enable this nginx module as well:
http://nginx.org/en/docs/http/ngx_http_realip_module.html

### Synopsis
    
    cloudflare-nginx-config-update.sh    [OPTIONS] [target]

### Options, Arguments

`target` is the name of the nginx configration snippet file. Its default value is `/etc/nginx/conf.d/cloudflare.conf`.

* `-c` Using `CF-Connecting-IP` header instead of the default `X-Forwarded-For`.
* `-4` Disabling IPv4 IPs.
* `-6` Disabling IPv6 IPs.
* `-x` Disabling real_ip_header directive.
* `-r` Real run mode: Overwrite the original file.
* `-s` Shows the difference between the original file and the newly generated one.
* `-n` Disabling backups. (By default he original file is saved with a `.bak` suffix.)
* `-d` Debug mode: shows the core logic's all steps.
* `-q` Quiet mode: suppress most of the output. (Arguments are processed in order of appearance, Previous argument's processing messages won't be suppressed. Move this to the first place if you want to suppress everything.)
* `-h` Help. Displays this page.

### Sample usages

By default nothing important will happen

    $ cloudflare-nginx-config-update.sh

Showing the potential changes

    $ cloudflare-nginx-config-update.sh -s

A standard usage:

    $ cloudflare-nginx-config-update.sh -sr

If you have logcheck or anything similar enabled, you may want to suppress the less interesting outputs:

    $ cloudflare-nginx-config-update.sh -qr

### Dependencies

* Nginx realip module: http://nginx.org/en/docs/http/ngx_http_realip_module.html
* bash
* wget
* cp
* diff (for showing the diff)

### Author

Veres Lajos

### Original source

https://github.com/vlajos/cloudflare-nginx-config-update

Feel free to use!
計劃將 ingress 遷移到 nginx？從這裡開始： kubernetes.nginx.org。




配置 HTTPS 伺服器
HTTPS 伺服器最佳化
SSL 憑證鏈
單一 HTTP/HTTPS 伺服器
基於名稱的 HTTPS 伺服器
     具有多個名稱的 SSL 憑證
     伺服器名稱指示
相容性
若要設定 HTTPS 伺服器，必須 在伺服器區塊的監聽套接字ssl上啟用該參數 ，並 指定 伺服器憑證 和 私密金鑰檔案的位置 ：

伺服器 {
    監聽 443 ssl；
    伺服器名稱 www.example.com；
    ssl_certificate      www.example.com.crt ;
    ssl_certificate_key www.example.com.key ;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    …
}
伺服器憑證是公開的，會傳送給每個連接到伺服器的用戶端。私鑰是安全的，應該儲存在存取受限的檔案中，但必須能夠被 nginx 的主進程讀取。私鑰也可以與憑證儲存在同一個檔案中。

    ssl_certificate www.example.com.cert;
    ssl_certificate_key www.example.com.cert;
在這種情況下，也應該限製文件存取權限。雖然憑證和金鑰儲存在同一個文件中，但只有憑證會被傳送給客戶端。

可以使用`ssl_protocols`和 `ssl_ciphers` 指令來限制連接，使其僅包含強版本的 SSL/TLS 協定和加密套件。預設情況下，nginx 使用 ` ssl_protocols TLSv1.2 TLSv1.3<ssl_protocols>` 和 ` <ssl_ciphers> ssl_ciphers HIGH:!aNULL:!MD5`，因此通常不需要明確配置它們。請注意，這些指令的預設值已 多次 變更。

HTTPS 伺服器最佳化
SSL 操作會消耗額外的 CPU 資源。在多處理器系統中，應運行多個 工作進程 ，其數量不得少於可用 CPU 核心數。 CPU 佔用率最高的操作是 SSL 握手。有兩種方法可以最大限度地減少每個客戶端的握手次數：第一種方法是啟用 keepalive 連接，以便透過一個連接發送多個請求；第二種方法是重複使用 SSL 會話參數，以避免並行連接和後續連接進行 SSL 握手。會話儲存在工作進程共享的 SSL 會話快取中，該快取由 `ssl_session_cache`指令配置。 1 MB 的快取大約可以儲存 4000 個會話。預設快取逾時時間為 5 分鐘。可以使用`ssl_session_timeout`指令 增加快取逾時時間 。以下是一個針對具有 10 MB 共享會話快取的多核心系統最佳化的範例配置：

worker_processes 自動；

http {
    ssl_session_cache shared:SSL:10m ;
     ssl_session_timeout 10m ;

    伺服器 {
        監聽 443 ssl；
        伺服器名稱 www.example.com；
        keepalive_timeout 70 ;

        ssl_certificate www.example.com.crt;
        ssl_certificate_key www.example.com.key;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;
        …

SSL憑證鏈
某些瀏覽器可能會對由知名憑證授權單位簽發的憑證提出異議，而其他瀏覽器則可能毫無問題地接受該憑證。這是因為頒發機構使用了一個中間憑證對伺服器憑證進行了簽名，而該中間憑證並不存在於特定瀏覽器隨附的知名可信任憑證授權單位的憑證庫中。在這種情況下，頒發機構會提供一組鍊式證書，這些證書應與已簽署的伺服器證書連接起來。在合併後的憑證檔案中，伺服器憑證必須位於鍊式憑證之前：

$ cat www.example.com.crt bundle.crt > www.example.com.chained.crt
產生的檔案應該用於 ssl_certificate指令中：

伺服器 {
    監聽 443 ssl；
    伺服器名稱 www.example.com；
    ssl_certificate www.example.com.chained.crt;
    ssl_certificate_key www.example.com.key;
    …
}
如果伺服器憑證和捆綁包的連線順序錯誤，nginx 將無法啟動並顯示錯誤訊息：

SSL_CTX_use_PrivateKey_file(" ... /www.example.com.key") 失敗
   （SSL：錯誤：05800074：x509 憑證例程::密鑰值不符）
因為 nginx 嘗試使用捆綁包的第一個憑證中的私鑰，而不是伺服器憑證。

瀏覽器通常會儲存接收到的、由受信任機構簽署的中間證書，因此常用的瀏覽器可能已經擁有所需的中間證書，並且可能不會對缺少完整證書鏈的證書發出警告。為了確保伺服器發送完整的憑證鏈，openssl可以使用命令列實用程序，例如：

$ openssl s_client -connect www.godaddy.com:443
…
憑證鏈
 0 s:/C=US/ST=Arizona/L=Scottsdale/1.3.6.1.4.1.311.60.2.1.3=US
     /1.3.6.1.4.1.311.60.2.1.2=AZ/O=GoDaddy.com, Inc
     /OU=MIS 部門/ CN=www.GoDaddy.com
     /serialNumber=0796928-7/2.5.4.15=V1.0，條款 5.(b)
   i:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc.
     /OU=http://certificates.godaddy.com/repository
     /CN=GoDaddy 安全認證機構
     /serialNumber=07969287
 1 s:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc.
     /OU=http://certificates.godaddy.com/repository
     /CN=GoDaddy 安全認證機構
     /serialNumber=07969287
   i:/C=US/O=GoDaddy集團公司
     /OU=GoDaddy 二級認證機構
 2 s:/C=US/O=Go Daddy 集團有限公司
     /OU=GoDaddy 二級認證機構
   i:/L=ValiCert 驗證網路/O= ValiCert 公司
     /OU=ValiCert 二級策略驗證機構
     /CN=http://www.valicert.com//emailAddress=info@valicert.com
…
在這個例子中，伺服器憑證 #0 的主題（「s」） 由頒發者（「 iwww.GoDaddy.com 」）簽名，而頒發者本身又是證書 #1 的主題，證書 #1 由頒發者簽名，證書 #2 本身又是證書 #2 的主題，證書 #2 由著名的頒發者ValiCert, Inc. ，其證書存儲在瀏覽器的證書中（位於 Jack 的證書庫中）。

如果尚未新增憑證包，則只會顯示伺服器憑證 #0。

單一 HTTP/HTTPS 伺服器
可以設定一台伺服器來同時處理 HTTP 和 HTTPS 請求：

伺服器 {
    聽 80；
    監聽 443 ssl；
    伺服器名稱 www.example.com；
    ssl_certificate www.example.com.crt;
    ssl_certificate_key www.example.com.key;
    …
}

在 0.7.14 版本之前，無法像上圖所示那樣，為單一監聽套接字選擇性地啟用 SSL。 SSL 只能使用 `ssl`指令為整個伺服器啟用，這使得無法設定單一 HTTP/HTTPS 伺服器。 `listen`指令ssl的參數 是為了解決這個問題而增加的。因此，不建議在現代版本中使用 `ssl`指令；指令已在 1.25.1 版本中移除。

基於名稱的 HTTPS 伺服器
當配置兩個或多個監聽相同 IP 位址的 HTTPS 伺服器時，經常會出現一個問題：

伺服器 {
    監聽 443 ssl；
    伺服器名稱 www.example.com；
    ssl_certificate www.example.com.crt;
    …
}

伺服器 {
    監聽 443 ssl；
    伺服器名稱 www.example.org；
    ssl_certificate www.example.org.crt;
    …
}
在這種配置下，瀏覽器會收到預設伺服器的證書，www.example.com無論請求的伺服器名稱為何。這是由 SSL 協定的行為導致的。 SSL 連線在瀏覽器傳送 HTTP 請求之前建立，因此 nginx 並不知道請求的伺服器名稱。所以，它可能只會提供預設伺服器的憑證。

解決此問題的最古老、最可靠的方法是為每個 HTTPS 伺服器分配一個單獨的 IP 位址：

伺服器 {
    監聽 192.168.1.1:443 ssl;
    伺服器名稱 www.example.com；
    ssl_certificate www.example.com.crt;
    …
}

伺服器 {
    監聽 192.168.1.2:443 ssl;
    伺服器名稱 www.example.org；
    ssl_certificate www.example.org.crt;
    …
}

具有多個名稱的 SSL 證書
還有其他方法允許多個 HTTPS 伺服器共享同一個 IP 位址。然而，所有這些方法都有其缺點。一種方法是在憑證的 SubjectAltName 欄位中使用多個名稱，例如 `example.com` www.example.com和 ` www.example.orgexample.com`。但是，SubjectAltName 欄位的長度是有限制的。

另一種方法是使用帶有通配符名稱的證書，例如 `example.com` *.example.org。通配符憑證可以保護指定網域的所有子網域，但僅限於一級。此憑證符合 `example.com` www.example.org，但不符合 `example.com` example.org和 `example.com` www.sub.example.org。這兩種方法也可以結合使用。憑證的 SubjectAltName 欄位可以同時包含精確名稱和通配符名稱，例如 `example.com` example.org和`example.com` *.example.org。

最好將包含多個憑證名稱的憑證檔案及其私鑰檔案放在HTTP配置層，以便在所有伺服器上繼承它們的單一記憶體副本：

ssl_certificate common.crt;
ssl_certificate_key common.key;

伺服器 {
    監聽 443 ssl；
    伺服器名稱 www.example.com；
    …
}

伺服器 {
    監聽 443 ssl；
    伺服器名稱 www.example.org；
    …
}

伺服器名稱指示
在單一 IP 位址上運行多個 HTTPS 伺服器的更通用解決方案是 TLS 伺服器名稱指示擴展(SNI，RFC 6066)。 SNI 允許瀏覽器在 SSL 握手期間傳遞請求的伺服器名稱，讓伺服器知道應該使用哪個憑證來建立連線。目前 大多數現代瀏覽器都支援SNI ，且 SNI 是 TLSv1.3 中強制實現的擴展，但某些舊版或特殊用戶端可能不支援。

SNI 只能傳遞域名，但如果請求中包含伺服器的 IP 位址，某些瀏覽器可能會錯誤地將 IP 位址傳遞為網域。因此，不應依賴這種機制。

要在 nginx 中使用 SNI，nginx 二進位檔案所使用的 OpenSSL 函式庫以及執行時間動態連結到的函式庫都必須支援 SNI。如果 OpenSSL 0.9.8f 版本在建置時使用了設定選項 “--enable-tlsext”，則支援 SNI。 從 OpenSSL 0.9.8j 版本開始，此選項預設為啟用。如果 nginx 在建置時啟用了 SNI 支持，則在使用「-V」開關運行時，nginx 將顯示以下資訊：

$ nginx -V
…
已啟用 TLS SNI 支持
…
但是，如果啟用了 SNI 的 nginx 被動態連結到一個不支援 SNI 的 OpenSSL 函式庫，nginx 將顯示警告：

nginx 最初設計時支援 SNI，但現在它已鏈接
動態地傳遞給不支援 tlsext 的 OpenSSL 函式庫，
因此，SNI不可用。

相容性

自 0.8.21 和 0.7.62 版本以來，SNI 支援狀態已透過「-V」開關顯示。
listenssl指令的參數 從 0.7.14 版本開始就得到了支援。在 0.8.21 版本之前，它只能與 參數一起指定。 default
自 0.5.23 版本起，SNI 已獲得支援。
自 0.5.6 版本起，已支援共享 SSL 會話快取。


1.27.3 及更高版本：預設 SSL 協定為 TLSv1.2 和 TLSv1.3（如果 OpenSSL 庫支援）。否則，當使用 OpenSSL 1.0.0 或更早版本時，預設 SSL 協定為 TLSv1 和 TLSv1.1。
版本 1.23.4 及更高版本：預設 SSL 協定為 TLSv1、TLSv1.1、TLSv1.2 和 TLSv1.3（如果 OpenSSL 庫支援）。
版本 1.9.1 及更高版本：預設 SSL 協定為 TLSv1、TLSv1.1 和 TLSv1.2（如果 OpenSSL 庫支援）。
版本 0.7.65、0.8.19 及更高版本：預設 SSL 協定為 SSLv3、TLSv1、TLSv1.1 和 TLSv1.2（如果 OpenSSL 函式庫支援）。
版本 0.7.64、0.8.18 及更早版本：預設 SSL 協定為 SSLv2、SSLv3 和 TLSv1。


版本 1.0.5 及更高版本：預設 SSL 密碼為「HIGH:!aNULL:!MD5」。
版本 0.7.65、0.8.20 及更高版本：預設 SSL 密碼為「HIGH:!ADH:!MD5」。
版本 0.8.19：預設的 SSL 密碼套件為「ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM」。
版本 0.7.64、0.8.18 及更早版本：預設 SSL 密碼套件為
「ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP」。

作者：伊戈爾·西索耶夫
編輯：布萊恩·默瑟
