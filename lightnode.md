## Light Node Nedir ve Nasıl Kurulur ?
- İşlemlerin doğruluğunu sağlamak için blokların header bilgilerini barındıran düğümlere verilen isimdir. Bu şekilde bütün ağı depolamadıkları için çok daha düşük depolama alanı ihtiyacı duyarlar. 

## Sistem Gereksinimleri
- Memory: 2 GB RAM
- CPU: Single Core
- Disk: 25 GB SSD

## Tmux Kullanımı
- Yeni tmux tekranı açmak: ```tmux```
- Tmux ekranı listelemek: ```tmux ls```
- Önceden açılmış tmux ekranını açmak: ```tmux attach -t tmuxsayfaismi```
- Tmux sayfasından çıkmak:``ctrl+b`` basıp sonra sadece ``d`` ye basmak

## Sistem Güncellemesi
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git make ncdu -y
```

## Go Yüklüyoruz
```
ver="1.18.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

- ``go version`` yazdığınızda ``go version go1.18.2 linux/amd64`` sonucunu vermesi gerekiyor.

## Celestia Light Node Yükleme
```
cd $HOME
rm -rf celestia-node
git clone https://github.com/celestiaorg/celestia-node.git
cd celestia-node/
git checkout tags/v0.3.0-rc2
make install
```

- ``celestia version`` yazdığımızda ``Semantic version: v0.3.0-rc2 Commit: 89892d8b96660e334741987d84546c36f0996fbe`` çıktılarını almalıyız.

## Inıt İşlemi
```
celestia light init
```

- Yukarıdaki kodu girdikten sonra aşağıdaki gibi bir çıktı almalıyız.
```
2022-09-15T16:03:10.588Z        INFO    node    node/init.go:26 Initializing Light Node Store over '/root/.celestia-light'
2022-09-15T16:03:10.589Z        INFO    node    node/init.go:62 Saving config   {"path": "/root/.celestia-light/config.toml"}
2022-09-15T16:03:10.590Z        INFO    node    node/init.go:67 Node Store initialized
```

## Cüzdan Oluşturma
```
make cel-key
./cel-key add <cüzdan-adınız> --keyring-backend test --node.type light
```
- İkinci kodu çalıştırdıktan sonra size cüzdan adresinizi ve kelimeleri gösterecek. Bunları mutlaka bir yere kaydedin.

## Cüzdan İsmi ve Adresi Öğrenmek
```
./cel-key list --node.type light --keyring-backend test
```

## Servis Oluşturma ve Node Başlatma
- ``<cüzdan-adınız>`` cüzdan adı bölümünü yukarıda koyduğunuz isimle değiştirmeyi unutmayın.
 
```
sudo tee <<EOF >/dev/null /etc/systemd/system/celestia-lightd.service
[Unit]
Description=celestia-lightd Light Node
After=network-online.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/celestia light start --core.grpc https://rpc-mamaki.pops.one:9090 --keyring.accname <cüzdan-adınız>
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

## Node Başlatma
```
systemctl enable celestia-lightd && systemctl start celestia-lightd && journalctl -u celestia-lightd.service -f
```

- Eğer ``No journal files were found`` hatası alıyorsanız aşağıdaki kodu girdikten sonra tekrar deneyin.
```
sudo systemctl restart systemd-journald
```
### Faucet Kullanma
- Test tokenı almak için da faucet kanalına (https://discord.gg/JeRdw5veKu) gidip ``$request celestia1.......`` şeklinde yazarak token alıyoruz. Başka cüzdandan da atabilirsiniz.
-----------------------------------------
- Senkronizasyonun bitmesini bekliyoruz.
-----------------------------------------
### Cüzdan Bakiyesi Sorgulama
```
curl -X GET http://127.0.0.1:26658/balance
```

### Blok Başlık (Header) Bilgisi Almak
- Blok sayısını yani 1'i isteğinize göre düzenleyebilirsiniz.
```
curl -X GET http://127.0.0.1:26658/header/1 | jq
```

### PayForData TX İşlemi
- Aşağıda verilen değerler hazır değerlerdir. Sizler şu siteden istediğiniz değeri vererek kendinize göre sorgulama yapabilirsiniz. (https://go.dev/play/p/7ltvaj8lhRl)

```
curl -X POST -d '{"namespace_id": "0c204d39600fddd3",
  "data": "f1f20ca8007e910a3bf8b2e61da0f26bca07ef78717a6ea54165f5",
  "gas_limit": 60000}' http://localhost:26658/submit_pfd | jq
