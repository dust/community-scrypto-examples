## business model and flow
![flow](res/scrypto_example_escrow.jpg)


## test

``` shell
resim reset

result=$(resim new-account)
export acct1=$(echo $result|grep "Account component address: "|awk -F ": " '{print $2}'|awk -F " " '{print $1}')
export acct_private1=$(echo $result | grep "Private key:" | awk -F "Private key: " '{print $2}')
result=$(resim new-account)
export acct2=$(echo $result|grep "Account component address: "|awk -F ": " '{print $2}'|awk -F " " '{print $1}')
export acct_private2=$(echo $result|grep "Private key:"|awk -F "Private key: " '{print $2}')

export xrd=030000000000000000000000000000000000000000000000000004
resim set-default-account $acct2 $acct_private2
result=$(resim new-token-fixed 8000000000 --description "Demo Token" --name "DemoToken" --symbol "DXT")
export dxt=$(echo $result | grep "Resource: " | awk -F "Resource: " '{print $2}')

resim set-default-account $acct1 $acct_private1
result=$(resim publish ".")
# echo $result
export pkg=$(echo $result  | awk -F "Package: " '{print $2}')

## component, bucket1(badge), bucket2(badge)
result=$(resim call-function $pkg Escrow new $xrd $dxt $acct1 $acct2)
export component=$(echo $result | grep "Component: "|awk -F"Component: " '{print $2}')
export badge1=$(echo $result | grep "Resource: "|awk -F"Resource: " '{if(NR==1)print $2}')
export badge2=$(echo $result | grep "Resource: "|awk -F"Resource: " '{if(NR==2)print $2}')

## transfer badge2 to acct2
## resim show $acct1
## resim show $acct2
resim transfer 1 $badge2 $acct2


## ready(U1)
resim call-method $component put_tokens 100,$xrd 1,$badge1
## ready(U2)
resim set-default-account $acct2 $acct_private2
resim call-method $component put_tokens 10,$dxt 1,$badge2

## accept(U1)
resim set-default-account $acct1 $acct_private1
resim call-method $component accept 1,$badge1
## accept(U2)
resim set-default-account $acct2 $acct_private2
resim call-method $component accept 1,$badge2

## withdraw(U2)
resim call-method $component withdraw 1,$badge2
## withdraw(U1)
resim set-default-account $acct1 $acct_private1
resim call-method $component withdraw 1,$badge1

```

## Scrypto 

#### referencing resources such as Bucket or Badge from the command line
$quantity,$resource_address

#### transfer
1. withdraw(resource) --> bucket(resource)
2. bucket(resource) --> account

#### badge
proof of auth, similar passport.

#### Concepts
| Scrypto                                 |  Java                           |
| --------------------------------------- | ------------------------------- |
| blueprint                               | jar(java Archive), application  | 
| package(struct & impl)                  | class                           |
| component                               | instance (object)               |
| function                                | static method                   |
| method                                  | method                          |

#### Resource, Vault, Bucket, Package, Component, Blueprint
