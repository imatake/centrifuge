# centrifuge

HTTP のメソッドごとに、接続先を振り分けます。

# 使い方：
```
% centrifuge -p :10443/ssl host1:80 GET:host2:443/ssl POST:host3:80
```

とすると、 10443 ポートで SSL で待ち受け、
GET リクエストは host2 へ、
POST リクエストは host3 へ、
その他のリクエストは host1 へリダイレクトします。

この際、 host2 へのアクセスは 443 番ポートで、 SSL を使用します。

また、 `-v=true` を指定すると、リクエストヘッダの内容、転送先の接続などの詳細な出力が表示される。


## tcp4/tcp6
```
% centrifuge -f tcp6 -p :10443/ssl host1:80
% centrifuge -f tcp4 -p :10443/ssl host1:80
```
`-f` オプションで、tcp6 を指定した場合は IPv6、tcp4 の場合は IPv4 で待ち受ける。
未指定の場合は、 tcp となり、待ち受けるアドレスから適に判断される。

## SSL-VPN
```
% centrifuge -p :10443/ssl host1:80 SSL-VPN:vpnserver:443/ssl
```

`SSL-VPN` を指定した場合、メソッド名ではなく、ヘッダに「SSL-VPN」が含まれるものが対象となる。

## TLS 証明書
### ACME - Let's Encrypt
```
% centrifuge -n hostname -p :10443/ssl host1:80
```

とすることで、ドメイン名 *hostname* で、Let's Encrypt の証明書発行を行います。
また、ドメインの所有確認を 10080番ポートで待ち受けるので、80番ポートから転送されるようにしておく必要がある。

### キーペアファイル
```
% centrifuge -c /etc/ssl/example.com.crt -k /etc/ssl/private/example.com.key -p :10443/ssl host1:80
```

とすることで、証明書 */etc/ssl/example.com.crt* と秘密鍵 */etc/ssl/private/example.com.key* を読み込む。

```
% centrifuge -c example.com.crt -k example.com.key -c example.net.crt -k example.net.key -p :10443/ssl host1:80
```

と、 `-c` と `-k` のキーペアを複数指定することで、それらのキーペアファイルを読み込む（ペアの順番は合わせる必要がある）。

# なんの役に立つの？

SSTP サーバーとの振り分けとか、自前のメソッドの定義が楽になります。

SSTP 接続は、ヘッダに SSTP で始まる文字列を送ってきますが、
Windows は最初の１文字、つまり S のみしか最初は送ってこないので、
以下のようになります。

```
% centrifuge -p :10443/ssl localhost:80 S:****.vpnazure.net:443/ssl
```

他にも、自前でメソッドを追加して、 443番ポートを節約できます。

```
% centrifuge -p :10443/ssl localhost:80 STONE:localhost:10022
```

とすると、 STONE ヘッダを持つ接続は、 localhost の 10022 番へリダイレクトされます。

なお、ヘッダも 10022 番へ送られるので、 stone 等で待ち受ける際には気を付けてください。

ssh と ssl を振り分けるのには、 sslh コマンドを推奨します。
