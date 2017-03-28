##Android底层使用的网络通信模块
1. 在2.3以前使用的是HttpClient
2. 在2.3及以后使用的是HttpOpenUrlConnection
3. 在5.0系统中，移除了HttpClient组件，而采用okhttp为主的网络通信框架。

##信任所有的证书
实现X509TrustManager

```
public class TrustAllCertsManager implements X509TrustManager {

    private static TrustManager[] trustManagers;

    @Override
    public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {

    }

    @Override
    public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {

    }

    @Override
    public X509Certificate[] getAcceptedIssuers() {
        return new X509Certificate[0];
    }

    /**
     * 允许所有的SSL
     */
    public static void allowAllSSL() {
        if (trustManagers == null) {
            trustManagers = new TrustManager[]{new TrustAllCertsManager()};
        }

        SSLContext sslContext = null;
        try {
            sslContext = SSLContext.getInstance("SSL");
            sslContext.init(null, trustManagers, new java.security.SecureRandom());
        } catch (NoSuchAlgorithmException | KeyManagementException e) {
            e.printStackTrace();
        }
        if (sslContext != null) {
            HttpsURLConnection.setDefaultSSLSocketFactory(sslContext.getSocketFactory());
            HttpsURLConnection.setDefaultHostnameVerifier(new HostnameVerifier() {
                @Override
                public boolean verify(String hostname, SSLSession session) {
                    return false;
                }
            });
        }
    }

}

```
重写HurlStack

```
public class TTHurlStack extends HurlStack {

    public TTHurlStack(UrlRewriter urlRewriter, SSLSocketFactory sslSocketFactory) {
        super(urlRewriter, sslSocketFactory);
    }

    @Override
    protected HttpURLConnection createConnection(URL url) throws IOException {
        if (url.toString().contains("https")) {
            TrustAllCertsManager.allowAllSSL();
        }
        return super.createConnection(url);
    }
}
```
##信任CA颁布的证书或者自签名的证书
```
private SSLSocketFactory newSslSocketFactory(Context ctx,@RawRes int id) {
        try {
            KeyStore trusted = KeyStore.getInstance("PKCS12", "BC");

            InputStream in = ctx.getResources().openRawResource(id);

            CertificateFactory cerFactory = CertificateFactory.getInstance("X.509");
            Certificate cer;
            try {
                cer = cerFactory.generateCertificate(in);
            } finally {
                in.close();
            }

            trusted.load(null, null);
            trusted.setCertificateEntry("ca", cer);

            String tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
            TrustManagerFactory tmf = TrustManagerFactory.getInstance(tmfAlgorithm);
            tmf.init(trusted);

            SSLContext context = SSLContext.getInstance("TLS");
            context.init(null, tmf.getTrustManagers(), null);

            return context.getSocketFactory();
        } catch (Exception e) {
            throw new AssertionError(e);
        }
    }

```
##信任PublicKey证书

```
 public SSLSocketFactory newSslSocketFactory(String publicKey) {
        TrustManager tm[] = {new PubKeyManager(publicKey)};
        SSLSocketFactory sslSocketFactory = null;
        try {
            SSLContext context = SSLContext.getInstance("TLS");
            context.init(null, tm, null);
            sslSocketFactory = context.getSocketFactory();
        } catch (NoSuchAlgorithmException | KeyManagementException e) {
            e.printStackTrace();
        }
        return sslSocketFactory;
    }
```