```
- Aşağıdakine benzer bir çıktı almalısınız.
```
{
   "height":214000,
   "txhash":"04A79AF9DA62FDB41ACD7D82EB0B9004AE4E4ED603B280A65816560B4F38A999",
   "data":"12200A1E2F7061796D656E742E4D7367506179466F7244617461526573706F6E7365",
   "raw_log":"[{\"msg_index\":0,\"events\":[{\"type\":\"message\",\"attributes\":[{\"key\":\"action\",\"value\":\"/payment.MsgPayForData\"}]},{\"type\":\"payfordata\",\"attributes\":[{\"key\":\"signer\",\"value\":\"celestia1vdjkcetnw35kzvtgxvmxwmnwwaaxuet4xp3hxut6dce8wctsdq6hjwfcxd5xvvmyddsh5mnvvaaq6776xw\"},{\"key\":\"size\",\"value\":\"27\"}]}]}]",
   "logs":[
      {
         "msg_index":0,
         "events":[
            {
               "type":"message",
               "attributes":[
                  {
                     "key":"action",
                     "value":"/payment.MsgPayForData"
                  }
               ]
            },
            {
               "type":"payfordata",
               "attributes":[
                  {
                     "key":"signer",
                     "value":"celestia1vdjkcetnw35kzvtgxvmxwmnwwaaxuet4xp3hxut6dce8wctsdq6hjwfcxd5xvvmyddsh5mnvvaaq6776xw"
                  },
                  {
                     "key":"size",
                     "value":"27"
                  }
               ]
            }
         ]
      }
   ],
   "events":[
      {
         "type":"coin_spent",
         "attributes":[
            {
               "key":"spender",
               "value":"celestia10jhckjxxymsufflglvypxscnmggetwh0gfasws",
               "index":true
            },
            {
               "key":"amount",
               "value":"1utia",
               "index":true
            }
         ]
      },
      {
         "type":"coin_received",
         "attributes":[
            {
               "key":"receiver",
               "value":"celestia17xpfvakm2amg962yls6f84z3kell8c5lpnjs3s",
               "index":true
            },
            {
               "key":"amount",
               "value":"1utia",
               "index":true
            }
         ]
      },
      {
         "type":"transfer",
         "attributes":[
            {
               "key":"recipient",
               "value":"celestia17xpfvakm2amg962yls6f84z3kell8c5lpnjs3s",
               "index":true
            },
            {
               "key":"sender",
               "value":"celestia10jhckjxxymsufflglvypxscnmggetwh0gfasws",
               "index":true
            },
            {
               "key":"amount",
               "value":"1utia",
               "index":true
            }
         ]
      },
      {
         "type":"message",
         "attributes":[
            {
               "key":"sender",
               "value":"celestia10jhckjxxymsufflglvypxscnmggetwh0gfasws",
               "index":true
            }
         ]
      },
      {
         "type":"tx",
         "attributes":[
            {
               "key":"fee",
               "value":"1utia",
               "index":true
            }
         ]
      },
      {
         "type":"tx",
         "attributes":[
            {
               "key":"acc_seq",
               "value":"celestia10jhckjxxymsufflglvypxscnmggetwh0gfasws/267",
               "index":true
            }
         ]
      },
      {
         "type":"tx",
         "attributes":[
            {
               "key":"signature",
               "value":"JMNihnKS/MtYJDprqEFGJuXh16tVADsDDxXaFFpvv2te57btl4LbiRzwRRiN2rvwkJ2zlAApu2ImT22MZBi5+A==",
               "index":true
            }
         ]
      },
      {
         "type":"tx",
         "attributes":[
            {
               "key":"fee",
               "value":"",
               "index":true
            }
         ]
      },
      {
         "type":"tx",
         "attributes":[
            {
               "key":"acc_seq",
               "value":"celestia13zx48t96zauht0kpcn0kcfykc9wn8fehzcp9wq/1024",
               "index":true
            }
         ]
      },
      {
         "type":"tx",
         "attributes":[
            {
               "key":"signature",
               "value":"mIZIjbzN0/RQAlQN7TDWzqtey3vVBPe7IO3+IIDhJstIH8QU9vsHfl0Rql9qWMZQG4dM+77w9WmUcnCeS7edfw==",
               "index":true
            }
         ]
      },
      {
         "type":"tx",
         "attributes":[
            {
               "key":"fee",
               "value":"",
               "index":true
            }
         ]
      },
      {
         "type":"tx",
         "attributes":[
            {
               "key":"acc_seq",
               "value":"celestia1h36gnnwzneu0csqzn2waph5y983hf3dkaznlgz/0",
               "index":true
            }
         ]
      },
      {
         "type":"tx",
         "attributes":[
            {
               "key":"signature",
               "value":"sfy+XyP7iWU+V9q3zEIOWxbGihvhzUKRLNVeXP+a+5oRefIA/Pyqfm13A5NU9I27hhfvpqo9vhXW1waRgcI9OA==",
               "index":true
            }
         ]
      },
      {
         "type":"message",
         "attributes":[
            {
               "key":"action",
               "value":"/payment.MsgPayForData",
               "index":true
            }
         ]
      },
      {
         "type":"payfordata",
         "attributes":[
            {
               "key":"signer",
               "value":"celestia1vdjkcetnw35kzvtgxvmxwmnwwaaxuet4xp3hxut6dce8wctsdq6hjwfcxd5xvvmyddsh5mnvvaaq6776xw",
               "index":true
            },
            {
               "key":"size",
               "value":"27",
               "index":true
            }
         ]
      }
   ]
}
```

- ``Not found`` hatası alırsanız cüzdanınızda yeterli coin yok demektir, explorerdan kontrol edin (https://testnet.mintscan.io/celestia-testnet https://celestia.explorers.guru).

- PayForData işleminizi gönderdikten sonra başarılı olduğunda node, işlemin dahil edildiği blok yüksekliğini cevap olarak gönderir. Sonra size geri dönmesini sağlamak için bu blok yüksekliğini ve PayForData işleminizi gönderdiğiniz namespace_id'yi kullanabilirsiniz. Bu örnekte elde ettiğimiz blok yüksekliği 214000 olduğunu düşünelim ve aşağıdaki kodda ilgili yere yazalım.
```
curl -X GET \
  http://localhost:26658/namespaced_shares/0c204d39600fddd3/height/214000 | jq
```

- Bu kodun çıktısı aşağıdakine benzer olmalı.
```
{
   "shares":[
      "DCBNOWAP3dMb8fIMqAB+kQo7+LLmHaDya8oH73hxem6lQWX1AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=="
   ],
   "height":214000
}
```

 
