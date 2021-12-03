# solution de paiement SSI
inspiré de la solution Hyperledger Aries :

https://github.com/hyperledger/aries-rfcs/blob/main/features/0075-payment-decorators/README.md#payment_receipt

## Overview

l’issuer propose un  prix pour un VC, l’utilisateur si il accepte la proposition choisit un des moyens de paiement proposé par l ‘issuer et transfère les fonds.
 il joint son reçu à l’issuer qui émet le VC.


Dans notre flow 


GET : l’issuer on ajoute un attribut “payment_request” au credentialOffer :

```javascript
credentialOffer = {
           "type": "CredentialOffer",
           "credentialPreview": {...},
           "expires" : 12/08/2021Z "
            “payment_request : {...}

```

Cf le json payment_request en annexe


### Step 1
Au dessous de l’affichage de l' offre , Il faudrait afficher à l’utilisateur le détail de la facture (cf attribut "détails" de payment_request et demander l’accord du user pour procéder au paiement. 


### Step 2
Si ok il faut proposer à l’utilisateur les moyens de paiements possibles qui sont dans l’attribut “supportedMethods” de payment_request de l’issuer et dont le label est fixé par le téléphone (hard codé), exemple de label propre au téléphone :

```javascript
{
   "supportedMethods": "basic_card",
   "label": “Pay with Card (VISA, Mastercard)”
 },
 {
   "supportedMethods": "talao_test",
   "label": “Pay with TALAO token (test)”
 },{
   "supportedMethods": "talao",
   "label": “Pay with TALAO token”
 },
 {
   "supportedMethods": "tezos",
   "label": “Pay with XTZ (Tezos)”
 },
```

### Step 3

Dans le cas ou la méthode choisie par el user est talao_test  il faut réaliser un transfert avec des “faux” tokens de tests qui se trouve sur une blockchain ethereum privée (talaonet).

Cf le code de contract.dart


Dans les autres cas afficher "Démo contact@talao.io”

### Step 4

Récuperer le hash de la transaction et l’envoyer a l’issuer avec un payment_receipt a ajouter a la response au GET faite par le wallet

```javascript
{
           “Subject_id”, ”did:tz:tz1e5YakmACgZZprF7YWHMqnSvcWVXZ2TsPW”,
            “payment_receipt”: {.....}
}
```



exemple :

```javascript

  payment_receipt": {
      "request_id": ".....mettre l id du vc ",
      "selected_method": "talao_test",
      "selected_shippingOption": "standard",
      "transaction_id": “....mettre le hsh de la transaction ...",
      "proof": "directly verifiable proof of payment",
      "payeeId": “... mettre l addresse ou on a envoyé les tokens ….",
      "amount": { "currency": "TALAO", "value": "1300" }
  }

```


## Autres

Pour que l’utilisateur puisse accéder a ses comptes de token il faut ajouter dans la page “information générale”  son addresse Tezos et son address ethereum :


addresse tezos c est celle qui est dans le did,


 addresse Ethereum est calculé dans contract.dart

## Annexe

### exemple de payment_request


```javascript

"payment_request": {
   "methodData": [
     {
       "supportedMethods": "basic_card",
       "data": {
         "supportedNetworks": ["visa", "mastercard"],
         "payeeId": "12345"
       }
     },
     {
       "supportedMethods": "talao_test",
       "data": {
         "supportedNetworks": ["talaonet"],
         "payeeId": "0xee09654eedaa79429f8d216fa51a129db0f72250"
       }
     },
     {
       "supportedMethods": "talao",
       "data": {
         "supportedNetworks": ["ethereum"],
         "payeeId": "0xee09654eedaa79429f8d216fa51a129db0f72250"
       }
     },
     {
       "supportedMethods": "tezos",
       "data": {
         "supportedNetworks": ["mainnet"],
         "payeeId": "did:tz:tz2NQkPq3FFA3zGAyG8kLcWatGbeXpHMu7yk"
       }
     }
   ],
   "details": {
     "id": "talao_demo_ticket",
     "displayItems": [
       {
         "label": "Ticket",
         "amount": { "currency": "EUR", "value": "55.00" },
       },
       {
         "label": "Sales Tax",
         "amount": { "currency": "EUR", "value": "5.00" },
         "type": "tax"
       },
     ],
     "total": {
       "label": "Total due",
       "amount": { "currency": "EUR", "value": "65.00" }
     },
     "modifiers": [
       {
         "supportedMethods": "talao_test",
         "total": {
           "label": "Total due",
           "amount": { "currency": "TALAO", "value": "1300" },
         },
       },
       {
         "supportedMethods": "talao",
         "total": {
           "label": "Total due",
           "amount": { "currency": "TALAO", "value": "1300" },
         },
       },
       {
         "supportedMethods": "tezos",
         "total": {
           "label": "Total due",
           "amount": { "currency": "XTZ", "value": "13" },
         },
       },
     ]
   }
 }

```
