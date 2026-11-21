# Arquitetura de monetização

## Compras e carteira

Tipos identificados no metadata:

- App.Flow.Platform.PaymentsController (tipo 3768): Purchase, Purchase_Yookassa, Purchase_Xsolla, RestorePurchases, ApplyPurchase, Purchase_ForAd, Purchase_ForFree, Purchase_Donut, GetProduct, GetPrice, IsSale.
- App.Flow.Platform.UnityIAPImpl (tipo 3781): inicialização do IAP, Purchase, ProcessPurchase, OnPurchaseFailed, RestorePurchases, Validate.
- Bank, SmartOffer, Offer, SmartOfferCollection, Battlepass, SubscriptionState e VipStack.

O manifesto declara com.android.vending.BILLING; a integração inclui Google Play IAP e referências a Xsolla/YooKassa. As URLs /xsolla/payments, /yookassa/payments e o endpoint de Paystation foram encontrados como literais. Nenhuma transação, validação ou restauração foi executada.

O mapeamento metadata→ELF localizou `PaymentsController.Purchase_Yookassa` em RVA 0x16F46A4 e `Purchase_Xsolla` em 0x16F47EC. O Ghidra recuperou o wrapper assíncrono de `Purchase_Yookassa`, mas a state machine e a validação/concessão não foram rastreadas até o fim; o pseudocódigo tem campos globais sem tipo. Esses RVAs confirmam a localização dos pontos de entrada nomeados, não o resultado ou autoridade de uma compra.

A interface de dados do StorageController também declara `PostValidate(receipt)`, implementado tanto pelo provider remoto quanto pelo `FakeDataServer`. Isso evidencia que o contrato de servidor cobre validação, mas não revela a regra implementada no provider fake nem prova que ele seja selecionado em uma sessão normal. Nenhuma compra foi feita, nenhum recibo foi fabricado e nenhum serviço de pagamento foi acessado.

Os nomes Donut, Bank e VipStack sugerem conceitos de moeda/carteira/benefício, mas a relação exata entre cada item, preço e recompensa requer cruzar tabelas, UI e métodos ligados. Não assumo que toda moeda especial seja comprada exclusivamente.

## Anúncios com efeito em gameplay

AdsController e RewardedVideo expõem LoadAd/ShowAd e categorias de posicionamento como energia, baú misterioso, desmontar perk, trocar aparência, acelerar bancada, gaiola do Raven, poker e chef. Portanto, anúncios não são somente código de telemetria: alguns posicionamentos estão associados a ações/recompensas do jogo.

No objetivo sem monetização, essas oportunidades precisam virar regras transparentes de gameplay: fontes alternativas para recurso, tempo, baú e bônus, com custos/probabilidades coerentes. Apenas ocultar a loja ou retirar os SDKs poderia deixar fluxos e recompensas sem caminho utilizável.

## Elementos comerciais versus progressão

| Elemento | Evidência | Tratamento necessário numa implementação offline |
|---|---|---|
| Compra de produto / restauração | PaymentsController, UnityIAPImpl, permissão BILLING | Desativar o caminho de compra de forma estrutural e manter a progressão disponível por gameplay. |
| Xsolla / YooKassa | métodos e literais de rotas/URLs | Remover a dependência de checkout e preservar a função de concessão como regra local legítima, sem simular recibos. |
| Ofertas / Smart Offers | Offer, SmartOffer, TryMakeOffer, TryTrigger, ApplyPurchase | Retirar segmentação/compras e manter apenas recompensas ou conteúdo ganho em jogo. |
| Battlepass / assinatura / VIP | tipos de Battlepass, SubscriptionState, VipStack | Definir progressão e recompensas sem estado de assinatura ou validade remota. |
| Recompensa de anúncio | RewardedVideo e categorias do controller | Substituir cada uso de recompensa paga/condicionada por missão, exploração, crafting ou tempo de jogo. |

## Dependências e limites

Parte da economia é domínio do jogo; parte está ligada a pagamento e anúncios. Os metadados mostram ambos entrelaçados em controladores, mas não permitem medir todo o balanceamento. Não foram alterados mecanismos de pagamento, falsificados recibos, contatadas lojas ou acessadas contas.

Evidências: metadata IL2CPP versão 39, permissões do manifesto em decoded-resources-base.json, assets de imagem de oferta no split Unity e classes/métodos acima.
