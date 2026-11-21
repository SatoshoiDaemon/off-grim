# Sistemas de gameplay e responsabilidades

## Como interpretar este mapa

Os nomes abaixo vêm das tabelas IL2CPP de Grim Soul 8.6.0, não de uma execução. Um método no cliente demonstra que há implementação/modelo local; não demonstra que a chamada funciona sem autenticação, configuração remota ou validação do servidor. A classificação de dependência é resumida aqui e detalhada em network-dependencies.md e offline-feasibility.md.

## Sistemas observados

| Sistema | Tipos / métodos indicativos | O que a evidência permite concluir |
|---|---|---|
| Núcleo de sessão | App.Core.DarkAppCore (tipo 3994): Init, Load, Run, ClaimSave, MarkAsCheater | Existe coordenador local para estado e ciclo de vida de sessão; persistência e antifraude podem conversar com backend. |
| Jogador | App.Core.Users.Player (tipo 4065) e modelos auxiliares de pets, horse, perks e equipamento | Estado e regras de personagem estão representados no cliente. |
| Inventário | App.Core.Inventories.Inventory, InventorySet; CanAdd, AddStack, RemoveStack, Split, DragItemTo, Exchange | Manipulação de slots/itens tem lógica local; saldo final e salvamento ainda podem ser sincronizados remotamente. |
| Combate | ArmorMath, DamageResolve, DamagePipeline | Nomes indicam cálculo e pipeline de dano local. Não confirmam autoridade para PvP, inimigos, drops ou validação de resultado. |
| Inimigos / IA | tipos Enemy, estados/rotinas de AI e modelos de localização | Há tipos de inimigo e IA no metadata. A leitura de comportamento e regras exatas exige métodos/pseudocódigo com vínculo nativo. |
| Mapas e exploração | App.Core.Maps.*, Dungeon, Location, MapMovement | Modelos de mapa, movimento, localização e masmorra existem no cliente. Geração/seed, eventos e desbloqueios podem depender de dados remotos. |
| Missões | App.Core.Quests.Quest, QuestCollection, TryActivate, TryComplete, TakeReward, Advance | Progressão e reivindicação de recompensa têm pontos no cliente. A origem das definições e validação de prêmios não foi confirmada. |
| Crafting / bancadas | modelos de receita, recursos e construção presentes nos tipos do domínio | Receitas e construção estão modeladas; não foi medido se tabelas locais cobrem todo conteúdo nem se há receitas/eventos remotos. |
| Progressão | níveis, perks, desbloqueios, receitas e migrações do jogador | Progresso localmente modelado; regras exatas de economia e limites pagos precisam ser cruzados com configs e chamadas. |
| Economia | Bank, recursos/moedas, Offer, SmartOffer, SmartOfferCollection | Carteira, ofertas e economia são parte do modelo de sessão. Monetização e moeda de jogo compartilham tipos e efeitos. |
| Eventos / notificações | ScheduleServerNotifications, GetLive, LoadVersionStatus, modelos de oferta e temporada | Há ganchos para conteúdo de servidor, estado live e notificações. A presença de hooks não determina dependência do loop principal. |
| Ads / recompensa | AdsController, RewardedVideo, LoadAd, ShowAd | O cliente integra anúncios recompensados em ações de energia, baú, perks e aceleração. Algumas ações afetam progressão e exigiriam fonte de recompensa substituta. |

## Persistência observada

DarkFileStorage expõe operações de leitura, gravação e substituição; DarkRefsStorage mantém refsData; StorageController possui Load, Save, LoadRefs, LoadData, Sync, Request, Receive, TryPostToken, LoadStat e métodos de bind de plataformas sociais. DarkMigrationService registra uma série de migrações de schema 1 a 27. A camada de dados separa a interface `IDarkDataServer` entre `DarkDataServer` (rede) e `FakeDataServer` (arquivo local com operações de load/save); a seleção do provider não foi confirmada.

Na state machine de `DarkFileStorage.WriteToFile`, o pseudocódigo recuperado usa `FileStream`/`WriteAsync`, verifica os caminhos temporário e anterior, e promove o arquivo temporário com operações `File.Move`. `FakeDataServer.PostLoad` e `PostSave` também leem/escrevem seu arquivo local. `StorageController.Sync` chama `Request` em um ponto de entrada estático identificado pelo Ghidra. Isso sustenta a existência simultânea de save local e sincronização; a execução real, formato completo e seleção do provider ainda não foram confirmados. O pacote não contém arquivo SQLite reconhecível por extensão. Não foram analisados dados pessoais ou saves de terceiros.

## O que ainda exige inspeção

- Continuar o pseudocódigo/xrefs além das funções-alvo já analisadas; o Ghidra 12.1.4 importou labels, mas sua análise geral expirou em 1.200 s e os pseudocódigos dirigidos ainda têm globals sem tipo. Prioridade: rastrear quem atribui `StorageController._dataServer` e se a build seleciona `FakeDataServer`.
- Decodificar completamente catálogo Addressables, tabelas de receitas, inimigos, mapas, drops e loja.
- Identificar o formato de save em um AVD descartável, sem rede e sem conta.
- Separar validação remota de cálculo local por comparação de execução offline/online autorizada. Não foi feita captura de rede nesta fase.
